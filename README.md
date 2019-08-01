A compilation of not trivial Git tricks and hacks

# Git advanced cheatsheet

#### Remove tag (local and remote)
```shell script
git push --delete origin TAG
git tag -d TAG
```

#### Stop tracking a commited file (prevent future updates)

**Temporarily** ignore changes:

During development it's convenient to stop tracking file changes to a file committed into your git repo. This is very
convenient when customizing settings or configuration files that are part of your project source for your own work
environment.

> git update-index --assume-unchanged <file>

Resume tracking files with:

> git update-index --no-assume-unchanged <file>

**Permanently** ignore changes to a file:

If a file is already tracked by Git, adding that file to `.gitignore` is not enough to ignore changes to the file.
You also need to remove the information about the file from Git's index:

_These steps will not delete the file from your system._ They just tell Git to ignore future updates to the file.

1. Add the file in your .gitignore.
1. > git rm --cached FILE
1. Commit


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


