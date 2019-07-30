# Git advanced cheatsheet
A compilation of Git tricks and hacks

#### Cherry-pick from another repo (using patch)

```shell script
cd taret-repo
git --git-dir=../<some_other_repo>/.git \
format-patch -k -1 --stdout <commit SHA> | \
git am -3 -k
```

(`-1` = single commit; `-3` = 3-way merge)

#### Move a file between repos preserving history

(Option A) Using patch (if history is sane):
```shell script
git log --pretty=email --patch-with-stat --reverse -- path/to/file_or_folder | (cd /path/to/new_repository && git am)
```

(Option B) Adding the source repo as temporary remote:
```shell script
git remote add srcrepo https://example.link/source-repository.git
git fetch srcrepo
git merge srcrepo/srcbranch --allow-unrelated-histories
git remote rm srcrepo
```
And resolve potential conflicts.

**N.B.** `fetch` and `merge` can be replaced by `pull srcrepo srcbranch`


