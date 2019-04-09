This page walks through a demonstration
git-commit-ectomy that you can perform
starting with an empty git repository.
It addresses the multi-branch case.

<br />
<br />

# table of contents

* [requirements](#requirements)
* [consult with your doctor](#consult-with-your-doctor)
* [demo surgery: setup](#demo-surgery-setup)
    * [step 1: create single branch](#step-1-create-single-branch)
    * [step 2: create multi\-branch split](#step-2-create-multi-branch-split)

<br />
<br />

# requirements

Before you begin, read through the [Easy Way](easy.md)
(even if it does not apply to your case) so that you
are familiar with how the process works. This page
will cover a slightly more complicated git-commit-ectomy.

# consult with your doctor

You should consult with your doctor to determine if a 
git-commit-ectomy is right for you.

This one-liner lists the 40 largest files in the repo:

```
$ 
git rev-list --all --objects | \
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


# demo surgery: setup

To set up the demo surgery, we will start by
creating a single branch with commits that 
are shared history between multiple branches.

Next, we create several branches with their
own commits not shared with other branches.

We want to show how to remove large files from
both the shared history of multiple branches,
and from a single branch's history.


## create initial shared commit history

Clone an example repo for performing surgery. You don't _need_
a remote repository to do the demo surgery, but we will use one
in our walkthrough.

```
$ 
git clone https://github.com/charlesreid1/git-commit-ectomy-example
cd git-commit-ectomy-example
```

In this example we will create a branch called `branch1` instead of
using the default `master` branch. Start by renaming the `master` branch
to `branch1`:

```
$
git branch branch1
git checkout branch1
```

Start with several text files, and add them to the repo history:

```
$
echo "hello foo" > foo.txt
echo "hello bar" > bar.txt

for item in `/bin/ls -1 *.txt`; do
    git add ${item} && git commit ${item} -m "adding ${item}"
done
```

Now add several large files to the repo history:


```
$ 
mkdir data1; mkdir data2
```

Now create some files in each of the two directories:

```
$ 
cd data1/
dd if=/dev/urandom of=bat bs=1048576 count=10
dd if=/dev/urandom of=cat bs=1048576 count=10
dd if=/dev/urandom of=dat bs=1048576 count=10
cd ../


cd data2/
dd if=/dev/urandom of=fat bs=1048576 count=10
dd if=/dev/urandom of=rat bs=1048576 count=10
cd ../
```

This gives the following directory structure:

```
$
tree .
.
├── bar.txt
├── data1
│   ├── bat
│   ├── cat
│   └── dat
├── data2
│   ├── fat
│   └── rat
└── foo.txt
```

Add the files to the repo in _separate commits_:

```
$
for item in data1/bat data1/cat data1/dat data2/fat data2/rat; do
    git add ${item} && git commit ${item} -m "adding ${item}"
done
```

At this point the git log should look like this:

```
$
git log --oneline
859fb5d (branch1) adding rat
7d104ee adding fat
ddf2903 adding dat
765708d adding cat
4c9f26f adding bat
3b92007 adding bar.txt
c2daf61 adding foo.txt
```

(So far, this is identical to the single-branch setup.)


## create multiple branches

Commit `859fb5d` (where we added the file `rat`) is the
last commit that is shared among the history of the three
branches we will create.

We start by checking out a particular commit and
creating each branch; this is equivalent to tagging
the commit `859fb5d` with three labels, `branch1`,
`branch2`, and `branch3`.

Next we add different files to the different branches.
Here is the directory structure we will create:

```
Branch 1               Branch 2               Branch 3                    
-------------------    --------------------   ----------------------      
                                                                          
.                      .                      .                           
├── bar.txt            ├── bar.txt            ├── bar.txt                 
├── foo.txt            ├── foo.txt            ├── foo.txt                 
├── data1              ├── data1              ├── data1                   
│   ├── bat            │   ├── bat            │   ├── bat                 
│   ├── cat            │   ├── cat            │   ├── cat                 
│   └── dat            │   └── dat            │   └── dat                 
├── data2              ├── data2              ├── data2                   
│   ├── fat            │   ├── fat            │   ├── fat                 
│   └── rat            │   └── rat            │   └── rat                 
│                      │                      │
└── branch1_data       └── branch2_data       └── branch3_data             
    ├── branch1.txt        ├── branch2.txt        ├── branch3.txt         
    ├── bat_branch1        ├── bat_branch2        ├── bat_branch3         
    └── cat_branch1        └── cat_branch2        └── cat_branch3         
```

Here are the commands to create the branch-specific 
files and directories:

```
$
for BRANCH in branch1 branch2 branch3; do

    git branch ${BRANCH}
    git checkout ${BRANCH}
    mkdir ${BRANCH}_data
    cd ${BRANCH}_data
    echo "hello ${BRANCH}" > ${BRANCH}.txt
    dd if=/dev/urandom of=bat_${BRANCH} bs=1048576 count=10
    dd if=/dev/urandom of=cat_${BRANCH} bs=1048576 count=10
    cd ../
    git add ${BRANCH}_data
    git commit ${BRANCH}_data -m "adding ${BRANCH}_data"

done
```







We illustrate the multi-branch case with the following
git repository branch structure:

```
o           adding cat_branch1
|
o           adding bat_branch1
|
o           adding branch1.txt
|
|   o       adding cat_branch2
|   |
|   o       adding bat_branch2
|   |
|   o       adding branch2.txt
|   |
|   |   o   adding cat_branch3
|   |   |
|   |   o   adding bat_branch3
|   |   |
|   |   o   adding branch3.txt
\   |   /
 \  |  /
  \ | /
   \|/
    o       adding rat
    |
    o       adding fat
    |
    o       adding dat
    |
    o       adding cat
    |
    o       adding bat
    |
    o       adding bar.txt
    |
    o       adding foo.tx
    |
    |
   [ ] 

```

# demo surgery: procedure

In this demo surgery, we will show how to remove two types of files:

* The `cat_branch1` file, which was added in a commit that is specific
  to a single branch's commit history (this is essentially equivalent
  to the [Easy Method](easy.md).

* The `cat` file, which was added in a commit that is common to
  multiple branches' commit histories (this is the more complicated case).


## git forget blob: `cat_branch1` (branch specific history)

We walk through how to remove a file added in a commit that
only exists in the commit history of a single branch.

This procedure is basically equivalent to the
[Easy Method](easy.md), which deals with a single
branch. You are rewriting the history of one branch,
so there will not be any other branches pointing to
the old history when you are finished.

Start by checking out the branch:

```
$
git checkout branch1
```

Check the size of the `.git` directory (before):

```
$
du -hs .git
```

Now run the git forget blob script and pass the
relative path to `cat_branch1` in the repository:

```
$
git-forget-blob.sh data1/cat_branch1
```

Verify that the file was removed by checking the
size of the `.git` directory (after):

```
$
du -hs .git
```

Now that you have rewritten the history of `branch1`
and replaced each commit with a new one, force-push 
the rewritten history to the remote:

```
$
git push origin master --force
```


## git forget blob: `cat` (shared history)

Now we walk through how to forget `cat`, a file
that was added in a commit that is shared between
two or more branches. 

This case is more complicated, because once we
rewrite the commit history of branch 1 to forget
the commit where `cat` was added, branch 2 and 
branch 3 still point to the old commit history.

To fix this, we graft branch 2 and branch 3 from
the old commit history onto the new commit history,
by picking two commits that are identical (but that
have different hashes) and using those as the 
source and destination of our rebase operation.


### forget blob in branch 1

We can remove a large file from any of the three
branches; in this case we start with branch 1.

Following the procedure above, we start by
checking out the branch, and checking on the
size of the .git folder:

```
$
git checkout branch1
du -hs .git
```

Now we run the forget blob script, and verify it
shrank the `.git` directory:

```
$
git-forget-blob.sh data1/cat
du -hs .git
```

Now, branch 1 has a different commit history 
from branch 2 and branch 3, going all the way
back to the commit that added `cat`:

```
o           Add cat_branch1
|
o           Add bat_branch1
|
o           Add branch1.txt
|
|   o       Add cat_branch2
|   |
|   o       Add bat_branch2
|   |
|   o       Add branch2.txt
|   |
|   |   o   Add cat_branch3
|   |   |
|   |   o   Add bat_branch3
|   |   |
|   |   o   Add branch3.txt
\   |   /
 \  |  /
  \ | /
   \|/
    o       Add rat
    |
    o       Add fat
    |
    o       Add dat
    |
    o       Add cat
    |
    o       Add bat
    |
    o       Add bar.txt
    |
    o       Add foo.tx
    |
    |
   [ ] 

```


### git rebase branch 2 and branch 3














