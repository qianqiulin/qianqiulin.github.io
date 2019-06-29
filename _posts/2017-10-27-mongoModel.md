---
layout:     post
title:      "MongoDB模式设计"
subtitle:   "mongo"
date:       2017-10-27 14:17:30
author:     "TyurinTsien"
header-img: "img/post-bg-mongo.jpg"
catalog: true
tags:
    - MongoDB
---

# 工单自动化处理平台

## MongoDB数据库设计

#### 原子能力数据存储模型设计

1.字段
   
	+ 包
		- id
		- 包名（提供接口系统名，不能中文）
		- 系统名称（提供接口的系统名称）
		- 包描述信息
		- 创建、修改人
		- 创建、修改时间
		- 操作类型：0：新建；1：修改。
		- 状态：0：待审核；1：审核通过；2：审核不通过。
	+ 文件
		- id
		- 文件名（原则上一个文件一个类，默认类名和文件名相同）
		- 文件描述信息
		- 创建、修改人
		- 创建、修改时间
		- 初始化代码（“def __init__()”）
		- 操作类型：0：新建；1：修改。
		- 状态：0：待审核；1：审核通过；2：审核不通过。
	+ 方法
		- id
		- 方法函数名
		- 方法名称
		- 方法描述信息
		- 创建、修改人
		- 创建、修改时间
		- 代码块
		- 入参模板
		- 出参模板
		- 操作类型：0：新建；1：修改。
		- 状态：0：待审核；1：审核通过；2：审核不通过。

2. 存储位置
    + 包
    + 数据库：mixFunc
    + 包-文件数据表：
        - 历史文档：hisMixFunc
        - 当前文档：nowMixFunc
	+ 包表：
		- 历史文档：hisPackage
		- 当前文档：nowPackage
	+ 方法文件表：
		- 历史文档：hisMethodFile
		- 当前文档：nowMethodFile
	+ 方法表：
		- 历史方法：hisMethod
		- 当前方法：nowMethod

3. 存储结构如下
+ 包
```
{
	"__id":"",
	"packageName":"包名（提供接口系统名，不能中文）",
	"systemName":"提供接口的系统名称",
	"packageDesc":"包描述信息",
	"packageUser":"创建/修改人",
	"packageTime":"创建/修改时间",
	"actionType":"0：新建；1：修改"
	"packageStatus":"0：待审核；1：审核通过；2：审核不通过",
	"methodFiles":[] #默认为空
}
```
+ 文件
```
{
	"fileName":"文件名",
	"fileDesc":"文件描述信息",
	"fileUser":"创建/修改人",
	"fileTime":"创建/修改时间",
	"initCode":"初始化代码",
	"actionType":"0：新建；1：修改",
	"fileStatus":"0：待审核；1：审核通过；2：审核不通过"
	}
```
+ 方法
```
{
	"__id":"",
	"methodName":"方法名",
	"methodDesc":"方法描述信息",
	"methodUser":"创建/修改人",
	"methodTime":"创建/修改时间",
	"inModel":"入参模板",
	"outModel":"出参模板",
	"code":"代码块",
    "actionType":"0：新建；1：修改",
	"methodStatus":"0：待审核；1：审核通过；2：审核不通过",
}		
```

#### 处理逻辑脚本存储模型设计
+ 用户自己写的工单处理逻辑脚本
1. 字段
    + 处理脚本
        - id
        - 文件名
        - 工单类型
        - 创建/修改人
        - 创建/修改时间
        - 操作类型：0：新建；1：修改
        - 状态：0：待审核；1：审核通过；2：审核不通过
        - 代码

2. 存储位置
    + 最新的文档存储在nowDealFile这个colleciton里，当修改了nowDealFile里的文档，需要将修改之前的那个文档存储在hisDealFile里，nowDealFile里只保存最新的文档
    + 数据库：dealFile
    + 数据表：
        - 历史文档：hisDealFile
        - 当前文档：nowDealFile

3. 存储结构如下
```
{
	"__id":"",
	"dealFileName":"处理逻辑文件名",
	"dealFileDesc":"处理逻辑描述",
    "dealFileType":"处理工单类型",
	"dealFileUser":"创建/修改人",
	"dealFileTime":"创建/修改时间",
	"actionType":"0：新建；1：修改"
	"dealFileStatus":"0：待审核；1：审核通过；2：审核不通过",
    "dealCode":""
}
```

#### 工单处理记录数据存储模型设计
+ 工单处理逻辑过程记录
1. 字段：
	+ 处理过程
		- id
		- 工单id
		- 处理动作
		- 处理时间
		- 处理描述
		- 处理结果（成功、失败、A端成功、Z端失败）
		- 接口日志
2. 存储位置：
	+ 数据库：appLogs
	+ 数据库：dealLogs
