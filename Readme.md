可下载：https://raw.githubusercontent.com/veip007/AutoRclone/master/gclone%20%E6%90%AC%E5%B1%B1%E4%B9%8B%E6%9C%AF.mhtml 观看


### 一.先安装 python 3 和 AutoRclone：
```
apt-get install -y python3 python3-pip git  # 安装系统依赖
git clone https://github.com/veip007/AutoRclone  # 拉取 AutoRclone 项目
cd AutoRclone  # 进入项目文件夹
pip3 install -r requirements.txt  # 安装项目依赖
```
然后打开这个地址：https://developers.google.com/drive/api/v3/quickstart/python ，点击```【Step 1: Turn on the Drive API】```下面的``` Enable the Drive API ```按钮，弹出的```【Configure your OAuth client】```对话框中保持默认的``` Desktop app ```不要动，点击右下角``` CREATE``` 按钮，开启成功之后点击 ```DOWNLOAD CLIENT CONFIGURATION``` 下载生成的``` credentials.json``` ，再将下载到本地的 ```credentials.json``` 上传至服务器的``` AutoRclone``` 文件夹下。

**快速方法（不推荐，有时会出现 bug）：**

此时回到 SSH ，执行``` python3 gen_sa_accounts.py --quick-setup 1 ```，复制返回的网址至浏览器打开，登陆上一步生成 ```credientials.json``` 文件时使用的账号，选择``` 允许``` ，然后复制返回的授权代码，粘贴至 ```SSH``` 终端，再复制新返回的网址至浏览器打开，使用刚才的账号登陆，点击 启用 ，回到 SSH 终端内按下回车，此时应该开始创建 SA 了，稍等片刻完成后可以看到 ```/root/AutoRclone/accounts/ ```目录下出现了一大堆 ```.json``` 后缀的 SA 授权文件。


**手动方法（推荐）：**


1.回到 SSH ，执行 ```python3 gen_sa_accounts.py --list-projects```

如果之前没有创建过项目的话返回值应该是空的，那么此时执行 ```python3 gen_sa_accounts.py --create-projects 1 ```来新建一个项目，之后再次``` python3 gen_sa_accounts.py --list-projects```，复制一下新建的项目名称，下一步要用到；

如果已存在项目，且要使用已有项目来生成 SA（请确保你知道自己在做什么），那么复制一下想要生成 SA 的项目名称，否则参考上一条的步骤来新建一个项目

2.执行 ```python3 gen_sa_accounts.py --enable-services ProjectName ```为项目开启所需要的服务，```ProjectName ```为上一步复制的项目名称，开启方法参考上文【快速方法】中的描述；

3.执行 ```python3 gen_sa_accounts.py --create-sas ProjectName``` 为项目生成 SA；

4.执行 ```python3 gen_sa_accounts.py --download-keys ProjectName``` 下载项目中 SA 的授权文件，稍等片刻 ```~/AutoRclone/accounts/ ```目录下应该出现了一大堆 ```.json ```后缀的 SA 授权文件。

### 二.为 SA 添加权限
这个部分非常繁琐，主要是因为这一过程俺只能手动完成，如果你有 G-Suite 管理员权限，那么可以参考 这里 来快速添加，下面是手动方法：

在此处：https://console.developers.google.com/apis/dashboard ，点击左侧的 ```凭据 ```可以看到创建好的项目及 SA ，自行想办法将 SA 的邮箱地址保存下来,命令如下：

```cat /root/AutoRclone/accounts/*.json | grep "client_email" | awk '{print $2}'| tr -d ',"' | sed 'N;N;N;N;N;N;N;N;N;/^$/d;G' > ~/email```

```cat ~/email```

接下来创建群组。打开 https://groups.google.com/ ，点击左上角的``` 新建群组 ```，填写相关信息后新建一个群组，建立好后点击右上角 ```管理群组``` ，点击左侧``` 直接添加成员 ```，将刚才保存的 SA 邮箱粘贴过来，注意每次只能添加 10 个，24 小时内只能添加 100 个，每个群组最多添加 600 个，邮箱地址间使用英文逗号分隔。

全部添加好之后点击右上角 ```关于``` ，记下【群组电子邮件】的地址，之后要对所有加入了群组的 SA 添加权限时，只需要添加至这个群组邮箱就行了。

### 三.使用 gclone 开始搬山

```bash <(wget -qO- https://raw.githubusercontent.com/veip007/AutoRclone/master/gclone.sh)```

查看 gclone 版本：

```gclone --version```

编辑 Rclone 配置文件：

```vim ~/.config/rclone/rclone.conf```

默认的配置文件路径在这里，不确定的话可以输入 ```rclone config file``` 来查看。

在最后加上下面内容：
```
[gc]
type = drive
scope = drive
service_account_file = /root/AutoRclone/accounts/1.json 
service_account_file_path = /root/AutoRclone/accounts/
team_drive = *****  
```

其中 ```service_account_file_path ```是存放 SA 账号授权文件的文件夹路径，不要漏了最后的``` /``` ，上面一行是该路径下任意一个 SA 账号授权文件的路径，注意文件名，例子中``` 1.json``` 仅为简写，请务必修改为一个 SA 账号文件夹下的真实文件名。```***```为团队盘ID

用法：

命令参数参考 Rclone ，都是一样的，直接将``` rclone ```换成 ```gclone ```即可无障碍使用（Rclone 怎么用呢，请参考 这篇文章 ）；

唯一不一样的地方在于，原版 Rclone 如果跨团队盘或者共享文件夹，需要多个配置盘符用于操作，gclone 支持对目录和文件的 ID 进行操作，搬运别人公开分享的文件时非常方便：

1.目录 ID ：

