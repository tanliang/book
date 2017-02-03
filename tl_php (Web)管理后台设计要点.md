# tl_php (Web)管理后台设计要点

tl_php 后台是一个从08年开始自研 MVC 框架，功能从最初的简单 CMS 内容管理，根据项目经历，不断调整功能，至此算初步完善，在开发新项目时，能达到事半功倍的效果，希望对大家有所帮助。

- 特性简介，url重写规则介绍
- 使用准备，sql导入，后台访问
- 权限控制，新增管理员
- 视图模块，视图widget
- 缓存加载，pd主动加载，mod被动调用
- 简单搜索，替代 SQL 的 like 查询
- 分表分库，hash，自动脚本
- 关于云服务（S3、DynamoDB）

## 特性简介

tl_php 是一个 MVC 模式的功能代码合集，目录结构如下：

![目录](resource/book2/tl_php_dir.jpg)

- 默认二级域名访问，对应 apps 目录下的模块名称，如： admin.tl.dev。IP 访问则一级 URL 值为模块名称，如：127.0.0.1/admin
- 配置目录 config 放置数据存储相关，分为 local(本地服), test(测试服), online(正式服)，目录下 test 文件默认为空，如有值且于当前访问 $_SERVER['HTTP_HOST'] 相同，则加载测试服配置信息
- lib 目录放置核心功能库 TL，及各种第三方库
- mod 目录放置具体，及扩展业务逻辑
- public 作为入口目录，配置 apache 或 nginx 时，均指向此
- var/cli/important.php 为数据库库分表初始化脚本，var/job/cron_message.php 为自动执行获取客服信息脚本。  

