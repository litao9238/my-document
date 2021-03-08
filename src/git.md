## git常用命令总结


1. 远程仓库相关命令
```
git clone [url]                                  从服务器克隆代码
git remote -v                                    查看远程仓库
git remote add origin [url]                      添加远程仓库
git remote rm [name]                             删除远程仓库
git remote set-url --push [name] [newUrl]        修改远程仓库
git pull [remoteName] [localBranchName]          拉取远程仓库
git push [remoteName] [localBranchName]          推送远程仓库
git push origin [remoteNmae]                     本地当前分支push到远程remoteNmae分支
git push origin [localName]:[remoteNmae]         本地localName分支作为远程remoteNmae分支(没有则自动创建)
git push --set-upstream origin [remoteNmae]      设置本地分支追踪远程分支, 之后可使用 git push 提交代码
git push -u origin [name]                        推送并将远程name分支作为默认分支, 之后可使用 git push 提交代码
```

2. 分支(branch)操作相关命令
```
git branch                                       查看本地分支       
git branch -a                                    查看所有的分支      
git branch -r                                    查看远程所有分支
git branch -m [oldName] [newName]                修改本地分支名
git branch [name]                                创建分支, 但不会自动切换为当前分支
git branch -d [name]                             删除本地分支 -D 为强制删除
git push origin --delete [name]                  删除远程分支
git checkout [name]                              切换分支
git checkout -b [newName]                        本地从当前分支创建并切换至newName分支
git checkout -b [newName] origin/[remoteNmae]    从远程分支remoteNmae创建本地分支newName并检出
```