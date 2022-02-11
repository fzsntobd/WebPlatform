机器学习平台 开发指南
===================

## 目录

<!-- toc orderedList:0 depthFrom:1 depthTo:6 -->

- [机器学习平台 开发指南](#机器学习平台-开发指南)
	- [目录](#目录)
	- [1. 依赖](#1-依赖)
	- [2. 快速开始](#2-快速开始)
	- [3. URL列表](#3-url列表)
	- [4. API文档](#4-api文档)
		- [4.1 获取菜单](#41-获取菜单)
		- [4.2 文件操作](#42-文件操作)
			- [4.2.1 上传文件](#421-上传文件)
			- [4.2.2 获得用户已有文件列表](#422-获得用户已有文件列表)
			- [4.2.3 获得用户已有文件](#423-获得用户已有文件)
			- [4.2.4 重命名用户已有文件](#424-重命名用户已有文件)
			- [4.2.5 删除用户已有文件](#425-删除用户已有文件)
		- [4.3 流程处理](#43-流程处理)
			- [4.3.1 执行流程](#431-执行流程)
			- [4.3.2 执行状态](#432-执行状态)
			- [4.3.3 发布or保存流程](#433-发布or保存流程)
			- [4.3.4 发布的流程的开关](#434-发布的流程的开关)
			- [4.3.5 执行发布的流程](#435-执行发布的流程)
				- [方法一:  需要登录（平台上使用）](#方法一-需要登录平台上使用)
				- [方法二：不需要登录（作为web服务供其他app使用）](#方法二不需要登录作为web服务供其他app使用)
			- [4.3.6 执行状态](#436-执行状态)
	- [5. 工作流文档](#5-工作流文档)
		- [5.1 数据源节点](#51-数据源节点)
		- [5.2 预处理节点](#52-预处理节点)
		- [5.3 模型训练节点](#53-模型训练节点)
		- [5.4 模型使用节点](#54-模型使用节点)
		- [5.5 分支节点](#55-分支节点)
	- [6. 数据库](#6-数据库)
		- [6.1 角色表role](#61-角色表role)
		- [6.2 用户表user](#62-用户表user)
		- [6.3 用户角色关系表user_roles](#63-用户角色关系表user_roles)
		- [6.4 数据集表dataset](#64-数据集表dataset)
		- [6.5 模型表model](#65-模型表model)
		- [6.6 流程表workflow](#66-流程表workflow)
		- [6.7 菜单表menu](#67-菜单表menu)
	- [7. UML用例图](#7-uml用例图)
		- [7.1 顶层用例图](#71-顶层用例图)
		- [7.2 文件管理用例图](#72-文件管理用例图)
		- [7.3 流程处理用例图](#73-流程处理用例图)
		- [7.4 数据库管理用例图](#74-数据库管理用例图)
	- [8. 历史版本](#8-历史版本)

<!-- tocstop -->


-------------------

## 1. 依赖
- Flask
- Flask-Amdin
- Flask-Cors
- Flask-Migrate
- Flask-RESTful
- Flask-Script
- Flask-SQLAlchemy
- Flask-User
- paramiko
- sklearn
- xmltodict

## 2. 快速开始
1. 初始化数据库

  * 在 **app/settings.py** 的设置好数据库的 **ip**, **端口**, **用户名**, **密码**, **数据库名** 并在数据库里建好该名称的数据库。
  * 执行 `python manage.py db init` 将 **app/models.py** 里的数据库模版初始化。
  * 执行 `python manage.py db migrate` 将数据库构建起来。

    注：可通过查询数据库查看是否构建成功。

2. 运行后台程序

  * 执行 `python runserver.py` 即可。

## 3. URL列表

URL|内容
-|-
/login|登录界面
/|操作面板，使用平台基本功能
/amdin|管理员界面，管理数据库

## 4. API文档

返回错误的统一格式为:
```
{
      "result":"fail",
      "error":错误码,
      "str":内容
}
```

以下是返回正确的格式。

### 4.1 获取菜单
方法: **get** 到 **/getMenu**

返回json:
```
{
  "result":"success",
  "类型1":具体列表[str1, str2, ...],
  "类型2":具体列表[str1, str2, ...],
  ...
}
```

### 4.2 文件操作
#### 4.2.1 上传文件
方法: **post** 到 **/ml/uploadFile**

属性：
* **file**: 文件
* **ext**: 扩展名(字符串)
* **header**：是否有属性名( **0** 代表没有, **1** 代表有)
* **split**：数据分隔符(字符串)

返回json：
```
{
  "result":"success",
  "filePaht":"文件地址",
  "attributes":属性列表,
  "content":文件内容
}
```

#### 4.2.2 获得用户已有文件列表
方法: **post** 到 **/ml/getFiles**

属性：
* **filetype**: 文件类型( `data` 数据， `model` 模型， `workflow` 流程)

返回json:
```
{
  "result":"success",
  #data or model
  #"filenameList":["","",.....]//文件名列表
  #workflow
  #"filenameList":{
    "publishList":["","",.....],
    "unpublishList":["","",.....]
  }//文件名列表
}
```

#### 4.2.3 获得用户已有文件
方法: **post** 到 **/ml/getFile**

属性：
* **filename**: 文件名(字符串)
* **filetype**: 文件类型( `data` 数据， `model` 模型， `workflow` 流程)

返回json：
```
{
  "result": "success",
  "filePath":获取结果（获取成功的话是会地址）,
  #如果获取的data
  #"attributes":属性列表,（如果获取的data才会有）,
  #"content":文件内容
  #"param":{
            'header':datafile.header,
            'split':datafile.split,
        }
  #如果获取的model
  #"modelImage":模型图片，没有为空
}
```

#### 4.2.4 重命名用户已有文件
方法: **post** 到 **/ml/renameFile**

属性：
* **oldname**: 旧文件名(字符串)
* **filetype**: 文件类型( `data` 数据， `model` 模型， `workflow` 流程)
* **newname**: 新文件名(字符串)

返回json：
``` json
{
  "result": "success",
}
```

#### 4.2.5 删除用户已有文件
方法: **post** 到 **/ml/delFile**

属性：
* **filename**: 文件名(字符串)
* **filetype**: 文件类型( `data` 数据， `model` 模型， `workflow` 流程)

返回json：
``` json
{
  "result": "success",
}
```

### 4.3 流程处理
#### 4.3.1 执行流程
方法: **post** 到 **/ml/execute**

属性：
* **flow**: 对应的xml(字符串)
* **start**: 开始的id(选填)
* **end**: 结束的id(选填)

返回json：
```
{
  "result":"success",
  "workflow":"workname"(正在执行的流程临时名)
}
```

#### 4.3.2 执行状态
方法: **get** 到 **/ml/getState?workflow=`workname`**

属性：
* **workflow**: 正在执行的流程临时名

返回json：
```
{
  "result":"success",
  "idlist":[id1,id2,....],
  "state":"running" or "finished", （注：如果请求到了"finished"，那么流程已经结束并被释放掉了，workname作废）
  "id1":{
    具体内容查看《工作流文档》
  },
  "id2":{
    具体内容查看《工作流文档》
  }
  ...
}
```

#### 4.3.3 发布or保存流程
方法: **post** 到 **/workflow/publish**

属性：
* **flow** : 对应的xml
* **enable** : `True` or `False`(是否激活)

返回json:
```
{
  "result":"success",
}
```

#### 4.3.4 发布的流程的开关
方法: **post** 到 **/workflow/switch**

属性：
* **flowName** : 流程名
* **enable** : `True` or `False`(是否激活)

返回json:
```
{
  "result":"success",
}
```

#### 4.3.5 执行发布的流程
##### 方法一:  需要登录（平台上使用）
方法: **post** 到 **/workflow/execute**

属性：
* **flowName** : 流程名

返回json:
```
{
  "result":"success",
  "workflow":正在执行的流程临时名
}
```

##### 方法二：不需要登录（作为web服务供其他app使用）
方法: **get** 到 **/workflow/`<username>`/`<workflow>`**
>如用户admin的工作流demo:/workflow/admin/demo

返回json:
```
{
  "result":"success",
  "workflow":正在执行的流程临时名
}
```

#### 4.3.6 执行状态

方法: **get** 到 **/workflow/getState?workflow=`workname`**

属性：
* **workflow**: 正在执行的流程临时名

返回json：
```
{
  "result":"success",
  "idlist":[id1,id2,....],
  "state":"running" or "finished", （注：如果请求到了"finished"，那么流程已经结束并被释放掉了，workname作废）
  "id1":{
    具体内容查看《工作流文档》
  },
  "id2":{
    具体内容查看《工作流文档》
  }
  ...
}
```

## 5. 工作流文档
### 5.1 数据源节点

属性：
* **type** : "`dataSource`"
* **action** : "`ReadData`"
* **fileName** : "`文件名`"

返回state:
```
{
      'filePath':文件路径,
      'content':内容,
      'param':{
          'header':是否有属性名,
          'split':分隔符,
          'attributes':属性名,
      }
}
```

### 5.2 预处理节点

属性：
* **type** : "`Preprocessing`"
* **cols** : 要用的“列”的列表，如果有属性为属性名，如果无属性为第几列（从0开始）
* **action** : 不同的action有不同的 **preAlg** 属性的可填列表，甚至有不同的附加属性
  * "`PrePro`"
    * **preAlg** 列表为 `missing_del`,`missing_mean`,`missing_insert`,`missing_mode`,`normalization_min_max`
  * "`Discretization`"
    * **preAlg** 列表为 `discretization`
    * **cutPoints+cols里对应的值** : "`分割节点`"
    * **labels+cols里对应的值** : "`分割名`"
* **preAlg** : 预处理算法， 受 **action** 影响

返回state:
```
{
      'filePath':文件路径,
      'content':内容,
      'param':{
          'header':是否有属性名,
          'split':分隔符,
          'attributes':属性名,
      }
}
```

### 5.3 模型训练节点

属性：
* **type** : "`trainModel`"
* **action** : "`Train`"
* **alg** : "`训练算法名`"
* 以及其他算法对应参数

返回state:
```
{
      'modelPath':模型地址,
      'imagePath':图片地址,
}
```

### 5.4 模型使用节点

属性：
* **type** : "`modelApplier`"
* **action** : "`UseModel`"
* 以及其他算法对应参数

返回state：
```
{
      'predictResult':该模型对应的结果
}
```

### 5.5 分支节点
* **type** : "`swith`"
* **action** : "`Swith`" 待定，不同的分支条件判断方法会有不同的action

  注：可以指出的多条线，每条线上面都需要设置条件

## 6. 数据库
### 6.1 角色表role
属性名|数据类型|可否为空|主键/外键|其他
-|-|-|-|-
id|int|否|主键||
name|varchar(50)|否||unique

### 6.2 用户表user
属性名|数据类型|可否为空|主键/外键|其他
-|-|-|-|-
id|int|否|主键||
username|varchar(50)|否||unique
password|varchar(255)|否|||
is_enabled|boolean|否||默认True

### 6.3 用户角色关系表user_roles
属性名|数据类型|可否为空|主键/外键|其他
-|-|-|-|-
id|int|否|主键||
user_id|int|否|外键||
role_id|int|否|外键||

### 6.4 数据集表dataset
属性名|数据类型|可否为空|主键/外键|其他
-|-|-|-|-
id|int|否|主键||
name|varchar(50)|否||unique
file_type|varchar(20)|否|||
path|varchar(256)|否|||
header|boolean|否|||
split|boolean|否|||
user_id|int|否|外键||

### 6.5 模型表model
属性名|数据类型|可否为空|主键/外键|其他
-|-|-|-|-
id|int|否|主键||
name|varchar(50)|否||unique
path|varchar(256)|否|||
image|varchar(256)|可|||
is_spark|boolean|否||默认False
user_id|int|否|外键||

### 6.6 流程表workflow
属性名|数据类型|可否为空|主键/外键|其他
-|-|-|-|-
id|int|否|主键||
name|varchar(50)|否||unique
path|varchar(256)|否|||
is_enabled|boolean|否||默认False
user_id|int|否|外键||

### 6.7 菜单表menu
属性名|数据类型|可否为空|主键/外键|其他
-|-|-|-|-
id|int|否|主键||
name|varchar(50)|否||unique
item_type|varchar(20)|否|||

## 7. UML用例图
### 7.1 顶层用例图

``` {plantuml}

@startuml
actor 管理员
participant 登录界面类
participant 用户管理类
database 数据库
管理员 -> 登录界面类: 发送用户信息
登录界面类 -> 用户管理类: 提交用户信息
用户管理类 -> 数据库: 查询用户信息是否正确
数据库 -> 用户管理类: 返回查询验证结果
用户管理类 -> 登录界面类: 返回验证结果
登录界面类 -> 管理员: 进入后台管理界面
@enduml

```

``` {plantuml}

@startuml
left to right direction
:管理员:
:普通用户:

rectangle 数据集管理用例{
  管理员 -> (数据集管理)
  普通用户 -> (数据集管理)
  数据集管理 ..> (数据集上传) : 包含
  数据集管理 ..> (数据集查看) : 包含
  数据集管理 ..> (数据集删除) : 包含
  数据集管理 ..> (数据集重命名) : 包含
}
@enduml

```

``` {plantuml}

@startuml
left to right direction
:管理员:
:普通用户:

rectangle 流程图管理用例{
  管理员 -> (流程图管理)
  普通用户 -> (流程图管理)
	流程图管理 ..> (流程图制作) : 包含
  流程图管理 ..> (流程图保存) : 包含
  流程图管理 ..> (流程图查看) : 包含
	流程图管理 ..> (流程图修改) : 包含
  流程图管理 ..> (流程图删除) : 包含
  流程图管理 ..> (流程图重命名) : 包含
	流程图管理 ..> (流程图执行) : 包含
}
@enduml

```

``` {plantuml}

@startuml
left to right direction
:管理员:
:普通用户:

rectangle 模型管理用例{
  管理员 -> (模型管理)
  普通用户 -> (模型管理)
  模型管理 ..> (模型查看) : 包含
  模型管理 ..> (模型删除) : 包含
  模型管理 ..> (模型重命名) : 包含
}
@enduml

```

``` {plantuml}

@startuml
left to right direction
:管理员:

rectangle 日志管理用例{
  管理员 -> (日志管理)
  日志管理 ..> (日志查看) : 包含
  日志管理 ..> (日志清空) : 包含
}
@enduml

```

``` {plantuml}

@startuml
left to right direction
:管理员:
:普通用户:

rectangle {
  管理员 -> (文件管理)
  普通用户 -> (文件管理)
  文件管理 ..> (文件提交) : 包含
  文件管理 ..> (文件查看) : 包含
  文件管理 ..> (文件删除) : 包含
  文件管理 ..> (文件重命名) : 包含
  管理员 -> (流程处理)
  普通用户 -> (流程处理)
  流程处理 ..> (流程设计) : 扩展
  流程处理 ..> (流程执行) : 扩展
  流程处理 ..> (流程管理) : 扩展
  管理员 -> (数据库管理)
  数据库管理 ..> (算法管理) : 扩展
  数据库管理 ..> (用户管理) : 扩展
  数据库管理 ..> (角色管理) : 扩展
  数据库管理 ..> (权限管理) : 扩展
  管理员 -> (日志管理)
}
@enduml

```

本平台的整体功能如顶层用例图所示，其主要功能模块划分以及各模块的主要功能描述如表所示。
模块名称|功能描述
-|-
文件管理|对用户的数据集以及模型进行管理。
流程处理|包括流程的设计、管理以及执行。流程管理包括对流程的增删改查以及流程的部署，流程的部署即将流程发布为一个web服务。
数据库管理|对用户、算法、权限及角色进行管理。用户管理包括添加用户、修改信息、修改密码等。算法管理即可以对可以选用的算法进行添加。
日志管理|运行日志管理和错误日志管理。


### 7.2 文件管理用例图

``` {plantuml}

@startuml
left to right direction
:管理员:
:普通用户:

rectangle 文件管理用例{
  管理员 -> (文件管理)
  普通用户 -> (文件管理)

  文件管理 ..> (文件提交) : 包含
  文件管理 ..> (文件查看) : 包含
  文件管理 ..> (文件删除) : 包含
  文件管理 ..> (文件重命名) : 包含
}
@enduml

```

文件管理所有的功能是管理 **数据集**、**模型**、**流程** 的通用功能，是对 **数据集**、**模型**、**流程** 的相关数据及文件进行管理。其中 **数据集** 是靠上传，**模型** 是训练后自动生成，**流程** 是流程设计后保存产生。

### 7.3 流程处理用例图

``` {plantuml}

@startuml
left to right direction
:管理员:
:普通用户:

rectangle 流程处理用例{
  管理员 -> (流程处理)
  普通用户 -> (流程处理)
  流程处理 ..> (流程设计) : 扩展
  流程处理 ..> (流程执行) : 扩展
  流程处理 ..> (流程管理) : 扩展
  流程管理 ..> (流程保存) : 包含
  流程管理 ..> (流程删除) : 包含
  流程管理 ..> (流程重命名) : 包含
  流程管理 ..> (流程部署) : 包含
}
@enduml

```

流程处理主要包括**流程设计**、**流程执行**、**流程管理**。
* 流程设计
	将各类图标根据数据的走向正确的连接成可执行的流程。
* 流程执行
   将已经保存的流程执行。或发布出去的服务被调用。
* 流程管理
	对流程进行保存、修改、删除等基本管理，以及对流程进行部署管理。

### 7.4 数据库管理用例图

``` {plantuml}

@startuml
left to right direction
:管理员:

rectangle 数据库用例{
  管理员 -> (数据库管理)
  数据库管理 ..> (算法管理) : 扩展
  算法管理 ..> (算法添加) : 包含
  算法管理 ..> (算法删除) : 包含
  算法管理 ..> (算法修改) : 包含
  算法管理 ..> (算法查看) : 包含
  数据库管理 ..> (用户管理) : 扩展
  用户管理 ..> (用户添加) : 包含
  用户管理 ..> (用户删除) : 包含
  用户管理 ..> (用户修改) : 包含
  用户管理 ..> (用户查看) : 包含
  数据库管理 ..> (角色管理) : 扩展
  角色管理 ..> (角色添加) : 包含
  角色管理 ..> (角色删除) : 包含
  角色管理 ..> (角色修改) : 包含
  角色管理 ..> (角色查看) : 包含
  数据库管理 ..> (权限管理) : 扩展
  权限管理 ..> (权限添加) : 包含
  权限管理 ..> (权限删除) : 包含
  权限管理 ..> (权限修改) : 包含
  权限管理 ..> (权限查看) : 包含
}
@enduml

```

数据库管理是对数据库进行综合管理，主要包括 **用户管理**、**权限管理**、**角色管理**、**算法管理**。
* 用户管理
	主要包括添加删除用户，以及基础信息的变更以及个人密码的修改。
* 权限管理
   管理用户与角色的对应关系。不同用户拥有不同的权限。且不同等级用户获取方式也不同。其中系统默认一个管理员用户，其他用户由默认管理员用户添加生成。
* 角色管理
	用户权限设为不同等级，不同等级权限收到不同程度的系统使用限制，而不同角色的用户享有不同等级的权限。
管理员用户可对所有等级权限进行添加、修改、删除等管理操作。
* 算法管理
	系统中有一定的预设算法。算法管理可对算法进行添加和修改，实现功能扩展。

## 8. 历史版本

> 2016年12月8日 15:41:29 更新至1.1.4版本
1. 菜单数据库， 即前端菜单从写在数据库里，从数据库里获取
2. admin， 即数据管理平台，管理员后台

> 2016年12月6日 17:26:08 更新至1.1.3版本
1. 文件位置，将模型的位置放到安全位置。
2. 将app.tools.userManager的当前用户(CurrentUser)类添加了按用户名实例化功能，同时保留
根据当前已登录用户来实例化的功能。
3. 添加发布的服务的调用方法。使用/workflow/<username>/<workflow> 调用
4. 给出一个数据、模型、流程重命名的接口
5. 文件删除中的实际删除处理

> 2016年12月6日 13:32:18 更新至1.1.2版本
1. 添加4个预处理算法
2. 更改预处理调用及传参方式

> 2016年12月2日 11:45:55 更新至1.1.1版本
* 对上个版本和前端交互做了调整，功能实现稳定版本。

> 2016年11月30日 22:25:18 更新至1.1.0版本
* 重构。填了以前不少的坑，解决了大量的矛盾，这里更新内容太多了，不一一列举

> 2016年11月22日 10:11:15 更新至1.0.6版本
1. 增加workflows插件
2. 增加用户所有的数据、模型、流程管理

> 2016年10月28日 11:25:38 更新至1.0.4版本
1. svm修正
2. 画图bug修正
3. 添加缺失值处理，含使用平均值来替代缺失值，使用插值法估计并替换缺失值，删除缺失值数据
