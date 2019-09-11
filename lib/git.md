# 合并多次提交

## 目录

* [目的](#目的)
* [原理](#原理)
* [git命令](#git命令)
* [操作方式](#操作方式)
  * [为何会出现冲突](#为何会出现冲突)
* [图示](#图示)

## 目的

一个功能的开发可能涉及多次commit，这常常是难以避免的，如下图所示，开发分支上有四次提交：
![avatar](/res/TIM截图20190819145149.jpg)
在**GitLab**中，我们通过**cherry-pick**的方式合并修改内容，而cherry-pick每次只能将一次commit合并到主分支，所以我们在有多次commit存在的时候，需要将多次分支进行合并，才可以更方便的进行cherry-pick。

## 原理

我们主要通过变基的方式，将多次commit的parent变更为刚拉分支时的那一次commit，这样可以既可以合并commit，还便于在主分支中观察到所有的修改内容，合并完成后，通过cherry-pick合并进主分支即可。操作原理如图所示：
![avatar](/res/TIM截图20190819164710.jpg)

## git命令

```git
git pull
git rebase -i HEAD~合并提交数量
git pull
git push
```

## 操作方式

1. 如图所示，我希望把我的这四次commit合并
2. 在进行变基之前，我们先pull一下分支，以防本地分支与远端分支产生冲突
3. 然后进行变基的操作，输入完命令后，git会自动打开vim
4. 我们把第一次的pick改为r，后面的全部改为s，然后保存退出
5. 接着在vim里面把上面的内容改为变基后，需要commit的描述，保存退出
6. 然后还会再弹出一次vim，直接保存退出即可
7. 完成后，git会显示Successfully Rebased......
8. 然后我们再从远端分支pull一次，这次弹出vim会因为我们本地出现的两个HEAD而冲突，git会自动将这两个HEAD合并
9. 然后我们进行push

### 为何会出现冲突

因为我们在rebase后，会出现一个新的commit，这个commit的parent就是分支刚建立时的最后一次提交，所以在此时，该分支会同时存在两个HEAD导致冲突，如下图：
![avatar](/res/TIM截图20190819143211.jpg)

所以git会自动将这两个connit进行合并，而合并后的parent就是这两个的commit的哈希值。
![avatar](/res/TIM截图20190819145731.jpg)

## 图示

![avatar](/res/TIM截图20190816153538.jpg)

---
![avatar](/res/TIM截图20190816153733.jpg)

---
![avatar](/res/TIM截图20190816154045.jpg)

---
![avatar](/res/TIM截图20190816154147.jpg)

---
![avatar](/res/TIM截图20190816154225.jpg)

---
![avatar](/res/TIM截图20190816154248.jpg)

---
![avatar](/res/TIM截图20190816154417.jpg)

---
![avatar](/res/TIM截图20190816154449.jpg)

---
