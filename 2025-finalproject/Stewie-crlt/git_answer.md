# **Git应用**



## 1. 回退修改到最近提交状态



### 方法一：使用 git checkout 回退工作区修改
`git checkout -- ||git checkout -- filename`


### 方法二：使用 git reset 回退暂存区修改
##### 将暂存区的修改移回工作区
`git reset HEAD

##### 然后使用 checkout 回退工作区修改
`git checkout 


### 方式三：使用 git restore
#### 恢复工作区文件
`git restore .

#### 恢复暂存区文件到工作区
`git restore --staged 


## 2. 回退已提交版本

### （1）不修改历史的方式
#### 方式一：使用 git revert
`git revert HEAD

#### 方式二：使用 git reset --soft
`git reset --soft HEAD~1


### （2）修改历史的方式
#### 方式一：使用 git reset --hard
`git reset --hard HEAD~1

#### 方式二：使用 git commit --amend
`git add .
`git commit --amend


## 3.合并分支的不同方式
### 方法一：使用 rebase（变基）
`git checkout feature-branch
`git rebase main

### 方法二：使用 cherry-pick（选择性合并）
`git checkout main
`git cherry-pick <commit-hash>


### 方法三：使用 squash merge（压缩合并）
`git merge --squash feature-branch
`git commit -m "合并feature分支的所有更改"