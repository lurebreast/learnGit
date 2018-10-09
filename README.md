# SzFramework

## Index
1. 介绍
2. 配置解读
3. 代码结构及部署结构
4. 工具脚本简介及使用
5. 简单使用范例
6. 核心模块
7. 请求与返回
8. 系列约定
9. 释疑
10. Profile工具
11. 日志系统 ~~11. Redis Log 已废弃~~
12. 错误处理

![Aaron Swartz](https://github.com/younghz/Markdown/raw/master/resource/Aaron_Swartz.jpg)
---

## 1. 介绍
SzFramework框架是一个，轻量级、简单模型、高并发请求的框架。其初衷是用来实现social network游戏这种模型较为简单，并发请求较多的需求的应用。此外，它也适用于开发轻量级的RPC请求服务器，以及制作简单的网站，工具站，等等。其模型映射的功能特性，能让开发者非常高效地进行开发。

---

## 2. 配置解读
### 2.1 配置文件格式
* 所有的配置文件都是PHP代码文件
* 以`config.php`为文件扩展名：{configname}.config.php
* 其格式为：

		<?php
		return array(
		    // 内容在这里
		);

* 使用 $configs = include {absoluteConfigFilePath}.config.php

### 2.2 配置文件路径
所有的配置文件都放在各自根目录的config文件夹下：    
{frameworkRoot|appRoot|moduleRoot}/config/{configname}.config.php

### 2.3 全局共有配置文件
#### 2.3.1 autoload.config.php

	<?php
	return array(
	    '类名' => '文件系统中该类的相对路径',
	    ...
	);

#### 2.3.2 exception.config.php

	<?php
	return array(
	    '错误id' => '错误信息文本',
	    ...
	);

### 2.4 框架配置文件
#### 2.4.1 vendor.config.php
第三方类库的加载路径配置：

* 类名 => 第三方类库相对路径

#### 2.4.2 publish.config.php
* PLATFORM：string，平台名称，定义在`SzPublish::$PLATFORMS`
* ENV：string，`DEV` | `LIVE` | `STAGING`，仅在DEV环境，多开发者目录功能会生效，其他环境系统会默认寻找`main`目录下的代码
* WEB_HOST：string，web服务器域名，不含http或https协议
* APP_CANVAS：string，app的页面url
* APP_ID：string，app的id
* APP_SECRET：string，app的秘钥
* ONLINE_VER：string，对外开放的在线版本号
* PREVIEW_VER：string，仅对开发者和测试者开放的版本号
* PREVIEW_PERCENT：int，0-100，开放玩家进入preview版本的比率
* THROTTLE_PERCENT：int，0-100，节流闸比率，在高峰时段限制玩家进入的开关，0表示节流闸并不启用。节流闸只会在`上线版本`和`预览版本`版本号相同的时候发挥作用，即非版本发布时期起作用
* DEV_LIST：array，array(id, id, …)，开发者和测试人员的名单
* WHITE_LIST：array，array(id, id, …)，其他的白名单
* IP_WHITE_LIST：array，array(ip, ip, ...)，IP白名单，当这个数组为空的时候，SzPublish不会启动IP白名单验证，反之则会验证发送请求的IP地址

MAGIC_TOKEN：在通过SzPublish class的过程中，如果客户端上传平台对应的认证code为`52d0ee3c031dd`的话（facebook平台是通过signed_request字段上传），系统会默认该验证通过，即为magic token

### 2.5 应用配置文件
#### 2.5.1 app.config.php
* ENV：string，`DEV` | `LIVE` | `STAGING`
* LANG：string，e.g `en_us`、`zh_cn`、`zh_tw` …
* TIMEZONE：string，PHP的timezone设置，e.g `Asia/Shanghai` …
* JS_VER：int，浏览器前端附加的js版本号
* CSS_VER：int，浏览器前端附加的css版本号
* PHP_VER：string，后端服务器代码版本号
* VERSION：string，客户端代码版本号
* WEB_HOST：string，web服务器域名，不含http或https协议
* CDN_HOST：string，静态资源CDN域名，不含http或https协议
* PLATFORM：string，上线平台名
* CANVAS_URL：string，游戏实际访问的url
* CANVAS_NAME：string，游戏访问的名称
* GOOGLE_ID：string，Google Analytics账号ID
* GAME_ID：string，公司内部定义的游戏编号
* MAINTENANCE：boolean，服务器是否处于维护状态
* API_COMPRESS：boolean，是否在客户端和服务器之间的通讯消息上进行压缩
* BASE64_ENCODE：boolean，是否在通讯消息压缩之后再进行 base64_encode 或在收到消息解压之前进行 base64_decode
* ~~RESPONSE_JSON_NUMERIC_CHECK：boolean，是否在SzResponseManager的最终返回JSON格式中，强制将数字类型字符串转化为数字~~
* RESPONSE_JSON_OPTIONS：int，在SzResponseManager的最终返回的JSON格式中，json_encode()方法所使用的参数,具体配置参考：[options](http://php.net/manual/en/json.constants.php)
* NOTIFY_PERSIST_RESULT：boolean，是否需要在请求通讯的返回值中加入数据库persistence的更新结果
* PERSIST_RESULT_FILTER：array，数组格式非K/V的Map，内部全部都是 orm.config.php 配置文件的Key，也就是ORM名，表示这些ORM的数据将会在UPDATE字段中被过滤掉
* PERSIST_RESULT_FORCE_OBJECT：array，数组格式非K/V的Map，内部全部都是 orm.config.php 配置文件的Key，也就是ORM名，表示这些ORM的数据将会在UPDATE字段中被强制转换成对象
* PLATFORM_ON：boolean，是否启用对接平台的接口，如果为否的话，所有的平台接口都会返回空
* LOG_ERROR_TRACE：boolean，是否在set_error_handler设定的error错误处理的时候，打印出debug_backtrace的堆栈
* LOG_EXCEPTION_TRACE：boolean，是否在set_exception_handler设定的exception错误处理的时候，打印出debug_backtrace的堆栈
* API_REPEAT_CHECK：boolean, 是否开启api重发包功能.如果有些项目不需要该特性或者项目处于开发期,可以将这个常量值置为false来关闭框架对该特性的支持,默认值为false.
* API_REPEAT_LIMIT:int, 用来限制指定api可重发次数上限.API_REPEAT_CHECK为true时该值才生效.
* API_REPEAT_EXPIRE:int, 用来限制指定api的缓存数据的缓存有效期,单位为秒,API_REPEAT_CHECK为true时该值才生效.
* API_SIGN_SECRET:string, 指定api签名密钥,API_REPEAT_CHECK为true时该值才生效.
* FRAMEWORK_ROOT：string，框架部署的根目录的绝对路径
* FRAMEWORK_VER：string，应用使用的框架版本号
* MODULE_ROOT：string，功能模块部署的根目录的绝对路径
* MODULE_VERS：array，应用使用的模块名与对应的模块版本号

		array(
		    '模块名' => '模块版本号',
		    ...
		)

#### 2.5.2 cache.config.php
* CACHE_TYPE：string，当前使用的缓存类型 `Memcached` | `Redis`
* SECURE_CACHE_ENABLED：boolean，是否使用不启用LRU的安全缓存，用来存储一些比较敏感的，不允许被系统自动回收的信息
* 缓存池配置：array，缓存服务器的配置信息，下述所有的配置都是这个格式，`array(host, port)`

		array(
		    array('127.0.0.1', 6379),
		    array('127.0.0.1', 6380),
		    ...
		)

* App_Memcached：array，应用级别的memcached缓存配置
* Static_Memcached：array，安全的memcached缓存配置
* App_Redis：array，应用级别的redis缓存配置
* Static_Redis：array，安全的redis缓存配置

#### 2.5.3 controller.config.php
* PRE_HANDLERS：array，控制器前置处理类的定义

		array(
		    前置处理类名, ...
		)
		
* POST_HANDLERS：array，控制器后置处理类的定义

		array(
		    后置处理类名, ...
		)

#### 2.5.4 database.config.php
* BATCH_INSERT_THRESHOLD：int，批量插入阀值，高于这个值的单条插入`SQL`语句将会被统一转为批量插入`SQL`语句
* 单一数据库配置：array，实现了读写分离，多台读(slave)一台写(master)，作为一个集群，无论redis还是mysql或者其他类型的数据库，这个配置都是统一的。不使用的项，默认值都给null
		
		array(
		    'DB_NAME' => string，数据库名,
		    'READER_HOST' => array(
		        array('hostValue', 'portValue'),
		        ...
		    ),
		    'READER_USER' => string，读权限用户名,
		    'READER_PWD' => string，读权限用户密码,
		    'WRITER_HOST' => array('hostValue', 'portValue'),
		    'WRITER_USER' => string，写权限用户名,
		    'READER_PWD' => string，写权限用户密码,
		)

* SHARD_STRATEGY：string，`Fixed` | `Dynamic`，定义在`SzAbstractDbFactory::SHARD_TYPE_*`
	* Dynamic：每个用户的数据在第一次入库的时候都会被分配一个分片id，以后永久不变，该对应关系存储在redis中
	* Fixed：根据配置的数据库个数，进行取模操作，决定应该读取的分片id是多少，因此不得轻易改动数据库配置中的实例个数，如果需要改动个数，需要预先重新分配数据
* `Dynamic`策略配置：
	* SHARD_ESTIMATE_REGISTER_USER_COUNT：int，预估的游戏注册用户总量
	* SHARD_WEIGHT_`数据库类型`：array，该配置请参考`SzUtility::getRandomElementByProbability`，使用到的数据库类型都需要配

			array(
			    shardId => 抽到该shardId的权重值,
			    …
			)

* `数据库类型`：array，数据库集群配置

		array(
		    0 => 单一数据库配置, // shardId => 数据库配置
		    1 => ...
		    ...
		)

#### 2.5.5 logger.config.php
* LOG_LEVEL：string，日志级别，高于(含)该级别的日志才会被记录下来，可选项：
	* LOG_DEBUG(7)
	* LOG_INFO(6)
	* LOG_NOTICE(5)
	* LOG_WARNING(4)
	* LOG_ERR(3)
* LOG_TYPE：string，日志类型，`SYSLOG` | `FILELOG` ~~| `REDISLOG(已废弃)` ~~
* FILELOG：
	* LOG_FILE：string，日志文件位置，绝对路径带文件名
* SYSLOG：
	* LOG_IDENTITY：string，syslog日志记录时候显示的名称
	* LOG_FACILITY：string，syslog日志记录的设备，参考[openlog() Facilities](http://php.net/manual/en/network.constants.php)
* ~~REDISLOG：已废弃~~
    * ~~参考11章 Redis Log~~
* STATISTICS：统计相关的配置，现暂时安放在logger配置表里
    * LOG_HOST：string，统计日志服务器host
    * LOG_PORT：int，统计日志服务器port
    * CONN_TIMEOUT：float，socket连接超时
    * SEND_TIMEOUT：float，socket发送超时
    * COMPRESS：boolean，是否压缩开关
* LOG_FILTER：用于区分项目产生日志的环境情况，默认值为："UNLABELED"。配置格式："项目名_运行环境", e.g："FV_DEV"

#### 2.5.6 orm.config.php
orm映射配置文件，结构：

	array(
	    ormName => orm配置,
	    ...
	)
	
orm配置：

* table：string，数据库表名，不适用时默认值为null
* columns：array，表字段
	* 该配置中的字段名必须和数据库中完全一致
	* 该配置中字段顺序必须和数据库中完全一致
* autoIncrColumn：int，自增长字段的字段id，字段id就是columns字段中的数组键，没有则给null
* diffUpColumns：array，差值更新字段列表，列在这个列表里的字段在update的时候，会使用`column = column +/- int`这样的格式进行更新。注意，仅能使用在数字类型的字段上。没有则给空数组，array()

		array(
		    字段id, 字段id, ...
		)

* updateFilter：array，格式同上，在update中不进行更新的字段，一般适用于主键和shardKey。没有则给空数组，array()
* toArrayFilter：array，格式同上，在Vo进行toArray的时候不组织到返回值中的字段，一般适用于不用于通知客户端的字段。没有则给空数组，array()
* searchColumns：array，格式同上，会在查询的时候出现在where条件里的字段。没有则给空数组，array()
* updateColumns：array，格式同上，会在更新的时候出现在where条件里的字段。没有则给空数组，array()
* jsonColumns：array，该字段用来描述数据库存储结构中，哪些字符串字段是用来保存json格式的内容，且其长度限制是多少。如果没有则给空数组，array()

		array(
		    字段id => 字段长度限制, ...
		)

* cacheColumn：int，字段id，缓存主键字段id，不适用时默认值为null，当数据被存储的到缓存里的时候选用哪个字段来构造缓存键
* shardColumn：int，字段id，分区字段id（当前数据存储到哪个数据库中），不适用时默认值为null
	* 举例：玩家的背包，主键是道具id，但是在查询的时候寻找数据库分区的时候，使用的是用户id这个字段，则用户id这个字段就是背包orm的shardColumn
* pkColumn：int，字段id，主键字段id（存储在*VoList的$this->list时候，数组的键值），不适用时默认值为null
* deleteColumns：array，格式同数组型字段，会在删除的时候出现在where条件里的字段。没有则给空数组，array()
* tableShardCount：int，模型分表数量，不适用时默认值为null
	* 分表：该字段决定当前数据模型需要在一个数据库内分成多少张表，且每个数据库内都有相同数量的该表
	* 举例：shardTableCount：X，则每个数据库中，都有当前这个数据模型表X张，编号从0 - (X-1)
* isList：boolean，表示该orm是否有列表模式
	* 举例：玩家的背包有列表模式，因为同一个玩家身上会有多个道具；而玩家自身这个对象则没有列表模式，因为只会有一个玩家
* dbType：string，数据库类型，定义在：`SzAbstractDb::DB_TYPE_*`
* cacheTime：int，缓存时长，单位为秒，该对象在缓存中存在的时长，过期后会自动消失
	* 如果配置为null，则缓存系统会给予一个适当的默认值，参见：SzAbstractCache::EXPIRES
	* 如果配置为0，表示当前对象不使用缓存

#### 2.5.7 profiler.config.php
详见第10部分

---

## 3. 代码结构及部署结构
### 3.1 代码结构
* 框架：直接参阅框架的源代码文件夹
* 应用：参考：`框架根目录/build/create`文件夹下的内容

### 3.2 部署结构
#### 3.2.1 应用
	/home/php/app/demo/
	                  | apps /
	                  |      | main /
	                  |      |      | latest /
	                  |      |      |        | app-codes ...
	                  |      |      | v0.1.0 /
	                  |      |      |        | app-codes ...
	                  |      |      | v0.1.1 /
	                  |      |      |        | app-codes ...
	                  |      |      | ...
	                  |      | devA /
	                  |      |      | ...
	                  |      | devB /
	                  |      |      | ...
	                  | lib  /
	                  |      | lib-codes ...
	                  | index.php
	                  | publish.config.php
	                  | SzPublish.class.php            

#### 3.2.2 框架
	/home/php/framework/
	                   | latest /
	                   |        | framework-codes ...
	                   | v0.1.0 /
	                   |        | framework-codes ...
	                   | v0.1.1 /
	                   |        | framework-codes ...
	                   | ...

#### 3.2.3 模块
	/home/php/module/
	                | profile /
	                |         | latest /
	                |         |        | module-codes ...
	                |         | v0.1.0 /
	                |         |        | module-codes ...
	                |         | v0.1.1 /
	                |         |        | module-codes ...
	                |         | ...
	                | payment /
	                |         | latest /
	                |         |        | module-codes ...
	                |         | v0.1.0 /
	                |         |        | module-codes ...
	                |         | v0.1.1 /
	                |         |        | module-codes ...
	                |         | ...
	                | ...

---

## 4. 工具脚本简介及使用
### 4.1 工具脚本
下述的工具脚本都是在开发的过程中，协助程序员工作的脚本，不参与到项目的部署中。

#### 4.1.1 基础
所有脚本的入口都在一个地方：`框架根目录/build/exec.php`，根据输入的参数不同，脚本会执行不同的功能。脚本的第一个参数为脚本运行的模式，它们定义在`Builder::MODE_*`，然后脚本会根据这个字符串，自动去查找`Builder::runMode*`这个入口，并执行它。除去第一个参数之外的参数，会合并作为这个入口的参数进行调用。

#### 4.1.2 脚本：创建新项目
这个脚本是用来创建一个全新的项目，参数：

* $appName：创建的新项目的名称
* $appRoot：新项目创建的目录

新项目会被创建在 `$appRoot/$appName`，所有 `框架根目录/build/create`的基础代码会被拷贝到指定的新项目目录下

#### 4.1.3 脚本：刷新项目
这个脚本是用来刷新当前存在的某个项目，包括框架自身。因为框架的构造是需要预定义的配置文件来指明library类的加载路径的，并且还需要工具来根据orm的配置文件来生成默认的orm类代码，这些配置文件和代码的生成工作由这个脚本来完成。参数：

* $path：需要刷新的项目的根目录

#### 4.1.4 脚本：测试项目
这个脚本是用来构建PHPUnit运行环境并进行单元测试的脚本。它会根据输入的路径参数，查找该路径下test文件夹，并运行该文件夹下的所有`*Test.php`单元测试文件。参数：

* $path：需要进行测试的项目根目录

### 4.2 ~~部署脚本 已废弃~~
下述的所有脚本都是放在部署机上的，也就是项目的服务器上，而不是git服务器上。使用git的钩子功能，由收到push通知的git服务器，通知部署机进行更新。

#### 4.2.1 ~~安装、更新与使用 已废弃~~
如果在package.js里的依赖库版本有更新，请使用：

	npm install --save-dev
	
命令进行依赖库安装的更新。然后使用：

	grunt --env=staging --repo=framework --dev=jonathan,mike,john
	
命令进行部署。

#### 4.2.1 ~~package.js 已废弃~~
所有的配置项，包含在该代码文件中，配置：

* devDependencies：运行该grunt脚本需要的依赖库，名字：版本号
* env：string，下面分为"dev"、"staging"、"production"等，看应用需求，env下的子项都是一系列的资源库名字，其中的子项：
	* clonePath：string，远程git版本库上的代码应该checkout到本地的什么位置，所有的资源使用同一个根目录
	* destPath：string，同上，这里则是更新、checkout指定版本之后再拷贝过来的实质部署位置
	* repo：string，git库的资源url，需要带上用户名和密码
	* branch：string，checkout的指定分支名
	* isApp：boolean，该资源库是否为应用库
	* isConfig：boolean | string，该资源库是否为应用配置库

配置注意项：

* framework配置库必须存在，名字必须为"framework"，且必须在所有其他库之前
* 配置资源库和应用资源库必须要有相同的tags
* 配置资源库必须配在应用资源库之后

#### 4.2.2 ~~Gruntfile.js 已废弃~~
grunt脚本的实质代码，根据输入的参数，进行代码部署：

* env：指定部署的环境，对应配置文件中的env，没有的话，默认为dev
* repo：指定部署的库名，对应配置文件中env下系列的资源库名，没有的话，默认部署所有的
* dev：指定应用的开发者文件夹，默认不给，则只部署"main"，文件夹名以逗号分隔

部署流程为：

* 克隆远程代码库到本地（如果本地还没有的话）
* 将grunt的工作目录切换到该目录
* checkout指定的分支
* 从远程代码库更新代码到本地分支
* 获取当前代码库所有的tags
* checkout各个tag，并进行代码部署
* 如果当前代码库为应用库，则会先将publish部分部署好
* 如果当前代码库为配置库，则还会将publish的配置文件部署到应用根目录
* 所有部署完成后，会再过一遍所有的应用库，根据main目录将各个开发者目录拷贝出来

---

## 5. 简单使用范例
参考：`框架根目录/build/create`文件夹下的内容，部署后即可直接访问

## 6. 核心模块
### 6.1 SzSystem
初始化三个绝对路径，以及系列核心组件单例类的实例化，代码：

	framework/SzSystem
* 框架根目录
* 应用根目录
* 模块根目录

SySystem内含系统用户唯一标识处理函数：handleIdentify，在系统内任何地方需要用户唯一标识的时候，请使用该函数。

这个函数能处理以下情况：

* 如果参数`$identify`不为默认值`null`，则保存传入的identify并返回（可重复执行保存identify，该逻辑是最早执行的）
* 如果参数`$identify`为默认值`null`则：
	* 检查SzSystem::$IDENTIFY是否为null，如果不为null表明已保存，直接返回
	* 检查HTTP请求中是否带`userId`参数，如果带，则保存并返回之
	* 解析HTTP请求内容`*`里的请求结构，返回第一个请求里的`第0号`参数，在通讯协议定义中，一般这个位置都是用户编号

这个函数不能处理的情况：

* 请求中不带任何参数，并且也不是API调用请求，e.g 首页展示请求
* 同上，GM首页展示，等其他页面展示请求

如果是上述情况，则需要程序员在对应的页面展示逻辑中进行用户唯一标识信息的识别，并手动调用该函数进行保存。

### 6.2 Config
指定位置配置文件内容读取，代码：

	framework/lib/config/SzConfig

### 6.3 Context
统一的数据库及缓存资源获得工厂，实现是在app级别的`SzContext`中，代码：

#### 6.3.1 Context
	framework/lib/context/SzAbstractContext
	    app/lib/context/SzContext

#### 6.3.2 Context工厂
	framework/lib/context/SzContextFactory
	
### 6.4 Controller
解析输入请求，控制请求流转，构造输出返回，代码：

#### 6.4.1 控制 和 分发 和 路由
	framework/lib/controller/SzController
	framework/lib/controller/SzDispatcher
	framework/lib/controller/SzRouter

#### 6.4.2 行为 和 预处理器
	framework/lib/controller/action/SzAbstractAction
	framework/lib/controller/handler/SzAbstractCtrlPreProcessHandler

#### 6.4.3 输入输出流
	framework/lib/controller/stream/SzRequest
	framework/lib/controller/stream/SzRequestManager
	framework/lib/controller/stream/SzResponse
	framework/lib/controller/stream/SzResponseManager

### 6.5 Cache
PHP runtime的对象、数据等的缓存，以及缓存系统（redis、memcache）的访问，代码：

#### 6.5.1 缓存访问
	framework/lib/cache/SzAbstractCache
	    framework/lib/cache/implementations/SzRedisCache
	    framework/lib/cache/implementations/SzMemcachedCache
	    
#### 6.5.2 缓存实例获取工厂
	framework/lib/cache/SzAbstractCacheFactory
	    app/lib/cache/SzCacheFactory
	    
#### 6.5.3 runtime系统缓存
	framework/lib/cache/SzSystemCache

### 6.6 Model
数据库的访问，代码：

#### 6.6.1 SQL构建器
	framework/lib/model/SzDbQueryBuilder

#### 6.6.2 持久化工具
	framework/lib/model/SzPersister

#### 6.6.3 VO序列化工具
	framework/lib/model/SzVoSerializer

#### 6.6.4 Model
	framework/lib/model/abstract/model/SzAbstractModel
	    framework/lib/model/implementations/model/SzMySqlModel
	    framework/lib/model/implementations/model/SzRedisModel

#### 6.6.5 Db
	framework/lib/model/abstract/db/SzAbstractDb
	    framework/lib/model/implementations/db/SzMySqlDb
	    framework/lib/model/implementations/db/SzRedisDb

#### 6.6.6 Db工厂
	framework/lib/model/abstract/db/SzAbstractDbFactory
	    framework/lib/model/implementations/db/SzDynamicShardDbFactory
	    framework/lib/model/implementations/db/SzFixedShardDbFactory
	        app/lib/model/db/SzDbFactory

#### 6.6.7 Vo
	framework/lib/model/abstract/vo/SzAbstractVo
	    framework/lib/model/abstract/vo/SzAbstractMySqlVo
	    framework/lib/model/abstract/vo/SzAbstractRedisVo

#### 6.6.8 VoList
	framework/lib/model/abstract/volist/SzAbstractVoList
	    framework/lib/model/abstract/volist/SzAbstractMySqlVoList
	    framework/lib/model/abstract/volist/SzAbstractRedisVoList

---

## 7. 请求与返回
### 7.1 请求
#### 7.1.1 请求的格式
所有的请求都是HTTP的POST请求，参数入口为`*`，格式为json字符串，原始格式为：

	array(
	    array(
	        "{$function}.{$action}",
	        $params => array(...)
	    ),
	    ...
	)
	
	---
	
如果开启了API重发功能.则需要添加sign,类型为string 及随机值 halt, 类型为int, 原始格式为:

	array(
	    array(
	        "{$function}.{$action}",
	        $params => array(...)
	    ),
	    ...
	)&sign=...&halt=..


#### 7.1.2 请求映射的路径及行为类
$function是该请求的功能块，$action是该请求的行为，$params是一个数组，包含了所有参数

	app/lib/controller/action/
	                         | $function /
	                         |           | ucfirst($function) . ucfirst($action) . 'Action.class.php'
	                         | page      /
	                         |           | PageIndexAction.class.php

#### 7.1.3 行为类的命名
行为类会根据`$function`的不同而分配到不同的子action类包种：

* ucfirst($function) . ucfirst($action) . Action.class.php
* page . index . Action.class.php -> `page`/PageIndexAction.class.php
* user . get . Action.class.php -> `user`/UserGetAction.class.php

#### 7.1.4 行为类的定义规则
必须申明实现execute函数，参数个数不定，可以根据需求自行编写：

* PageIndexAction::execute()
* UserGetAction::execute(`$userId`)

#### 7.1.5 行为类execute参数的定义
* 所有可用的参数类型统一定义在：`SzAbstractAction::TYPE_*`
* 当前行为类的execute的函数个数、顺序及类型，定义在行为类的`SzAbstractAction::$paramTypes`参数中

		array(
		    0 => TYPE_INT
		    1 => TYPE_FLOAT
		    2 => TYPE_STRING
		)

* 上述的例子中，该行为类的execute函数调用的时候必须有3个参数，第一个参数的类型为int，第二个参数的类型为float，第三个参数的类型为字符串，任何一个参数不存在，或者类型不正确，该execute都不会被执行，并有异常被抛出

#### 7.1.6 客户端请求的要点
客户端在缓存积累请求的时候，一旦有添加物品的请求时必须立刻发送到服务器，不可继续积累。因为服务器后端的数据持久化是在所有业务逻辑完成后统一执行的，而添加物品的请求完成后，数据并没有加入到清单中，后续的业务逻辑会出错。e.g：

* 请求包API列表：添加物品A 2个，使用物品A 1个
* 添加物品A业务逻辑完成，但是并没有持久化
* 背包对象内并没有物品A，结果在使用的时候失败了

此外，客户端在发出一个请求之后，必须等到发出的请求得到返回，才能发送下一个请求。要么等到请求超时时间超过，那么这算作一个失败的请求，进入道错误处理的逻辑中。

### 7.2 返回
#### 7.2.1 行为类的返回
返回值必定是一个SzResponse，包含了body内容和系列的headers等设定

#### 7.2.2 SzResponseManager的规则
* 默认的body永远是一个数组
* 只要有一个SzResponse里的file不为空，则当前的response就为文件下载模式
* 只要SzResponseManager里的contentType是`text/html`格式，那么数组格式的body会被取出0这个index的值，进行返回
* 如果contentType是`application/json`格式，那么返回值会是下述格式的json字符串：

		array(
		    'code' => $returnCode,
		    'msg'  => $returnValue,
		    'gmt'  => $gmtTimestamp
		)

* 这里code 0表示无异常，非0，则code值就是异常值，这个时候msg是一个字符串，就是错误信息
* 如果压缩开关打开的话，返回值body会被`gzcompress`压缩

#### 7.2.3 普通请求的返回结果
普通请求的返回结果，有意义的内容都在msg字段内，该字段会是一个json对象，其中包含一系列从0开始的自然键，其数量和请求包里的请求数量是相同的，且一一对应。

	array(
	    'code' => 0,
	    'msg' => array(
	        0 => 0号请求的返回结构,
	        1 => 1号请求的返回结构,
	        ...
	    )
	)

#### 7.2.4 持久层的返回结果
持久层的返回结果会被包含在msg字段下的`UPDATE`和`DELETE`这个键内，格式为：

	array(
	    'UPDATE' => array(
	        //-------------------
	        // LIST MODE:
	        'Item' => array( // orm name
	            $userId => array(
                   $itemId => array(
                       $columnName => $columnValue,
                       ...
               ),
               $itemId => ...,
               ...
           ),
           $userId => ...,
               ...
           ),
           'Soldier' => ...,
           //-------------------
           // SINGLETON MODE:
           'Profile' => array(
               $userId => array(
                   $columnName => $columnValue,
                   ...
               ),
               $userId => ...,
               ...
           ),
           'Token' => ...,
       ),
       'DELETE' => array(
	        //-------------------
	        // LIST MODE:
	        'Item' => array( // orm name
	            $userId => array(
                   $itemId, ...
               ),
               ...
           ),
           //-------------------
           // SINGLETON MODE:
           'Profile' => array(
               $userId, ...
           )
       )
	)
	
#### 7.2.5 API重发机制
在某些时候，因为网络中断或者其他原因，客户端请求协议失败，客户端需要重新调用协议，来进行客户端数据、逻辑更新，但是服务端有可能已经执行过此逻辑了，会造成系统报错，处于此原因，设计了api的重发机制，规则：	

* 在app.config.php中设置项目是否支持开启API重发功能
* 客户端需要在协议的url中，添加一个参数`sign`，每个协议的`sign`值需保证唯一
* 如果开启了api重发功能，服务端在协议执行完成之后，会将协议的返回结果，存储到cache中
* 服务端接收到协议后，根据`sign`值，去cache中查找是否已存在此`sign`值，若不存在，按正常逻辑执行，若已经存在，说明是此协议属于重发协议，直接返回cache中存储的返回结果
* 客户端不允许无限制的重发协议，重发次数配置在app.config.php中

### 7.3 请求的处理流程
	index.php
		SzController::process()
			SzRouter::parseRawInputs()
				SzProcessHandler::preProcess()
					SzRouter::formatRequests()
						loop SzRequestManager::shiftRequest(
							SzRequestManager::mergeResponse(
								SzDispatcher::dispatch(
									*Action::execute()
								)
							)
						)
							SzRequestManager::mergeResponse(
								
							)
								SzProcessHandler::postProcess()
									SzPersister::persist()
										SzRequestManager::send()

---

### 7.4 开启API重发功能后的处理流程.
	index.php
		SzController::process()
			SzRouter::parseRawInputs()
			    SzParam::checkApiSign()
			        if getAppCache with SzParam::getReqParam('sign')
			            responseCount++
			            SzRequestManager::send()
			            setAppCache
			        else
                        SzProcessHandler::preProcess()
                            SzRouter::formatRequests()
                                loop SzRequestManager::shiftRequest(
                                    SzRequestManager::mergeResponse(
                                        SzDispatcher::dispatch(
                                            *Action::execute()
                                        )
                                    )
                                )
                                    SzRequestManager::mergeResponse(
                                        
                                    )
                                        SzProcessHandler::postProcess()
                                            SzPersister::persist()
                                                setAppCache with SzParam::getReqParam('sign')
                                                SzRequestManager::send()
                    end
---

## 8. 系列约定
### 8.1 错误ID号
* 框架错误：10000 - 19999
* 应用程序错误：20000 - 29999
* 模块错误：30000 - 39999
	* module_profile：    30000 - 30099
	* module_item：       30100 - 30199
	* module_sns：        30200 - 30299
	* module_unlock：     30300 - 30399
	* module_privilege：  30400 - 30499
	* module_payment：    30500 - 30599
	* module_achievement：30600 - 30699
	* module_vip：        30700 - 30700
	* module_gm：         30800 - 30899
	* module_mission：    30900 - 30999
	* module_mail：       31000 - 31099   
	* module_ranking：    31100 - 31199
	* module_statics：    31200 - 31299
	* module_guild：      31300 - 31399
	* module_socket：     31400 - 31499

### 8.2 文件约定
* 所有的类文件必须以`class.php`为后缀
* 所有的配置文件必须以`config.php`为后缀
* 所有的配置文件必须放在`根目录/config` 文件夹下
* 所有的测试用例必须放在`根目录/test` 文件夹下
* 所有的测试用例必须以`*Test.php`格式命名

---

## 9. 释疑
### 9.1 为什么在非list类型的vo缓存存储的时候，不使用hash而是使用string
首先我们需要了解两者的差别，和先决条件：

* hash占用内存小，json格式的string占用内存大
* set命令复杂度为O(1)，而hmset复杂度为O(N)，N取决于输入的field个数多少
* 在一个程序中，非list类型vo数量不多，但是字段可能比较多（参考用户对象）

综上所述，对于非list的vo对象应该使用和list对象相同的序列化方法，然后作为字符串存储，在整体存储的过程中，对CPU的消耗较小，且减轻了整体序列化的复杂度。

### 9.2 为什么要使用集中持久化来处理多包请求
我们先要理解系统的瓶颈，一般来说系统的瓶颈常在数据库部分，即磁盘IO，然后才是网络IO，和前端服务器CPU等资源。因为作为轻量级的解释性开发语言，PHP的无状态特性决定了它能通过横向扩展来迅速加强其系统负载能力（买不起服务器不在讨论范围内）。    
作为一款框架，首要解决的就是如何减轻持久化的时候的磁盘IO次数，归并多包的持久化请求并一起执行是最好的解决方法。

---

## 10. Profile工具
Profile是一个很重要的工作，在开发的时候和线上产品分析的时候都很有必要。框架中集成了facebook开发的xhprof工具：

* github：[facebook/xhprof](https://github.com/facebook/xhprof)
* pecl：[pecl/xhprof](http://pecl.php.net/package/xhprof)

此外，为了方便管理xhprof的结果集，并进行可视化展示，我们需要一个便利的工具，这里在框架中集成的是XhGui：

* github：[perftools/xhgui](https://github.com/perftools/xhgui)

使用这个工具在数据收集的部分，也就是框架运行的时候需求的是：

* [XHProf](http://pecl.php.net/package/xhprof) php扩展，用来产生profile数据
* [MongoDB PHP](http://pecl.php.net/package/mongo) php的mongo数据库扩展，如果profile结果选择直接输入数据库的话，则需要该扩展

而观察收集起来的profile数据，则需要额外的两个php扩展：

* [mcrypt](http://php.net/manual/en/book.mcrypt.php)
* [dom](http://php.net/manual/en/book.dom.php)

这里项目程序需要做的只是填写配置文件，profiler.config.php：

* save.handler：string，使用什么手段来存储profile的结果，`mongodb` | `file`
* save.handler.filename：string，如果使用文件来保存profile的结果，文件位置和文件名是什么，e.g '/tmp/xhgui/xhgui_' . date('Ymd') . '\_' . uniqid() . '.dat'
* db.host：string，mongo数据库的连接地址，e.g mongodb://username:password@127.0.0.1:27017
* db.db：string，保存使用的数据库名
* db.options：array，额外的数据库选项，参考[MongoClient::__construct](http://www.php.net/manual/en/mongoclient.construct.php)
* profiler.enable：callable，是否在当前的php进程中启用profiler，返回值为true，则启动，反之则不启动
* profiler.simple_url：callable，在存储profile结果的时候，如何改写当前的url路径，默认策略为：`preg_replace('/\=\d+/', '', $url)`

## ~~11. Redis Log 已废弃~~
* 日志模块由两块数据组成
	* 日志索引结构表
		* 日志索引表的键为"logger_index:${timestamp}"
			* 此时间戳由“冷却节点算法”计算得到
			* 每5分钟更新一次键的时间戳
				* eg：首次节点为：2014-01-01 00:00:00， 第二次节点为2014-01-01 00:05:00 这个冷却节点内发生的日志都会放在此键内
		* 日志索引表的field为UUID，根据“唯一识别码算法”计算得到
			* 注：此UUID将会成为日志详细数据结构表的键
		* 以UUID作为field的value内保存玩家用户数据与日志类型，内容为json字符串
	* 日志详细数据结构表
		* 保存具体日志信息的结构表
		* 键为"logger_detail:${UUID}"
		* 其中每个field为日志的一项内容，value则是实际内容

* 日志查询系统
	* 根据时间查询：
		* 通过“冷却节点算法”获得时间段内的所有时间片的时间戳
		* 轮询索引结构表的数据，把时间片的uuid全部取出并计算数量，根据page页显示，每页n条数据
	* 根据userId，type查询
		* 通过“冷却节点算法”获得时间段内的所有时间片的时间戳
		* 轮询索引结构表的数据，取出符合条件的uuid，取出并计算数量，根据page页显示，每页n条数据

* 缓存结构
    * 报错数据存储结构
        缓存数据库：redis
        缓存方式：hash
        键值：SzLoggerIndex:$timestamp
        时效：10天

        字段名	 | 类型 | 默认值 | 索引 | 备注
        --- | --- | --- | --- | ---
        key | int | - | - | $uuid
        value | json | - | - | [$uuid, $userId, $type]

        * $uuid: 唯一识别码
        * $timestamp: 根据冷却节点算法，每5分钟，更新添加一个新$timeStamp
        * $userId: 用户id
        * $type: 日志类型

    * 报错数据索引存储结构
        缓存数据库：redis
        缓存方式：hash
        键值：SzLoggerDetail:$uuid
        时效：10天

        字段名	 | 类型 | 默认值 | 索引 | 备注
        --- | --- | --- | --- | ---
        identify | int | - | - | 用户名
        type | int | - | - | 日志类型
        data1 | string | - | - | 输入内容
        data2 | string | - | - | 输出内容
        session | int | - | - | 登入识别码
        time | int | - | - | 时间戳

## 11. 日志系统 含 业务数据采集
### 11.1 简述
程序日志和业务日志采集都统一使用linux的操作系统的syslog通道，以及LogStash进行采集。

* 程序日志会通过LogStash进入到ElasticSearch，直接使用Kibana进行搜索、观察
* 业务日志会通过LogStash落地到AWS的S3，后续使用大数据系统进行解析

### 11.2 syslog
所有的日志采集都通过linux的syslog通道，高并发、低延迟、低负载。LogStash和程序分身是分离的，会进行异步日志分析和消化，不会影响程序自身的性能。    
Debug本地调试的情况可以使用file日志形式，方便输出、观察。

### 11.3 日志格式
所有的日志在输送到syslog的时候都是字符串格式，字符串的内容是json格式。

#### 11.3.1 基础格式
基础格式：

	{
	  "time":   日志输出时间, 
	  "channel":日志输出通道, 
	  "level":  日志输出级别, 
	  "message":日志输出信息
	}

其中channel分为：

* program：程序日志（调试信息）
* business：业务日志（数据分析）

message按上述channel不同，其格式也不同，详见下文 11.3.2 和 11.3.3

#### 11.3.2 程序日志
message内容是一个json格式的字符串，含：

* identify：用户唯一标识，可能是用户ID等
* info：日志本体内容，字符串
* session：用户session
* params：json格式，额外参数，丰富日志内容，方便过滤查询

#### 11.3.3 业务日志
message内容同样是一个json格式的字符串，其格式按照不同的数据统计部门可能不尽相同，在框架内无描述。    
具体请咨询数据部门的格式需求。

### 11.4 程序使用
#### 11.4.1 程序代码设计
* SzLogger：公开使用的工厂类和实例获取类，根据级别有下述开放接口可供调用
	* debug：调试级别，仅在线上需要进行调试的时候打开
	* info：信息级别，普通的讯息级别，一般应用仅需要开放到这一级
	* notice：需要注意的级别，比较重要，但是不是危害内容
	* warn：非常危险，但是暂时仍未造成实际伤害
	* error：错误级别，已经造成实质伤害了
	* setLogLevel：设置日志级别
* SzAbstractLogger：日志抽象层，公开接口定义和一些公共工具函数都实现在这里
* Implementations：
	* SzSysLogger：将日志输出到Linux操作系统的syslog
	* SzFileLogger：将日志输出到某个本地磁盘文件
* 配置文件参见：`2.5.5 logger.config.php`

#### 11.4.2 程序使用范例
日志接口调用：

* 第一位，$message，string，放置的是程序日志输出的范畴，及基本的日志信息
* 第二位，$params，array，放置的是日志的额外结构化信息
	* 第二位的数组里的KEY我们称为：TAG
	* 某些特定的TAG是作为系统使用的TAG预占用掉了，参见：SzAbstractLogger::LOG_TAG_*
		* 用户session，定义为：SzAbstractLogger::LOG_TAG_SESSION

范例：

	SzLogger::get()->debug('SzMySqlModel: SQL', array(
	    'type'  => 'update',
	    'sql'   => $query
	));

#### 11.4.3 预定义常量
参见：

* SzAbstractLogger::LOG_CHANNEL_*
* SzAbstractLogger::LOG_TAG_*

#### 11.4.4 应用级别使用规范
* 在第一位的文本字段中，使用类名、函数名，加上简洁的文字描述
	* e.g SzMySqlModel: SQL
* 将比较丰富的、可结构化的日志内容放到第二位的数组里，这样做就能在日志系统内进行快速查询
* 上述的第一位文本字段内容和第二位数组中的TAG在应用中反复被用到时，就需要考虑是否需要将其抽出放在常量中
	* e.g SzAbstractLogger::LOG_TAG_SESSION

## 12. 错误处理
### 12.1 PHP错误分类
主要应该分为PHP程序级别的错误和应用级别的Exception：

* 前者请参见php.ini配置文件里的错误配置项：`error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT`，含多种级别的错误
* 后者就是普遍的程序Exception

### 12.2 常用错误处理
根据12.1的分类：

* PHP级别的错误一般使用日志记录的方式进行文件记录，然后使用第三方的解析方式进行错误统计和分析，但是大多数情况下都做的不好，很多时候错误都直接被忽略了
* Exception的错误处理完全靠应用程序自身，一般也做得不是很好，甚至还不如PHP的错误文件记录

### 12.3 框架错误处理机制
现PHP的框架会将错误和异常全部捕获，然后记录到日志系统内，使用外部的日志分析工具进行解析记录，方便查询、统计。    

#### 12.3.1 错误处理类及事件注册
错误处理总入口：

	SzErrorHandler::init

注册了3个事件，通过下列的3个事件，所有的PHP错误都会被捕获，并得到正确处理：

	SzSystem::registerShutdownHandler(self::$instance, 'handleFatal');
	SzSystem::registerErrorHandler(self::$instance, 'handleError');
	SzSystem::registerExceptionHandler(self::$instance, 'handleException');

#### 12.3.2 handleException
handleException会捕获所有框架所属应用程序内抛出的所有Exception。   
如果是原生的Exception`new Exception(...)`，则记录日志后继续抛出。这会造成一个PHP级别的Fatal Error，其内容为未捕获的Exception，被记录到PHP的错误日志内。   

如果是框架的Exception`new SzException(...)`，则记录之后重新构建返回值返回给客户端。这种Exception不会造成任何PHP级别的错误处理，因为在程序级别，这个错误已经被处理掉了。

使用的是PHP的`set_exception_handler`。

#### 12.3.3 handleError
handleError则用来处理12.1点种说明过的PHP程序级别错误，其中需要注意的是Fatal Error级别的错误是不会被这个事件处理的。因为PHP自身的错误处理机制，Fatal Error是不会被`set_error_handler`这个函数设定的事件捕获的。   

通过`set_error_handler`捕获到的错误，将不会出现在PHP的错误日志文件中，也就是说被吃掉了。这一点必须注意。

#### 12.3.4 handleFatal
handleFatal是用来处理12.3.3中描述说无法被捕获的Fatal Error的。因为PHP自身没有原生的事件能处理这个错误，我们使用了一种迂回的方式：

使用`register_shutdown_function`，注册了一个PHP退出事件，在其中进行`error_get_last`监测最后是否有发生过错误，如果有且发生的错误确实是PHP的Fatal Error，那么就行错误处理，否则直接跳过。

只有Fatal Error，在`register_shutdown_function`中被日志系统记录下来之后，仍旧会走到PHP的错误日志里。

### 12.4 举例说明
#### 12.4.1 PHP的Notice
代码：

	echo $name; // name not defined

错误日志：

	SzErrorHandler: Error caught, params: {"type":"E_NOTICE","no":8,"content":"Undefined variable: name","file":"\data1\www\flowershop2.shinezoneapp.com\apps\jonathan\latest\lib\controller\action\page\PageIndexAction.class.php","line":66}

#### 12.4.2 PHP的Notice
代码：

	echo 'invalid: ' . 9/0;

错误日志：

	SzErrorHandler: Error caught, params: {"type":"E_WARNING","no":2,"content":"Division by zero","file":"\data1\www\flowershop2.shinezoneapp.com\apps\jonathan\latest\lib\controller\/action\/page\PageIndexAction.class.php","line":65}

#### 12.4.3 PHP的Fatal Error
代码：

	$a = new ClassNotFound(); // ClassNotFound not defined

错误日志：

	SzErrorHandler: Error caught, params: {"type":"E_ERROR","no":1,"content":"Class 'ClassNotFound' not found","file":"\data1\www\flowershop2.shinezoneapp.com\apps\jonathan\latest\lib\controller\action\page\PageIndexAction.class.php","line":69}

#### 12.4.4 PHP原生Exception
非框架SzException的原生Exception，会在捕获记录日志之后再次抛出，则会在日志系统里造成两次记录。

第一次是`set_exception_handler`捕获到的。   
第二次是`register_shutdown_function`捕获到的Fatal Error，其内容是：未捕获的Exception，因为上一次捕获之后又再次抛出了。

代码：

	throw new Exception(123);

错误日志1：

	SzErrorHandler: Exception caught, params: {"msg":"123","code":0,"trace":"#0 \data1\www\flowershop2.shinezoneapp.com\apps\jonathan\latest\lib\controller\action\page\PageIndexAction.class.php(23): PageIndexAction->displayIndex()\n#1 [internal function]: PageIndexAction->execute()\n#2 \data1\www\flowershop2.shinezoneapp.com\framework\1.0.4.0\lib\controller\SzDispatcher.class.php(41): call_user_func_array(Array, Array)\n#3 \data1\www\flowershop2.shinezoneapp.com\framework\1.0.4.0\lib\controller\SzController.class.php(108): SzDispatcher->dispatch(Object(SzRequest))\n#4 \data1\www\flowershop2.shinezoneapp.com\apps\jonathan\latest\www\index.php(14): SzController->process()\n#5 \data1\www\flowershop2.shinezoneapp.com\index.php(13): require('\data1\www\flow...')\n#6 {main}"}

错误日志2：

	SzErrorHandler: Error caught, params: {"type":"E_ERROR","no":1,"content":"Uncaught exception 'Exception' with message '123' in \data1\www\flowershop2.shinezoneapp.com\apps\jonathan\latest\lib\controller\/ction\page\PageIndexAction.class.php:70\nStack trace:\n#0 \data1/www\flowershop2.shinezoneapp.com\apps\jonathan\latest\lib\controller\action\page\PageIndexAction.class.php(23): PageIndexAction->displayIndex()\n#1 [internal function]: PageIndexAction->execute()\n#2 \data1\www\flowershop2.shinezoneapp.com\framework\1.0.4.0\lib\controller\SzDispatcher.class.php(41): call_user_func_array(Array, Array)\n#3 \data1\www\flowershop2.shinezoneapp.com\framework\1.0.4.0\lib\controller\SzController.class.php(108): SzDispatcher->dispatch(Object(SzRequest))\n#4 \data1\www\flowershop2.shinezoneapp.com\apps\jonathan\latest\www\index.php(14): SzController->process()\n#5 \data1\www\flowershop2.shinezoneapp.com\index.php(13): require('\data1\www\flow...')\n#6 {main}\n  thrown","file":"\data1\www\flowershop2.shinezoneapp.com\apps\jonathan\latest\lib\controller\action\page\PageIndexAction.class.php","line":70}

#### 12.4.5 框架Exception：SzException
代码：

	throw new SzException(20000);

错误日志：

	SzErrorHandler: Exception caught, params: {"msg":"Water: Insufficient water!","code":20000,"trace":"#0 \data1\www\flowershop2.shinezoneapp.com\apps\jonathan\latest\lib\controller\action\page\PageIndexAction.class.php(23): PageIndexAction->displayIndex()\n#1 [internal function]: PageIndexAction->execute()\n#2 \data1\www\flowershop2.shinezoneapp.com\framework\1.0.4.0\lib\controller\SzDispatcher.class.php(41): call_user_func_array(Array, Array)\n#3 \data1\www\flowershop2.shinezoneapp.com\framework\1.0.4.0\lib\controller\SzController.class.php(108): SzDispatcher->dispatch(Object(SzRequest))\n#4 \data1\www\flowershop2.shinezoneapp.com\apps\jonathan\latest\www\index.php(14): SzController->process()\n#5 \data1\www\flowershop2.shinezoneapp.com\index.php(13): require('\data1\www\flow...')\n#6 {main}"}
