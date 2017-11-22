## 问题
> git push 失败后，执行git pull 再执行git pull会引发merge操作,如何避免？

## 解决
- git reset  --soft [commit ID：本地commit之前的ID]
- git stash :暂存之前的修改
- git pull ：获取git远程仓库的更新
- git stash apply ：将暂存的修改拿出
- git add ...:重新add 需要的提交
- git commit ...:重新commit
- git push ...: 重新push