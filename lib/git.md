# 合并多次提交

## 目的

在**GitLab**中，我们通过**cherry-pick**的方式合并修改内容，而cherry-pick每次只能将一次commit合并到主分支，所以我们在有多次commit存在的时候，需要将多次分支进行合并，才可以更方便的进行cherry-pick。

## 原理

我们主要通过变基的方式，将多次commit的parent变更为刚拉分支时的那一次commit，这样可以既可以合并commit，还便于观察到所有的修改内容。

## git命令

    git pull
	git rebase -i HEAD~合并提交数量
	git pull
	git push

## 操作方式

如图所示，我希望把我的这四次commit合并
![avatar](/res/TIM截图20190816153538.jpg)
在进行变基之前，我们先pull一下分支，以防本地分支与远端分支产生冲突。
![avatar](/res/TIM截图20190816153733.jpg)
然后进行变基的操作，输入完命令后，git会自动打开vim。
我们把第一次的pick改为r，后面的全部改为s，然后保存退出。
![avatar](/res/TIM截图20190816154045.jpg)
接着在vim里面把上面的内容改为变基后，需要commit的描述，保存退出。
![avatar](/res/TIM截图20190816154147.jpg)
然后还会再弹出一次vim，直接保存退出即可。
![avatar](/res/TIM截图20190816154225.jpg)
完成后，git会显示Successfully Rebased......
![avatar](/res/TIM截图20190816154248.jpg)
然后我们再从远端分支pull一次，这次会因为我们本地出现的两个HEAD而冲突，git会自动将这两个HEAD合并。
![avatar](/res/TIM截图20190816154417.jpg)
### 为什么会出现两个HEAD
因为我们在rebase后，会出现一个新的commit，这个commit的parent就是分支刚建立时的最后一次提交，所以在此时，该分支会同时存在两个HEAD导致冲突，如下图：
```
graph LR
A[拉分支时的最后一次commit] -- rebase --> B(HEAD?)
A --> C(( ))
C --> E(( ))
E --> F(( ))
F --> G(HEAD?)

```

所以git会自动将这两个connit进行合并，而合并后的parent就是这两个的commit的哈希值。
```
graph LR
A[ ] -- rebase --> B( )
A --> C(( ))
C --> E(( ))
E --> F(( ))
F --> G( )
B -- parent --> D[git自动生成的merge]
G -- parent --> D

```

然后我们进行push
![avatar](/res/TIM截图20190816154449.jpg)

---
