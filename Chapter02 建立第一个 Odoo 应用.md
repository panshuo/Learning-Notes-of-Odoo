###Chapter02 建立第一个 Odoo 应用

####Odoo遵循了类MVC结构, 接下来在实现一个To-Do应用的过程中会逐一解析这些不同的层
- model层, 定义了应用的数据结构
- view层, 描述用户界面
- controller层, 支撑应用的业务逻辑

#### 理解模块插件(附加组件)和应用
- 模块插件类似用于搭建Odoo应用的积木. 一个模块可以为Odoo添加新特性，或修改现存的功能。
- 应用是为Odoo添加核心功能的一种方式.

*理解：模块是应用的一个子集，一个应用可能是添加一个新的重要功能给Odoo，模块有可能只是对现存的功能进行修改。
一个模块是不是应用程序在manifest文件中定义*

对于业界通行的规则来说，直接修改一个现存的模块的代码是一种不好的行为。尤其对Odoo的的官方模块更不该如此
这样会造成原生代码和修改过的代码互相之间不能明确区分开。并且在升级的时候Odoo会覆盖修改过的代码造成一些难以解决的困难。
Odoo最主要的理念之一就是“继承机制”，允许自定义的模块继承已存在的模块，而且可以从不同的层面比如数据模型，业务逻辑，或者用户界面来继承。

#### 包含一个 '__manifest__.py' 文件的目录就是一个模块。
在早期版本中这个描述文件的名字是 __openerp__.py 虽然这个名字还能被兼容但是不赞成使用。
这个Odoo模块需要能被Python导入，所以 __init__.py 文件也是必须的。
模块的目录名就是它在Odoo中的正式名字，在这里将会使用 todo_app

*注意：正式名字必须是一个合法的Python标识符，以英文字母开始并且只能包含英文字母，数字，下划线*

#### 建立模块的基本骨架
为了保持代码整洁，建立一个新的模块目录
`$ mkdir ~/odoo-dev/custom-addons'

然后建立Python模块的 '__init__.py' 文件

`$ mkdir ~/odoo-dev/custom-addons/todo_app`

`$ touch ~/odoo-dev/custom-addons/todo_app/__init__.py`


现在应该添加一个 '__manifest__.py' 文件在' __init__.py' 文件的同级目录
添加以下内容：

`{
'name': 'To-Do Application',
'description': 'Manage your personal To-Do tasks.',
'author': 'Daniel Reis',
'depends': ['base'],
'application': True,
}`

depends 应该是一个Odoo模块依赖包的列表.在安装这个模块的时候Odoo会自动安装它们.虽然这不是一个强制使用的属性，但是建议总是使用它。
而且如果没有特殊的依赖需求，那么也应该指一个核心的 base 模块。请在这里应该仔细检查并确保所有依赖都在这里被显式的传入了。不然在全新安装一个数据库的时候（由于缺少依赖组件）可能会失败，也有可能发生导入错误。现在使用指定的模块路径来启动服务器
`$ cd ~/odoo-dev`

`$ ./odoo/odoo-bin -d todo --addons-path="custom-addons,odoo/addons" --save`

仔细观察服务器日志。应该会有一条 INFO 日志 ? odoo: addons paths:[...] 包含了我们的 custom-addons 目录

![img](http://teonline.top/images/odoo01.png)

#### 创建数据模型 (Model Layer)
Model用来描述商业对象，比如机会，销售订单或合作伙伴（顾客，供货商等等）。
一个Model拥有一系列的属性并可以定义它的特定事物。Model是通过从Odoo模板类继承来的Python类来实现。
在安装或者升级模块的时候Odoo自动处理这些，将它们直接翻译成数据库对象。这种机制叫做对象关系映射 (ORM).

Odoo 开发指导建议关于 Model 的Python文件应该放在一个名字为 models 的子目录内。为了简单起见, 这里不遵照这个建议
创建一个 todo_model.py 放在 todo_app 模块的根目录并添加以下代码：

`from odoo import models, fields`

`  class TodoTask(models.Model):`

`  _name = 'todo.task'`

`  _description = 'To-do Task'`

`  name = fields.Char('Description', required=True)`

`  is_done = fields.Boolean('Done?')`

`  active = fields.Boolean('Active?', default=True)`

#### 添加菜单项目 (View Layer)
待编辑

#### 添加业务逻辑 (Business Logic Layer)
为之前建立的数据Model类添加类方法，以便在前端界面按钮被点击的时候能被调用。首先需要引入Odoo的API

`from odoo import models, fields, api`

Toggle Done button 的操作很简单: 只要切换 Is Done? 这个标志的状态就行了。 使用 @api.multi 装饰器对表记录进行这个逻辑操作。
在这里 self 代表了表中整个的数据集，我们需要遍历整个数据集中的每条记录：

`@api.multi`

` def do_toggle_done(self):`

` for task in self:`

` task.is_done = not task.is_done`

` return True`

虽然这个类方法在这里确实不需要返回值，但是我们还是应该让它返回一个True。因为客户端使用 XML-RPC 来调用这个方法
这个协议不支持服务器方法返回一个 None 值。
对于 Clear All Done 按钮，需要深入一点点。这个操作需要查找 active 和 is_done 的值都为 True 所有记录。然后把它们设置为非活动状态

`@api.model`

`def do_clear_done(self):`

`  dones = self.search([('is_done', '=', True)])`

`  dones.write({'active': False})`

`  return True`

#### 添加访问安全保护
安全保护信息需要提供一个数据文件给模块来导入到 ir.model.access Model
将为 employee 组在这个 Model 上添加全部访问权限。基本所有用户都属于 Employee 这个最基本的访问控制组。
这个数据文件将使用 CSV 文件格式，并命名为 security/ir.model.access.csv 内容略。

#### 行级别的访问安全保护

待编辑

#### 添加应用图标
将图标文件放置在 /static/description/icon.png 再升级应用

#### 总结
本章从创建一个新模块开始，覆盖了最常使用的几个元素：
- Model 
- 三种基本视图类型 (form, list, search)
- 通过 Model 类的方法实现的业务逻辑
- 访问权限控制

*注意： 在模型里面添加新的列时，总是需要升级模块。修改 Python 代码后,包括 manifest 文件，需要重启服务器。改变了 XML 或 CSV 也需要升级模块
总之碰到任何疑难杂症都可以试试这两个操作。*
