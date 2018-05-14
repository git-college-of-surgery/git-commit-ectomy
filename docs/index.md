# git-commit-ectomy

Perform a git-commit-ectomy to forever remove problematic commits from your repo history.

This uses the [git-forget-blob.sh](https://tinyurl.com/git-commit-ectomy) script from @nachoparker.

![git-commit-ectomy main banner image](/img/git-commit-ectomy.jpg)

[Visit the Git College of Surgery on Github](https://github.com/git-college-of-surgery)

This surgery can happen one of three ways:

* [Git-Commit-Ectomy the Easy Way: Single Branch](easy.md)

* [Complications: Dealing with Branches](branches.md)

* [Git-Commit-Ectomy the Hard Way: Cherry Picking](cherrypicking.md)


# consult with your doctor

You should consult with your doctor to determine if a 
git-commit-ectomy is right for your repository.

This one-liner lists the 40 largest files in the repo
(modify the `tail` line to change the number):

```
$ git rev-list --all --objects | \
     sed -n $(git rev-list --objects --all | \
     cut -f1 -d' ' | \
     git cat-file --batch-check | \
     grep blob | \
     sort -n -k 3 | \
     \
     tail -n40 | \
     \
     while read hash type size; do
          echo -n "-e s/$hash/$size/p ";
     done) | \
     sort -n -r -k1 
```

