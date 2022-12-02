# 同级多SSH，多github账户
https://note.dolyw.com/command/04-Git-MultiUser.html#_1-3-%E7%99%BB%E5%BD%95%E9%85%8D%E7%BD%AE%E5%85%AC%E9%92%A5
## 一、切换目录
```
cd Desktop
```
```
cd ~/.ssh
```
## 二、生成密钥
执行命令，生成第一个账号的SSH
```
ssh-keygen -t rsa -C "13936656837@163.com"
```
> **不要一路回车**，在第一个对话的时候输入公私钥重命名为id_rsa_personal

执行命令，生成第二个账号的SSH
```
ssh-keygen -t rsa -C "jing.li3@thoughtworks.com"
```
> 重命名为 id_rsa_company
## 三、修改config
生成配置文件
```
touch config
```
修改
```
Host personal
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_personal

Host company
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_company

```
## 四、在github里添加ssh
在settings里黏贴id_rsa_personal.pub公钥文件
## 五、验证连接是否成功
```
ssh -T personal
```
```
ssh -T company
```
默认域名：
```
ssh -T git@github.com
```
***

# 修改密钥后,各种clone push要修改域名
## 一、修改push域名
https://cloud.tencent.com/developer/article/1952800
> 将默认域名**git@github.com**修改为**personal**

查看现在远程
```
git remote -v

origin  git@github.com:savannna/3-springboot-learning-by-doing.git (fetch)
origin  git@github.com:savannna/3-springboot-learning-by-doing.git (push)
```
修改为：
```
git remote set-url origin personal:savannna/3-springboot-learning-by-doing.git
```
