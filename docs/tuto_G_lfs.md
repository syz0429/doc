
## 大文件支持

```shell
git init .
# 让仓库支持LFS
git lfs install
git lfs track "*"
git add .gitattributes
git commit -m "Add \"*.bigfile\" LFS config "

# 添加大文件
git add dyrone.bigfile
git commit -m "Add a really big file"
# 查看跟踪的大文件
git lfs track
git push
```

常用命令
```shell
git lfs status  // 查看当前git lfs对象的状态

git lfs ls-files  // 查看当前哪些文件是使用lfs管理的

# 放弃工作区的更改
git checkout -- .
# 放弃暂存区的更改
git reset HEAD
```




## 参考

* [如何存储 Git 大文件？](https://www.cnblogs.com/88223100/p/How-do-I-store-large-Git-files.html) 

* [如何使用 Git LFS](https://help.aliyun.com/zh/yunxiao/user-guide/how-to-use-git-lfs)




