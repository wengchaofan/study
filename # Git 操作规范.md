# Git 操作规范  
----- 
 
## **Git 常用命令**

### 初始化仓库  
```
  git init ##初始化本地仓库 生成.git文件 ssh(rsa)/keygen
  cat ~/.ssh/id_rsa.pub
  git remote add origin/branch_name ##关联上下游  
  git remove -v [upstream] ##查看关联  
  touch ##创建你的第一个 git file  
```

### 文件操作
```
  git add . .* ./*  ##三者区别  
  git status ##查看文件状态
  git stash  
  git stash list  
  git stash pop[index]  
  git stash clear ##存取本地文件
```

### 信息操作
```
  git commit -m / -a / --amend ##三者的区别
  message的组成建议：
   1. 功能标签 [feat, fix, test, reset, timing ...]
   2. 设计模块 [ifaas-connection, ifaas-datamining, 超级布控]
   3. 修改方式 [修改xxx处理逻辑, 处理xxxx问题, 调整xxxx表现]
   4. 补充
  推荐组成eg: 
    1. feat(新增XXXX功能 | 涉及服务/模块 | 补充)
    2. fix(修改XXXXX | 修改XXX交互方式 | bug编号 00001)
  推荐插件
    yarn commit
```


### 分支操作
```
  git log --since="2020.03.08" --graph--oneline --decorate
  git reflog
  git show

  git checkout . -b -B --orphan --merge [--datch] branch_name
  git branch -D -d -r
  git fetch
  git reset --hard
  git revert
  git push -f

```

### 重点讲解
```
  强大的 rebase -i squash save drop [vim] wq qa! ws
```

 **常见问题**
 ```
  1.提交代码有冲突？？？？？？？？？
  2.我！的！代！码！被！覆！盖！了！
  3.我的gitTree特别的乱。
  4.合并某一个(几个)commit 到 某一分支？
  5.回退版本。
  6.回退暂存。
 ```
 

### 一些关于Git管理的建议
```
  1.分支命令与管理
  2.commit的规范与提交
  3.分支合并的利与弊
  4.抽离版本的基本法
```

