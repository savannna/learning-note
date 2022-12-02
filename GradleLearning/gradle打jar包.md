https://bbs.huaweicloud.com/blogs/301728  
- 打完的jar包在build/libs里
***
- 排除test编译
```
gradle build -x test
```
***
经常要注意过时的版本写法
- 查看过时写法
```
gradle --warning-mode all
```
