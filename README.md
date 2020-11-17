A compilation of not trivial Git tricks and hacks

# Git advanced cheatsheet

#### Misc
Unset remote tracking branch: `git branch --unset-upstream`

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

(Option C) Fetching a repository branch:

```
git fetch https://example.link/source-repository.git master
gIt checkout master
git merge FETCH_HEAD
```
And resolve potential conflicts.

**N.B.** `fetch` and `merge` can be replaced by `pull srcrepo srcbranch`

#### Merge a whole remote repository into the current one (subtree merge)

This will allow you to migrate code from repository `srcrepo` to repository `dstrepo` while
preserving `srcrepo`'s history and authorship of files. This can be done into a specific prefix path
of destination repository instead of merging into the root, effectively allowing the merge of two
different projects together.

```shell script
git remote add -f srcrepo https://example.link/source-repository.git
git merge -s ours --no-commit --allow-unrelated-histories srcrepo/master
git read-tree --prefix=path/to/destination -u srcrepo/master
git commit -m "Srcrepo merged into dstrepo"
git remote rm srcrepo
```

Notes (in command order):
  * `-f` stands for `fetch`, so add remote and fetch it.
  * `-s` ours makes it not apply the changes, as that would apply them to the root of the repository
    (not to a subdirectory).
  * `-u` makes it apply them not only to index, but also to the working directory.

More about it:
  * https://gist.github.com/x-yuri/8ad01701db51ec2891ca431b78c58c72
  * https://stackoverflow.com/a/32684526/6108874

## Squash all commits in a branch (disregarding it has merge-commits in between)

To rebase a branch upon the point it was born, e.g. `develop` and squash all commits in that branch
(typical use case can be a shared branch with frequent _pulls_ from develop), do the following:

```shell
git reset $(git merge-base origin/develop YOUR_BRANCH)   # or just `develop` if updated with origin
git add -A
git commit
```

All the commits in `YOUR_BRANCH` will be gone, leading to a single final commit on top of `develop`.

**N.B.** Rebasing rewrites the history, so be sure the branch is no longer shared when you squash
it. Finally do a `push --force` if the branch was already pushed.
