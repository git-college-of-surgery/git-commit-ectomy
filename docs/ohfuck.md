# Oh Fuck: Please Send Backup

Pretty much what happened to me,
when I used the single-branch procedure
instead of the multi-branch procedure.

Possible complications to all of this 
include:

* Clueless individuals who do not
    follow basic instructions and continue to push
    old versions of the repo, which can be proactively
    addressed by branch protection but sometimes
    must also be retroactively dealt with; 

* Accidentally carrying out the procedure multiple times,
    resulting in 2, 3, 4, even 5 copies of each commit,
    and the duplicate commits simply refuse to die

* A confusing, tangled, messy commit history that is
    completely uninterpretable


### the first case

In the first case, start by turning on branch protection,
then clone a fresh copy of the repository (complete with
all the duplicate commits)


### the second case

In the second case, find the branch you want to keep,
then rebase or cherry pick any missing commits onto it.
Now remove all the other branches. If these remote branches
are not being removed, make sure you have your syntax right,
and make sure you're using the `--force` command.

```
git push remote_name :branch_to_delete
```

For example, if I want to delete the branch
`master` because I am creating a branch `new_master`
with improved/cleaned history,
I would do the following:

```
git checkout <hash-of-commit-to-split-at>   # starting point for new branch
git branch new_master                       # create new branch
git checkout new_master                     # switch to new branch

...make some commits...
...rebase some stuff...
...cherry pick some stuff...
...now new branch has a long and totally different 
    commit history from master...

git push origin new_master                  # push all the new stuff to remote "origin"

git branch -D master                        # delete git branch master
git push origin :master                     # propagate deletion of branch "master" to remote "origin"
```


### the third case

If you have an absolute clusterfuck of a commit history,
you need a gifted surgeon. The more gifted the surgeon,
the more of your repo history you'll be able to retain.

If you want to perform surgery with a battle axe, 
you can simply leapfrog a whole series of complicated
splits, merges, rebases, and just jump to a point in 
the repo where things are saner and calmer.

Clone a fresh copy of the repo, and checkout the 
commit where the clusterfuck is over, when things
are saner and calmer:

```
git checkout <commit-hash>
```

Now, you're going to copy every single file
in that folder into your current (cluster-fucked)
repo, exactly, word for word, character for character.

If you have extra files, those are okay. If you have 
deleted files that are in the saner, calmer commit state,
you must add them back in.

Once everything matches, commit your changes.
Tag this commit as your "Stargate" - this is the 
commit that will allow you to rebase from 
one branch to the other. The repo will be in
exactly the same state (with exception of extra files)
between these two commits, so changes to one commit
can be applied to the other just fine.

Note that you will lose all information about commits 
that are not rebased or cherry picked, i.e., all the 
commits that were involved with the clusterfuck.

