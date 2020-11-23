# 实验室服务器连接指北

------
@author   钱宇欣

@version  2020.11.23
本文档用于说明实验室GPU服务器的连接方式。

------
## Contents
- [使用 ssh 连接服务器](#ssh)
- [使用 Pycharm 连接服务器](#pycharm)
- [使用 Jupyter Lab / Notebook 连接](#jupyter)
- [使用 Tensorboard 监视训练](#tensorboard)
- [使用 FTP 上传文件](#ftp)
- [后台运行](#tips)

------

## SSH Connection
([↑up to contents](#ssh))

###1. 使用 Linux 终端
在本地终端输入
```
ssh username@hostaddress
```
之后输入密码即可登录到服务器。

### 2. PuTTY
使用 Windows 操作系统的同学可使用 [PuTTY][1] 工具进行连接。
打开工具后先输入服务器地址，之后在终端分别输入账户名及密码，完成连接。

[1]: https://www.chiark.greenend.org.uk/~sgtatham/putty/
------

## Pycharm Connection
([↑up to contents](#pycharm))
### 1. 连接配置
编写好Python代码源文件之后，现在我们来设置[PyCharm][2]与云主机的连接。
点击菜单中的 Tools -> Deployment -> Configuration 打开Deployment对话框。
点击左上角的 "**+**" ，选择 SFTP ，之后点击 SSH Configuration 后的 "..."，创建一个 SSH 连接配置。
在弹出的 SSH 连接配置窗口，点击左上角的 "**+**" 号，输入主机地址以及登陆用的用户名和密码。
点击"Test Connection"测试连接，点击Apply保存。

[2]:https://www.jetbrains.com/pycharm/

### 2. 代码同步
配置完SSH后，在同一个Deployment对话框内点击 Mappings 设置路径映射，设置服务器代码同步的目标路径。在 SFTP 配置页面选择"Mappings"选项卡，在"Deplyment Path"，点击Apply保存。本地工程项目内的所有代码都会同步到该路径下。
设置好目标路径之后，再点击 Tools -> Deployment -> Options ，勾选"Create empty directories"选项，设置同步代码时自动创建文件夹。
在菜单中确认 Tools -> Deployment -> Automatic Upload (always) 选项是勾选上的。这样就可以确保Python代码可以自动同步到云主机，防止出现本地和云主机代码不一致的情况。

### 3. 远程Python解释器配置
点击 File -> Settings -> Project -> Project Interpreter ，点击齿轮 -> Add ，选择"SSH Interpreter"，勾选"Existing server configuration"。
在下拉菜单中选择刚刚创建的配置，并点击Pycharm提示的"Moving"。点击"Next"，之后要设置服务器上Python解释器的路径。在服务器配置好的Python环境中，在终端输入"which python"查看当前Python解释器的路径，并复制到当前页面的Interpreter中。

###4. Tips
在3.里修改解释器路径的地方也有一个Mappings，其与2中的设置的区别可以理解为主分支与开发分支。在本地修改代码并运行成功后，点击Tools -> Deployment -> Upload to "XXX@xxx" 便可将本地项目中的代码上传。之后连接到服务器采用后台运行的方式运行代码（这样的好处是本地机器不用一直保持和服务器的连接）。

------

## Jupyter Lab/Notebook Connection
([↑up to contents](#jupyter))

### 1. Jupyter安装
远程连接到服务器，切换到要使用的虚拟环境，使用
```1
#jupyterlab
conda install -c conda-forge jupyterlab
pip install jupyterlab
#jupyternotebook
conda install -c conda-forge notebook
pip install notebook
```

安装 [Jupyter][3] Lab或Notebook。
[3]:https://jupyter.org/index.html


### 2. Jupyter密码生成
在终端输入
```
jupyter-notebook password
```
生成密码，或使用ipython终端输入
```
ipython
from notebook.auth import passwd
passwd()
```
创建密码，保留输出的哈希值。


### 3. Jupyter配置
在终端输入
```
jupyter lab --generate-config
jupyter notebook --generate-config
```
生成配置文件，位于"~/.jupyter/jupyter_notebook_config.py"。使用vim打开，取消以下几行的注释并修改值。
```
c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.password = 'xxx:xxxxxxx' #刚刚生成的哈希密钥
c.NotebookApp.port = xxxx #自定义，也可不修改
c.NotebookApp.allow_remote_access = True
```

### 4. 运行与访问
在服务器终端输入
```
jupyter-lab
jupyter notebook
```
运行，在本地服务器上输入"hostaddress:xxxx"，输入刚刚设置的密码，则可以访问相应应用。

### 5. Tips

本地终端关闭后，启动的jupyter应用也会被关闭，可使用nohup等方法放入后台，详见本指南最后一章。
若在本地启用了系统代理，使用Firefox需手动在设置中跳过Jupyter地址的代理。

------

## Tensorboard 
([↑up to contents](#tensorboard))
在服务器上运行[Tensorboard][4]
```
tensorboard --log_dir=xxx --bind_all
```
之后在本地服务器访问程序显示的端口即可。

[4]:https://tensorflow.google.cn/tensorboard

------
## FTP-FileZilla
([↑up to contents](#ftp))
我们建议使用[FileZilla][5]在服务器和本地机之间传递数据。
在本地终端输入
```
sudo apt-get install filezilla
```
安装应用。启动后输入服务器地址，账号与密码则可进行传输。
[5]:https://www.filezilla.cn/
------
## Background Process
([↑up to contents](#tips))
以上连接方案，包括Jupyter等使用本地终端启动的应用在关闭本地的终端、浏览器之后都会被杀死，需要使用"nohup"命令。
``` 
# 后台启动juputer-lab
nohup jupyter-lab
# 后台启动脚本
nohup bash xx.sh
```
对于模型训练，可以采用Tmux将训练任务放在后台。
### Tmux
Tmux 是一款终端复用命令行工具，一般用于 Terminal 的窗口管理。Tmux 拥有如下特性：
可以同时开启多个会话和窗口，并持久地保存工作状态。
断线后任务能够在后台继续执行。
Tmux 由如下三个基础组成：
Session。即会话，任务通常在 session 中运行，在断开SSH连接后 session 仍会保持。
Window。即窗口，一个会话可以包含多个窗口。可以存在多个窗口。
Pane。即窗格，一个窗口可以包含多个窗格。类似于 Vim 中 ctrl+w 再按 v 后的效果。

在使用ssh连接到服务器的本地终端输入
```
tmux
```
即可进入 Tmux 环境。此时默认开启了一个 session-name 为 0 的 Tmux 会话。
在该终端输入执行模型训练的命令，此时如果和服务器断开连接，tmux 中的任务还会继续保持运行状态。重新连接到服务器后，输入命令
```
tmux a -t 0
```
就可以再次打开刚刚断开的窗口，其中 0 为之前会话的 session-name。
如果想从该会话中退出，可以输入如下命令：
```
tmux detach
```
这时就会回到普通的 Terminal 终端窗口。
此时可以再次输入 tmux 命令开启一个新的会话。Tmux 默认的 session-name 会逐次加1，再次新建的会话默认 session-name 就是 1 了。
如果想指定session的名称方便记忆，可以使用 -s 参数：
```
tmux new -s [session-name]
```
在普通 Terminal 终端中，可以使用这个指令查看所有的 Tmux 会话：
```
tmux ls
```