共享目录和团队盘应该带 ```--drive-server-side-across-configs```：

```gclone copy gc:{【源盘】ID} gc:{【目的盘】ID}  --drive-server-side-across-configs```

目录 ID 可以是：普通目录，共享目录，团队盘；

也支持目录 ID 后跟后续路径：

```gclone copy gc:{【源盘】ID} gc:{【目的盘】ID}/media/  --drive-server-side-across-configs```

2.文件 ID：

共享目录和团队盘应该带``` --drive-server-side-across-configs```：

```gclone copy gc:{共享文件的 ID} gc:{【目的盘】ID}  --drive-server-side-across-configs```

支持目录 ID 后，跟后续路径：

```gclone copy gc:{共享文件的 ID} gc:{【目的盘】ID}/media/  --drive-server-side-across-configs```

注意：命令中如果使用 ID 格式来指定路径，需要用花括号括起来，不然程序会认为那一串乱码一样的东西是文件夹名字而不是目录 ID。

举个栗子:

```gclone copy gc:{0App-QeDCIy_mUk9PVA} gc:{10zOvIf8yBmIuZgBfC3rcDKWHIlODZjXF}/media/  --drive-server-side-across-configs -P```

如何查看团队盘，文件夹以及文件的 ID ：

浏览器中打开对应的团队盘 / 文件夹 / 文件，此时查看地址栏中类似``` https://drive.google.com/drive/u/2/folders/0App-QeDCIy_mUk9PVA ```这样的地址，其中末尾的 0App-QeDCIy_mUk9PVA 就是相应的团队盘 / 文件夹 / 文件对应的 ID 了，如果是别人分享出来的内容，那么结尾一般会多``` ?usp=sharing``` 这一段，把这段删掉，只保留 ID 就可以了。

### 四.一些注意事项

1.生成 SA 及创建 Google Groups 使用普通的谷歌账号就可以；

2.创建好 SA ，并将其加入到群组之后，对于需要搬运的资源，保证【源盘】和【目的盘】都给这个群组的【群组邮箱】权限就能搬了，【源盘】有读取权限就够了，【目的盘】需要编辑权限；

3.基于以上两点，即使你没有 G-Suite 账号（即没法自己开团队盘），只要加入了别人开给你的团队盘（网上有部分大佬提供这方面的资源，收费免费的都有，请自行寻找），并有加人权限，那么就可以参照本文教程开始搬山了；

4.如果要更换机器，只需要把【SA 授权文件】（本例中在``` /root/AutoRclone/accounts/ ```路径下）及【gclone 配置文件】（本例中路径为 ```/root/.config/rclone/rclone.conf```）备份，在新机器上安装好 gclone 并在配置文件中修改一下 SA 授权文件的路径就可以直接使用了；

5.生成 SA 过程中如果 AutoRclone 遇到什么奇怪的错误，请善用``` python3 gen_sa_accounts.py --help```；

6.单个团队盘限制 40 W 文件数，注意不要超限；

7.文件数比较多的话，直接 ```gclone copy``` 可能会漏文件或重复复制，这是因为复制过程中切换 SA 和 Google Drive 允许重名文件两个原因导致的，解决办法：

1)首先对【源盘】执行``` rclone dedupe``` 命令，确保源盘无重复文件，命令用法参考 这篇文章 ；

2)不要使用~~ gclone copy~~ ，而用 ```gclone sync``` 命令；

上面这条有误导，因为俺主要用来备份，【源盘】和【目的盘】文件结构是精确一致的。如果你【目的盘】还有其他文件，直接 sync 会导致目的盘被覆盖为源盘，请慎用此命令。

3)跑完一遍 ```sync copy``` 后使用``` rclone size ```分别查看【源盘】和【目的盘】的文件数和大小（size 命令使用 gclone 会有点 bug ，所以此处切换回 Rclone），如果【源盘】大于目标盘，说明没复制完全，多跑几次 ```sync copy ```命令后再比较，直到【目的盘】大于【源盘】，说明有重复文件了，此时使用 dedupe 命令对【目的盘】去重；

4)再次 size 查看文件数，如果【源盘】文件数大于【目的盘】，就重复 sync copy 命令，如果【源盘】小于【目的盘】，就使用 dedupe 命令去重；

5)重复以上步骤，直到 rclone size 后【源盘】和【目的盘】的文件数和大小严格相等。

这个问题多出现在【源盘】是个人盘的共享文件夹，且包含的文件数较多的时候，因为个人盘的 API 有点 bug ，会导致分享的文件数较多时，通过 API 操作时读取到的文件列表不全，需要等待一段时间才能读取完全，团队盘一般不会出现这个问题

8.以上方法只适用于【目的盘】是团队盘的情况。【目的盘】是个人盘（比如无限容量的教育邮箱）的情况稍微有点复杂：

1)像上文【3.使用 gclone 开始搬山】中的示例一样直接使用会报错，原因是使用了 SA 向个人盘来写入文件，但个人盘只有号主自己有写入权限，无法使用 SA；

2)如何向个人盘搬运：

可以使用类似``` gclone gc:{【源盘】ID} he-sb:{文件夹 ID}/路径/ ```格式的命令，其中 ```he-sb``` 是通过 ```rclone config ```配置好的个人盘配置名称。命令格式和配置文件与原版 Rclone 是一致的（因为 gclone 其实就是魔改的 Rclone，除了【自动切换 SA】和【使用目录 ID 代替路径】以外的功能大家就把它当作 Rclone 来用就好了）。

通过以上两点，想必大家已经看出来了———当【目的盘】是个人盘的时候是无法突破每日 750 G 限制的，这是 Google Drive 权限的限制，目前还没见到有人能解决。
