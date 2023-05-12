---
title: "Create a pull request from a reverted remote branch"
seoTitle: "Create a pull request from a reverted remote git branch"
seoDescription: "Create a pull request from a reverted remote git branch"
datePublished: Fri Oct 15 2021 17:44:26 GMT+0000 (Coordinated Universal Time)
cuid: ckusnunkj1vmaebs19xgi3e7g
slug: create-a-pull-request-from-a-reverted-remote-branch
tags: github, git, gitlab

---

Recently I landed across the situation where I had made commits directly to `dev` branch (which happens to be our `release` branch) instead of `feature` branch.

The general process that is followed in my company (and perhaps many others) is that we make changes to our `feature` branch and raise a pull request against `dev` (release) branch which is then reviewed and finally merged.

Accidently I pushed commits directly to `dev` branch instead of `feature` branch & I quickly reverted those commits back by simple git command
```
git revert <revert-commit-hash>
git commit -m "reverting accidental commits"
git push 
```

After that I quickly checked out to my `feature` branch, made those changes again, pushed all changes to `feature` branch. As per next step, I had to raise PR of my latest changes against `dev` branch, and when I did that, I got message:
> There isn’t anything to compare.   
> dev is up to date with all commits from feature branch. Try switching the base for your comparison.

The problem here is `dev` branch is ahead of the `feature` branch (because of revert commit). You will notice that your previous work which was reverted will be gone! The solution to this is reverting the reverted commit!

I had to follow 4 steps to fix this problem which I read from this [awesome article](https://medium.com/@shanikae/create-pull-request-from-a-reverted-git-branch-27219cadf9b5)

**Step 1:** Create and checkout to a new branch tracking `dev` branch
```
git checkout -b temp-branch -t dev
```

**Step 2:** Revert the commit that was made by mistake (here commit will be reverted back from dev branch since our temp branch is tracking dev)
```
git revert <revert-commit-hash>
git commit -m "reverting the reverted commits"
git push -u origin temp-branch
```

**Step 3:** Checkout the original `feature` branch
```
git checkout feature-branch
```

**Step 4:** Raise PR against dev branch
```
git push -u origin feature-branch
```

### Pictorial representation of scenarios discussed above:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634319597245/4p44KUwbW.png)
