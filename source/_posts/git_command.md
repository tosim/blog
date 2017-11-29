---
title: git_command
copyright: true
tags:
- command
categories:
- command 
---

# git基本用法
学习git时候的基本入门命令

<!-- more -->

## 创建版本库

### 1.创建文件夹
```
$ mkdir learngit
$ cd learngit
```

---

### 2.初始化为版本库
```
$ git init
```

---

## 版本控制

### 1.版本回退
```
$ git log --pretty=oneline
3628164fb26d48395383f8f31179f24e0882e1e0 append GPL
$ git reset --hard 3628164
```

----

### 2.管理修改
```
$ git add filename
$ git commit -m "commit info"
```

---

### 3.撤销修改

#### 丢弃工作区的修改
```
$ git checkout -- filename
```

#### 丢弃暂存区的修改
```
$ git reset HEAD filename
```

---

### 4.删除文件
```
$ git rm filename
```

---

## 远程仓库

### 1.创建ssh-keygen,默认保存在用户目录下的.ssh文件夹下有<code>id_rsa</code>和<code>id_rsa.pub</code>
```
$ ssh-keygen -t rsa -C "youremail@example.com"
```

### 2.将<code>id_rsa.pub</code>中的内容添加到你的github的ssh管理中

### 3.在github上新建项目

### 4.将本地版本库同步到git仓库
```
$ git remote add origin git@github.com:example/example.git
```

---

## 分支管理

### 1.创建分支
```
$ git checkout -b dev
```

### 2.查看分支
```
$ git branch
```

### 3.分支合并
```
$ git merge dev
```

### 4.解决冲突
```
<<<<<<< HEAD
Creating a new branch is quick & simple.
=======
Creating a new branch is quick AND simple.
>>>>>>> feature1
```
解决冲突后再次add,commit 就完成了分支的合并

---

### 5.工作现场
#### 保存工作现场
```
$ git stash
```

#### 查询工作现场列表
```
$ git stash list
stash@{0}: WIP on dev: 6224937 add merge
```

#### 恢复工作现场
```
$ git stash pop
```
或
```
$ git stash apply
```
或
```
$ git stash apply stash@{0}
```

## 多人协作
### 1.clone master分支并关联
```
$ git clone git@github.com:example/example.git
```

### 2.clone dev分支并关联
```
$ git checkout -b dev origin/dev
```

### 3.修改之后合并到dev分支并push到远程

### 4.若push成功则ok

### 5.若push失败
```
$ git pull
```
抓取远程最新的对应的分支

### 6.若pull成功,则再push

### 7.若pull失败,则说明有冲突,手动解决冲突后再push
