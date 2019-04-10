# Oh F&!k: Please Send Backup

When you use the single-branch procedure
instead of the multi-branch procedure...
this is what happens.

Possible complications to all of this 
include:

* Users who did not get the memo about rewriting the entire
    history of the git reopsitory and did not clone a fresh
    copy of the repo once the new history was pushed, and continue
    to push the old history to the remote. (This can be proactively
    dealt with by using branch protection, but you may need to
    retroactively deal with this problem);

* Accidentally carrying out the procedure multiple times,
    resulting in 2, 3, 4, even 5 copies of each commit,
    and the duplicate commits simply refuse to die;

* A confusing, tangled, messy commit history that is
    completely uninterpretable


### the first case: users pushing old history

When the git-commit-ectomy is complete, it is recommended
that you turn on branch protection right away. But in case
you did not, and a user pushed the old history to the remote,
here is how to deal.

From a copy of the repository with _only_ the new history
(this is going to be the copy of the repository that is
on the surgeon's computer, preferrably the repo in which
the git-commit-etomy was originally performed),
run the command `git push origin master --force`
(where `origin` is the remote with the duplicate history,
and `master` is the name of the branch you're pushing. 
Replace these with whatever remote/branch names you are
using.)

Turn on branch protection immediately afterwards.

**If you do not have a copy of the repository with
_only_ the new history** (i.e., if you removed the
copy of the repository in which the git-commit-ectomy
was performed), you _can_ still separate
the old and new histories. However, you _must_ have
a copy of the old git log and/or the new git log,
so that you have some way of knowing which commits
belong to which history. If you have this info,
continue on to "the second case."

### the second case: scrubbling multiple attempts

If you have performed a git-commit-ectomy multiple times,
and the duplicate commits are simply piling up and will 
not go away, you will need to use either a git rebase
or a git cherry pick operation.

Find the branch that you want to keep, and the commit
at which to start the new branch with the new history.

Then rebase or cherry pick any missing commits onto it.

Finally, remove all other branches on the remote.

**Example:** Suppose we are trying to fix the history
of the `master` branch, which is a mess, by creating
a new branch `new_master` that has an improved and 
cleaned-up version of the `master` branch history.

We start by checking out the particular commit where
we wish `new_master` to diverge from `master`:

```
git checkout <hash-of-commit-to-split-at>   # starting point for new branch
git branch new_master                       # create new branch
git checkout new_master                     # switch to new branch
```

Next, we modify the repository by making commits,
doing git rebase and git cherry pick operations,
and creating a new branch with a long and totally
separate commit history from master.

```
...make some commits...
...rebase some stuff...
...cherry pick some stuff...
...now new branch has a long and totally different 
    commit history from master...
```

Finally, when the `new_master` branch is ready, push it
to the remote (e.g., `origin`):

```
git push origin new_master                  # push all the new stuff to remote "origin"
```

The last step is to delete all other branches except for
the `new_master` branch, so that we don't preserve the 
messy history of the `master` branch. (The whole point of
the `new_master` branch was to clean it up.)

Deleting the master branch on the remote is a two step 
operation: first, delete the branch locally, then push
the deleted "ghost branch" to the remote (which will
propagate the deletion of the branch):

```
git branch -D master                        # delete git branch master locally
git push origin :master                     # propagate deletion of branch "master" to remote "origin"
```

Note the syntax for deleting a branch is:

```
git push <name-of-remote> :<name-of-branch-to-delete>
```

(If you have trouble removing the remote branches, 
double-check your syntax is right, and make sure you
use the `--force` flag.)

### the third case: a surgeon's nightmare

If you have an absolute clusterf--k of a commit history,
you need a gifted surgeon. The more gifted the surgeon,
the more of your repo history you'll be able to retain.

Here is a minimal-complications method that removes more 
of your history than you'd probably want
(see above note about finding a gifted surgeon). 

This method is to leapfrog a whole series of 
complicated splits, merges, rebases, and other tangles,
and jump directly to a point in the repo where things 
are saner and calmer. This is done by creating two commits
(two repo states) that are exactly identical, and which
can be used as the source and destination of a git rebase 
operation.

The battle axe method requires adding one commit to 
the clusterf--ked branch, which will modify the state
of the repo to match precisely the state of the repo
where things are saner and calmer.

To do this, check out the repo commit where the
clusterf--k is over, when things are saner and
calmer:

```
git checkout <commit-hash>
```

Now, you're going to copy every single file 
in that folder into your current (clusterf--ked)
repo, _exactly, word for word, character for character_.

If there are extra files in your current (clusterf--ked)
repo, those are okay and can be left alone. If you have
files that are in the saner, calmer commit state, you
must add those files in to your current (clusterf--ked)
repo.

Once everything matches the calmer/saner state, 
commit your changes.

Tag these two commits as your "Stargates" - these two
commits are linked by having the repository in the exact
same state (with exception of extra files), which allows
you to rebase from one branch to the other. Any changes
made to files in the repo can be transferred without
conflicts.

Note that you will lose all information about commits 
that are not rebased or cherry picked, i.e., all the 
commits that were involved with the clusterf--k.

