# blog
fex.meishakeji.com

# 请fork项目后进行博客编写，不要在本项目进行编写
 操作步骤：
``` js
  // 1.配置当前fork的仓库的原仓库地址
  git remote add upstream <原仓库github地址>
  // 2.查看当前仓库的远程仓库地址和原仓库地址
  git remote -v
  // 3.获取原仓库的更新。
  git fetch upstream
  // 4.合并到本地分支
  git merge upstream/master
  // 5.查看跟新
  git log
  // 6.推送更新
  git push origin master
```