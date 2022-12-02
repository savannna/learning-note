##  创建一个数据库
https://cloud.tencent.com/developer/article/1794083
https://cloud.tencent.com/developer/article/1794083
```
create database xxxx character set utf8;(编码中文正常显示)
show databases;
use xxxx;
alter database MyDB_one character set utf8;
```
## 前置处理
> 添加环境变量 
> - export PATH=${PATH}:/usr/local/mysql/bin/    
> - source ~/.zshrc   

# 打开mysql
```
mysql -u root -p
```
# mysql忘记密码
https://www.cnblogs.com/jiuyi/p/6211271.html
https://blog.csdn.net/m0_47696151/article/details/119717177  (语句换成小写)
```
sudo /usr/local/mysql/support-files/mysql.server stop
sudo /usr/local/mysql/bin/mysqld_safe --skip-grant-tables
```
