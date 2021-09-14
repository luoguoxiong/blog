# SQL注入

## 原因
例如做一个系统的登录界面，输入用户名和密码，提交之后，后端直接拿到数据就拼接 SQL 语句去查询数据库。如果在输入时进行了恶意的 SQL 拼装，那么最后生成的 SQL 就会有问题。

比如后端拼接的 SQL 字符串是：

SELECT * FROM user WHERE username = 'user' AND password = 'pwd';
如果不做任何防护，直接拼接前端的字符，就会出现问题。比如前端传来的user字段是以'#结尾，password随意：

SELECT * FROM user WHERE username = 'user'#'AND password = 'pwd';
密码验证部分直接被注释掉了。

## 防范
后端应该对于字符串有转义，可以借助成熟的库的 API 来拼接命令，而不是自己手动拼接。