3. 存储结构如下
```
{
	"__id":"",
	"dealLogName":"处理逻辑文件名",
	"billID":"工单id",
	"dealLogAct":"处理动作",
    "dealLogTime":"处理时间",
	"dealLogDesc":"处理描述",
	"dealLogResult":"处理结果",
	"itfLog":[]	
}
```

#### 工单数据存储模型设计
电子工单发过来的工单信息为xml格式的，直接转为json格式再加一个字段billStatus（工单状态）。
1. 存储设计接收工单信息判断工单类型，不同工单类型存到不同collection里，用户自己写脚本读取数据库里的工单并调用相对应的处理逻辑脚本。
2. 存储位置
	+ 数据库：billInfo
	+ 数据表：工单类型英文+s

#### 接口调用数据存储模型设计
+ 本系统访问外部系统提供的接口调用记录、外部系统访问本系统提供的接口调用记录
1. 字段：
	+ 接口日志
		- id
		- 发送方
		- 接收方
		- 服务名
		- 发送时间
		- 接收时间
		- 入参
		- 出参

2. 存储位置
    + 数据库：appLogs
    + 数据表：itfLogs

3. 存储结构
```
{
	"__id":"",
	"sender":"发送方",
	"receive":"接收方",
	"serverName":"服务名",
	"sendTime":"发送时间",
	"receiveTime":"接收时间",
	"input":"入参",
	"output":"出参"
}
```

#### 系统安全及用户数据存储模型设计
存储系统用户基本信息、权限

##### 基本信息
1. 字段：
	+ 用户信息
		- 用户id
		- 用户名
		- 密码
		- 公司
		- 部门
		- 手机
		- 座机
		- 邮箱
		- 用户状态
		- 角色
2. 存储位置
	+ 数据库：safeData
	+ 数据表：userInfo

3. 存储结构：
```
{
	"__id":"",
	"userName":"用户名",
	"userPwd":"密码",
	"company":"公司",
	"part":"部门",
	"phone":"手机",
	"tel":"座机",
	"email":"邮箱",
	"userStatus":"用户状态",
	"roles":[]
}
```

##### 角色信息
1. 字段
	+ 角色
		- 角色id
		- 角色名
		- 角色状态
		- 权限资源
2. 存储位置
	+ 数据库：safeData
	+ 数据表：roles
3. 存储结构
```
{
	"__id":"",
	"roleName":"发送方",
	"roleStatus":"接收方",
	"resources":[]
}
```

##### 权限资源
1. 字段
	+ 权限资源
		- id
		- 资源名称（eg:原子功能）
		- 权限类别（eg:添加、删除、修改、审核。。。）
		- 权限状态
		- url
2. 存储位置
	+ 数据库：safeData
	+ 数据表：resources
3. 存储结构
```
{
	"__id":"",
	"resourcesName":"资源名称",
	"actionType":"权限类别",
	"resourceStatus":"权限类别",
	"url":"发送时间"
}
```

#### 总结

##### 数据库数据表按功能点汇总

1. 原子能力
	+ 数据库：mixFunc
    + 包-文件数据表：
        - 历史文档：hisMixFunc
        - 当前文档：nowMixFunc
	+ 包表：
		- 历史文档：hisPackage
		- 当前文档：nowPackage
	+ 方法文件表：
		- 历史文档：hisMethodFile
		- 当前文档：nowMethodFile
	+ 方法表：
		- 历史方法：hisMethod
		- 当前方法：nowMethod
2. 处理逻辑
	+ 数据库：dealFile
    + 数据表：
        - 历史文档：hisDealFile
        - 当前文档：nowDealFile
3. 日志记录
	+ 数据库：appLogs
	+ 处理逻辑日志：
		- 数据表：dealLogs
	+ 接口调用日志：
		- 数据表：itfLogs
4. 工单信息
	+ 数据库：billInfo
	+ 数据表：工单类型英文+s
5. 用户基本信息、角色及资源
	+ 数据库：userData
	+ 用户信息：
		- 数据表：userInfo
	+ 角色：
		- 数据表：userData
	+ 资源：
		- 数据表：roles

##### 数据库数据表汇总

+ mixFunc
	- hisMixFunc
	- nowMixFunc
	- hisPackage
	- nowPackage
	- hisMethodFile
	- nowmethodFile
	- hisMethod
	- nowMethod
+ dealFile
	- hisDealFile
	- nowDealFile
+ billInfo
	- 工单类型英文+s
+ appLogs
	- dealLogs
	- itfLogs
+ userData
	- userInfo
	- roles
	- resource

#### 分工

+ 原子功能：钱秋林、吴敏民（mixFunc）
+ 处理文件：钱秋林、吴敏民（dealFile）
+ 应用日志：吴书鑫、郑金钢（appLogs）
+ 系统权限：吴书鑫、郑金钢（userData）

+ 工单存储：一起（billInfo）


## 平台功能设计