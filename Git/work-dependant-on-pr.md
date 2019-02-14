# Work on a branch depending on another branch which in under review

Source for solutions: <https://softwareengineering.stackexchange.com/questions/351727/working-on-a-branch-with-a-dependence-on-another-branch-that-is-being-reviewed>

## The issue

You are working on **two** features:

- `feature_a`
- `feature_b`

The first feature, `feature_a` is done, and you have submitted the pull request. It is currently in review. Meanwhile, you want to work on `feature_b`, however it is dependant on `feature_a` being merged. 

How can you continue your work?

## Solution

There are a bunch of possible ways to continue development while `feature_a` is in review. My personal favorite is:

1. Start `feature_b` from `feature_a`:

```bash
git checkout feature_a
git checkout -b feature_b
```

2. Whenever `feature_a` changes while it is waiting to get merged into `master`, you _rebase_ `feature_b` on it:

```bash
# ... commit something onto feature_a
git checkout feature_b
git rebase feature_a
```

3. Finally, as soon as `feature_a` has been merged into `master`, you simply get the new `master` and rebase `feature_a` onto it a last time:

```bash
git checkout master
git pull origin master
git checkout feature_b
git rebase --onto master feature_a feature_b
```
