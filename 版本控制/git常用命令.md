#git常用命令
* clone代码，会自动创建目录

`git clone git@github.com:Michaelromic/MyGitBook.git ./GitBook`
* 从远程获取最新版本并merge到本地

`git pull origin gh-pages`
* 把修改提交到暂存区

`git add .`
* 提交说明 (一定要先提交说明)

`git commit -m "修正xxxbug"`
* 推送本地master分支到远端master分支

`git push origin master:master`

---
* 切换 gh-pages 分支

`git checkout -b gh-pages`
* 新建 gh-pages 分支:

`git push origin gh-pages`
* 切换回master分支

`git checkout master`

---
* 查看远程分支

`git branch -a`
* 查看本地分支

`git branch`
* clone其他分支代码

`git clone -b branchname git@github.com:Michaelromic/MyGitBook.git ./branchname `

---
* git切换ssh和http协议：
    *  查看当前remote: 
        `git remote -v`
    * 切换到ssh：
        `git remote set-url git@github.com:drchen126/b0.git`
也可以直接改当前git目录里面有个配置文件。