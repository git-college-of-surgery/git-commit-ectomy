# performing a transplant with patch

![Painting: The first successful kidney transplantation, Brigham Hospital, Harvard University, 1954.](img/first-successful-kidney-transplantation-brigham-1954-babb-1996.jpg)

<br />
<br />

if you are not able to save the branches as in the
[Multi-Branch Git-Commit-Ectomy Method](branches.md),
you can still save the changes that a particular
set of commits made to a particular file or set of
files using the `patch` utility together with `git`.

## creating patches with git

git has the ability to create patches two ways.

The first way is using `git format-patch`, which
creates one patch per commit. This method preserves
meta-information about the commit and is useful for
transerring changes one commit at a time.

The second way is using `git diff --patch`, which
simply outputs the difference between the two commits,
without any meta-information.

We cover the latter method, since it is more general
and can apply to ranges of commits.

### branch to branch

Suppose you have some changes to the code that live in
a separate branch, and you wish to save those changes 
for later (after the git-commit-ectomy is finished).

For example:

```
                   B   master
A  o---o---o---o---o
    \
     \
      o---o---o---o 
                  C   feature_branch
```

To create a patch from the feature branch, it is best to
run `git diff` to get the diff between the two branches,
B and C, rather than the diff between the feature branch
and the original commit, A and C.

To produce the patch:

```
git diff --patch feature_branch master > make_feature_branch.patch
```

Note that this will _not_ output diff information for binary files.

To apply the patch, use the `git am` command
(note, of all the git subcommand,s this subcommand
has the _absolute worst name imaginable_):

```
git am < make_feature_branch.patch 
```

Alternatively, it is recommended that you keep the patch in
cold storage, and only apply it to the master branch when the
git commit ectomy is finished.

