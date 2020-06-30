# Linux的用户管理

**用户	权限	资源**

相对于的管理

#### useradd 

创建一个用户 useradd [用户名]

#### passwd	

passwd [用户名] 

下一步输入密码和确定密码

用户之间不能相互访问

#### groupadd

添加组，groupadd 组名 

#### usermod  

给用户添加组

usermod -a -G [组名] [用户名]

#### chown

修改资源的组

chown root:[组名]	[资源名]

#### chmod

有两种方式一种是数值型，另一种是字符型

chmod g(组)+w(添加的权限) [资源名称]





