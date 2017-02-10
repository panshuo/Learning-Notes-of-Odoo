## Chapter01 开始 Odoo 开发
本书作者建议使用 Debian 系发行版来进行开发，Odoo使用的Python版本为2.7

####首先创建一个专门用户来开发Odoo：
`$ useradd -m -g sudo -s /bin/bash odoo # 创建一个名字为 'odoo' 的用户 并指定shell为bash，赋予sudo权限，创建用户目录`

`$ passwd odoo #为 'odoo' 设置密码`

####从9.0版开始 Odoo需要CSS预处理器less，通过npm来安装：
`$ sudo apt-get install npm # 安装Node.js和包管理器`

`$ sudo ln -s /usr/bin/nodejs /usr/bin/node # 为nodejs创建软链接 方便调用`

`$ sudo npm install -g less less-plugin-clean-css # 安装less编译器`

####接下来创建一个Odoo开发目录 'odoo-dev' 并从github下载代码仓库到本地：
`$ mkdir ~/odoo-dev && cd ~/odoo-dev`

`$ git clone https://github.com/odoo/odoo.git -b 10.0 --depth=1 # -b参数指定分支为10.0`

`$ ./odoo/setup/setup_dev.py setup_deps # 安装Odoo系统的依赖包`

`$ ./odoo/setup/setup_dev.py setup_pg # 安装PostgreSQL并为当前用户创建一个数据库超级用户`

####然后就可以启动一个Odoo服务器实例了：
`$ ~/odoo-dev/odoo/odoo-bin`

Odoo服务器实例默认在8069端口进行监听，使用浏览器打开 http://<服务器地址>:8069 就可以访问Odoo系统了。
第一次访问会显示一个助手页面来创建数据库，但是作为开发者可能需要使用多个数据库，通过命令行来配置更为方便。

####成为PostgreSQL超级用户才能创建数据库，下面的命令为当前的Unix用户创建了一个PostgreSQL超级用户：
`$ sudo createuser --superuser $(whoami)`

####使用 createdb 命令来创建一个新数据库:
`$ createdb demo`

####然后使用 -d 参数来运行 Odoo，将Odoo的数据模式(表格)初始化并写入这个空的数据库:
`$ ~/odoo-dev/odoo/odoo-bin -d demo`

初始化需要几分钟时间，结束时会输出一条 'INFO' 级别的的日志信息:
>Modules loaded

*注意：Odoo的主要稳定版本之间数据库是不兼容的 (附加组件同样不兼容)*

####查看帮助：
`$ ./odoo-bin --help`

####配置文件搜索路径优先级：用户主目录下的 .odoorc -> openerp-serverrc 也可使用 --conf=
<filepath> 这个参数来指定使用哪个配置文件

####在全新安装下 .odoorc 是不会自动生成的，需要使用 --save 参数来创建默认配置文件。
`$ ~/odoo-dev/odoo/odoo-bin --save --stop-after-init`
这里使用了 --stop-after-init 选项使服务器完成操作后自动停止。这个选项一般用来运行测试或检查模块升级是否正常完成。

#### --xmlrpc-port=<port> 命令行参数运行改变默认的监听端口，在同一个主机上同时运行多个Odoo实例时需要改变默认端口。
`$ ~/odoo-dev/odoo/odoo-bin --xmlrpc-port=8070`

#### --log-level 选项可以用来设置日志的详细级别，比如想要显示debug级别的日志，使用 --log-level=debug 选项
- debug_sql 显示服务器生成的SQL查询语句
- debug_rpc 显示服务器收到请求的详细内容
- debug_rpc_answer 显示服务器发送响应的详细内容

#### 在 Settings 菜单右下方 Activate the developer mode 可激活开发者模式
#### OCA 协调对社区的贡献并管理了一个模块仓库在这里 https://github.com/OCA/ .
#### 默认的Odoo附加组件目录位于 ~/odoo-dev/odoo/addons/. 使用 --addons-path 选项可以使用指定的附加组件目录
`$ ./odoo-bin -d demo --addons-path="../todo_app,./addons"`
