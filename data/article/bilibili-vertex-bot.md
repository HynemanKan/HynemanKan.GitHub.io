# qq_bot

本项目为酷Q的CoolQ HTTP API插件Python SDK 的服务框架，在原有基础上集成了插件管理、会话管理和一个简易后台，让使用Python的开发者能更加方便的开发CoolQ服务。仅支持 Python 3.6+

关于CoolQ HTTP API插件，见 [richardchien/coolq-http-api](https://github.com/richardchien/coolq-http-api)。

关于Python SDK，见 [richardchien/python-cqhttp](https://github.com/richardchien/python-cqhttp)。

## 用法
该框架需要Mysql，Redis的支持

### 安装环境

```sh
pip install pymysql,redis,cqhttp
```

### 建立必要数据库

```sql

CREATE TABLE `admin`  (
  `id` varchar(9) NOT NULL,
  `password` varchar(32) NOT NULL,
  `salt` varchar(36) NOT NULL,
  `auth_class` int(11) NOT NULL,
  `OTP_key` varchar(16) NOT NULL,
  PRIMARY KEY (`id`)
);

DROP TABLE IF EXISTS `error_log`;
CREATE TABLE `error_log`  (
  `time` datetime(0) NOT NULL,
  `error_detail` text NOT NULL
);

CREATE TABLE `private_message_plugin`  (
  `plugin_name` varchar(50)  NOT NULL,
  `plugin_bname` varchar(50) NOT NULL,
  `package_name` varchar(200) NOT NULL,
  `active` int(11) NULL DEFAULT NULL,
  PRIMARY KEY (`plugin_name`)
  ) ;
CREATE TABLE `group_message_plugin`  (
  `plugin_name` varchar(50) NOT NULL,
  `plugin_bname` varchar(50) NOT NULL,
  `package_name` varchar(200) NOT NULL,
  PRIMARY KEY (`plugin_name`)
) ;
CREATE TABLE `group_message_plugin_activate`  (
  `g_id` varchar(20) NOT NULL,
  `plugin_name` varchar(50) NOT NULL,
  CONSTRAINT `fake_plugin_name` FOREIGN KEY (`plugin_name`) REFERENCES `group_message_plugin` (`plugin_name`) ON DELETE CASCADE ON UPDATE CASCADE
);
CREATE TABLE `plugin_out_blueprint`  (
  `plugin_name` varchar(30) NOT NULL,
  `url` varchar(30) CHARACTER NOT NULL,
  `package_name` varchar(200) NOT NULL
);

```

### 配置 CoolQ HTTP API插件

见 [richardchien/python-cqhttp](https://github.com/richardchien/python-cqhttp)

### 配置反向代理

将/admin/和/out/映射到想要的端口或域名上。前者将是管理页面的入口，后者将是用户页面的入口

### 配置 setupfile.py 文件

```py
out_url = "your_out_address"
cq_api_root = 'your_api'
cq_access_token = 'your_token'
cq_secret = 'your_secret'
redis_host = 'host'
redis_port = "port"
redis_db = "db"
mysql_host = "host"
mysql_port = "port"
mysql_username = "username"
mysql_password = "your_password"
mysql_db = "your _db"
```

### 部署

`bot.run()` 只适用于开发环境，不建议用于生产环境，因此 SDK 从 1.2.1 版本开始提供 `bot.wsgi` 属性以获取其内部兼容 WSGI 的 app 对象，从而可以使用 Gunicorn、uWSGI 等软件来部署。

### 验证
在数据库`private_message_plugin`表中添加如下数据，以启用测试插件。

| plugin_name | plugin_bname | package_name | active |
|  ----  | ----  | ----  | ----  |
| private_test | 测试 | plugin.private_message_plugin.test_plugin | 0 |

通过一下函数生成管理员用户

```py
import hashlib
import uuid
import pyotp
salt = str(uuid.uuid4())
print("your salt:",salt)
md5 = hashlib.md5()
your_password = "your_password"
md5.update(f"{your_password}{salt}".encode("utf-8"))
print("password:",md5.hexdigest())
user_otp_key = pyotp.random_base32()
print("your otp kay:",user_otp_key)
```

并添加到`admin`表中

| id | password | salt | auth_class | OTP_key |
| ---| -------- | ---- | ---------- | ------- |
| your_id | md5_password | salt | 0 | otp kay |

此时后台应可以访问，向机器人账号发送"test"，应回复"success"。
