This page walks through a demonstration
git-commit-ectomy that you can perform
starting with an empty git repository.

It addresses the **multi-branch case**.

![Painting: Morton and the First Use of Ether (Hinkley 1893).](img/william-morton-first-use-ether.jpg)

<br />
<br />

# table of contents

* [requirements](#requirements)
* [consult with your doctor](#consult-with-your-doctor)
* [an ascii art crash course in surgery](#an-ascii-art-crash-course-in-surgery)
* [demo surgery: setup](#demo-surgery-setup)
    * [create initial shared commit history](#create-initial-shared-commit-history)
    * [create multiple branches](#create-multiple-branches)
* [demo surgery: procedure](#demo-surgery-procedure)
    * [git forget blob: cat\_branch1 (branch specific history)](#git-forget-blob-cat_branch1-branch-specific-history)
    * [git forget blob: cat (shared history)](#git-forget-blob-cat-shared-history)
        * [forget blob in one branch](#forget-blob-in-one-branch)
        * [prepare for rebase by tagging commits](#prepare-for-rebase-by-tagging-commits)
        * [perform rebase operation](#perform-rebase-operation)


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


# an ascii art crash course in surgery

Suppose our patient has a particularly painful 
and unnaturally large commit located in their
multi-branch commit history:

```
            This is the commit
             to be "ectomied"
             \/
             __
    o---o---(__)---o---o--------o---o---o  branch A
                        \        
                         \---o---o---o  branch B
                          \   
                           o---o---o  branch C
```

The commit is first fixed on one of the branches,
creating a new history. However, the other branches
still require the old history to be retained:

```
                     This commit is the last
                      commit shared by the history
                       of each of the branches
                       \/

          ----o----o---o   new (shared) history
         /              \
        |                -------o---o---o  branch A
        |     __
    o---o----(__)----o---o       old (shared) history
                          \        
                           \---o---o---o  branch B
                            \   
                             o---o---o  branch C

```

The second step of the procedure is to rebase
branch B and branch C onto the new history.
The rebase command requires three pieces of
information:

* The source commit (where to cut)
* The destination commit (where to graft)
* The branch name (how much to cut and graft)

Let us label the diagram above with the rebase
source and destination commits, and use valid
git branch names:

```
          ----o----o---o   rebase_dest
         /              \
        |                -------o---o---o  branchA
        |
        |     __
    o---o----(__)----o---o   rebase_src 
                          \        
                           \---o---o---o  branchB
                            \   
                             o---o---o  branchC
```

Now the commands to rebase branch B and branch C are:

```
git rebase --onto rebase_dest rebase_src branchB
git rebase --onto rebase_dest rebase_src branchC
```

The commit history that results will look like the following:

```
          ----o----o---o
         /              \
        |                \-----o---o---o  branchA
        |                 \    
    o----                  \--o---o---o  branchB
                            \ 
                             o---o---o  branchC
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
$ git log --oneline
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
git branch branch1; git branch branch2; git branch branch3
for BRANCH in branch1 branch2 branch3; do

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

The log should now look like the following:

```
$ git log --oneline
2acb13d (branch3) adding branch3_data
a9473c6 (branch2) adding branch2_data
bfc1937 (branch1) adding branch1_data
859fb5d adding rat
7d104ee adding fat
ddf2903 adding dat
765708d adding cat
4c9f26f adding bat
3b92007 adding bar.txt
c2daf61 adding foo.txt
```

Visually, the commit history looks like this:

```
$ git lg2
* 4221760 - 2019-04-09 10:45:30 (branch3)
|             adding branch3_data - C Reid
| * b83c9aa - 2019-04-09 10:45:28 (branch2)
|/            adding branch2_data - C Reid
| * f30c355 - 2019-04-09 10:45:26 (branch1)
|/            adding branch1_data - C Reid
* 859fb5d - 2019-04-09 10:34:19
|           adding rat - C Reid
* 7d104ee - 2019-04-09 10:34:19
|           adding fat - C Reid
* ddf2903 - 2019-04-09 10:34:18
|           adding dat - C Reid
* 765708d - 2019-04-09 10:34:18
|           adding cat - C Reid
* 4c9f26f - 2019-04-09 10:34:18
|           adding bat - C Reid
* 3b92007 - 2019-04-09 10:33:21
|           adding bar.txt - C Reid
* c2daf61 - 2019-04-09 10:33:14
            adding foo.txt - C Reid
```

Or, drawing it with an ascii art tree:

```
o           adding branch3_data
|
|   o       adding branch2_data
|   |
|   |    o  adding branch1_data
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

In this demo surgery, we will show how to remove two files:

* The `cat_branch1` file, which was only added to branch 1
  (this is essentially the equivalent procedure to the 
  single-branch [Easy Method](easy.md).)

* The `cat` file, which was added in a commit that is common to
  multiple branches' commit histories (this is the more complicated case).


## git forget blob: `cat_branch1` (branch specific history)

We walk through how to remove a file that was only added
to one branch.

This procedure is basically equivalent to the
[Easy Method](easy.md), which deals with a single
branch. You are rewriting the last bit of the history
of that one branch. When the old commits are changed
to new commits, there will not be any other branches
pointing to the old history, so things are not complicated.

Start by checking out the branch:

```
$
git checkout branch1
```

Download the git forget blob script:

```
$
wget https://tinyurl.com/git-forget-blob-mac-sh -O git-forget-blob.sh
chmod +x git-forget-blob.sh
```

Check the size of the `.git` directory (before):

```
$ du -hs .git
290M	.git
```

Now run the git forget blob script and pass the
relative path to `cat_branch1` in the repository:

```
$ ./git-forget-blob.sh branch1_data/cat_branch1
Enumerating objects: 89, done.
Counting objects: 100% (89/89), done.
Delta compression using up to 4 threads.
Compressing objects: 100% (81/81), done.
Writing objects: 100% (89/89), done.
Total 89 (delta 18), reused 0 (delta 0)
Rewrite f30c3554eb89d764adb91244197dd62b58f10de2 (7/8) (1 seconds passed, remaining 0 predicted)    rm 'branch1_data/cat_branch1'

Ref 'refs/heads/branch1' was rewritten
Enumerating objects: 43, done.
Counting objects: 100% (43/43), done.
Delta compression using up to 4 threads.
Compressing objects: 100% (33/33), done.
Writing objects: 100% (43/43), done.
Total 43 (delta 7), reused 36 (delta 2)
```

Verify that the file was removed by checking the
size of the `.git` directory (after):

```
$ du -hs .git
100M	.git
```

Also check the log to ensure that branch 1 has a new commit
history, and that no other branches had their histories
affected:

```
$ git lg2
* 4221760 - 2019-04-09 13:45:30 -0700 (4 hours ago) (branch3)
|           adding branch3_data - C Reid
| * b83c9aa - 2019-04-09 13:45:28 -0700 (4 hours ago) (branch2)
|/            adding branch2_data - C Reid
| * 70b607b - 2019-04-09 13:45:26 -0700 (4 hours ago) (HEAD -> branch1)
|/            adding branch1_data - C Reid
* 859fb5d - 2019-04-09 10:34:19 -0700 (7 hours ago)
|           adding data2/rat - C Reid
* 7d104ee - 2019-04-09 10:34:19 -0700 (7 hours ago)
|           adding data2/fat - C Reid
* ddf2903 - 2019-04-09 10:34:18 -0700 (7 hours ago)
|           adding data1/dat - C Reid
* 765708d - 2019-04-09 10:34:18 -0700 (7 hours ago)
|           adding data1/cat - C Reid
* 4c9f26f - 2019-04-09 10:34:18 -0700 (7 hours ago)
|           adding data1/bat - C Reid
* 3b92007 - 2019-04-09 10:33:21 -0700 (7 hours ago)
|           adding bar.txt - C Reid
* c2daf61 - 2019-04-09 10:33:14 -0700 (7 hours ago)
            adding foo.txt - C Reid
```

Now that you have rewritten the history of `branch1`
and replaced each commit with a new one, you can replace
the history of the branch on the remote by doing a 
force-push:

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


### forget blob in one branch

We start by checking on the log (this uses the same
repo as resulted from rewriting branch 1 only):

```
$ git lg2
* 4221760 - 2019-04-09 13:45:30 -0700 (4 hours ago) (branch3)
|           adding branch3_data - C Reid
| * b83c9aa - 2019-04-09 13:45:28 -0700 (4 hours ago) (branch2)
|/            adding branch2_data - C Reid
| * 70b607b - 2019-04-09 13:45:26 -0700 (4 hours ago) (HEAD -> branch1)
|/            adding branch1_data - C Reid
* 859fb5d - 2019-04-09 10:34:19 -0700 (7 hours ago)
|           adding data2/rat - C Reid
* 7d104ee - 2019-04-09 10:34:19 -0700 (7 hours ago)
|           adding data2/fat - C Reid
* ddf2903 - 2019-04-09 10:34:18 -0700 (7 hours ago)
|           adding data1/dat - C Reid
* 765708d - 2019-04-09 10:34:18 -0700 (7 hours ago)
|           adding data1/cat - C Reid
* 4c9f26f - 2019-04-09 10:34:18 -0700 (7 hours ago)
|           adding data1/bat - C Reid
* 3b92007 - 2019-04-09 10:33:21 -0700 (7 hours ago)
|           adding bar.txt - C Reid
* c2daf61 - 2019-04-09 10:33:14 -0700 (7 hours ago)
            adding foo.txt - C Reid
```

We start by removing the large file `cat` from one
of the branches, then rebase the other branches onto
the new history that is created.

Here, we remove the file `cat` from branch 2, and rebase
branches 1 and 3 onto it.

Start by checking out the branch, and checking on the
size of the .git folder:

```
$
git checkout branch2
```

```
$ du -hs .git
100M	.git
```

Now we run the forget blob script:

```
$ ./git-forget-blob.sh data1/cat
Enumerating objects: 43, done.
Counting objects: 100% (43/43), done.
Delta compression using up to 4 threads.
Compressing objects: 100% (28/28), done.
Writing objects: 100% (43/43), done.
Total 43 (delta 7), reused 43 (delta 7)
Rewrite 765708d262678c2bbc15dd5b999863ad5a4dab01 (4/8) (0 seconds passed, remaining 0 predicted)    rm 'data1/cat'
Rewrite ddf2903c6d1fe41b171af7764579bc4d6a63e70f (5/8) (0 seconds passed, remaining 0 predicted)    rm 'data1/cat'
Rewrite 7d104ee90da6b3adf08f9a86d2342e3c8690dc64 (6/8) (0 seconds passed, remaining 0 predicted)    rm 'data1/cat'
Rewrite 859fb5d7db420b8aa745c50162b71c20331e90a9 (7/8) (0 seconds passed, remaining 0 predicted)    rm 'data1/cat'
Rewrite b83c9aaecf238324aa2b8a51da1ac7f69015d4d4 (8/8) (0 seconds passed, remaining 0 predicted)    rm 'data1/cat'

Ref 'refs/heads/branch2' was rewritten
Enumerating objects: 51, done.
Counting objects: 100% (51/51), done.
Delta compression using up to 4 threads.
Compressing objects: 100% (38/38), done.
Writing objects: 100% (51/51), done.
Total 51 (delta 9), reused 40 (delta 5)
```

Check the size of the `.git` directory:

```
$ du -hs .git
100M	.git
```

It did not change! We can see why when we check the log:

```
$ git lg1
* 4221760 - (4 hours ago) adding branch3_data - C Reid (branch3)
| * 5e2b657 - (4 hours ago) adding branch2_data - C Reid (HEAD -> branch2)
| * 6ec8be5 - (7 hours ago) adding data2/rat - C Reid
| * 15ec3be - (7 hours ago) adding data2/fat - C Reid
| * 03a671e - (7 hours ago) adding data1/dat - C Reid
| * 3cda75c - (7 hours ago) adding data1/cat - C Reid
| | * 70b607b - (4 hours ago) adding branch1_data - C Reid (branch1)
| |/
|/|
* | 859fb5d - (7 hours ago) adding data2/rat - C Reid
* | 7d104ee - (7 hours ago) adding data2/fat - C Reid
* | ddf2903 - (7 hours ago) adding data1/dat - C Reid
* | 765708d - (7 hours ago) adding data1/cat - C Reid
|/
* 4c9f26f - (7 hours ago) adding data1/bat - C Reid
* 3b92007 - (7 hours ago) adding bar.txt - C Reid
* c2daf61 - (7 hours ago) adding foo.txt - C Reid

```

branch 1 and branch 3 still refer to the old commit that added `cat`,
so the full contents of the `cat` file are still in the `.git` directory's
blobs. 

Now that branch 2 has a new version of the shared history,
we want to rebase branch 1 and branch 3.


### prepare for rebase by tagging commits

Start the rebase operation for the remaining branches by
labeling our rebase source and destination commits. As
shown in the ascii art diagrams at the top of the page,
we want to find the last commit that our branches shared
in common in their commit history, and mark that commit
in the old history as the "rebase source" and mark that
commit in the new history as the "rebase destination".

In the case of our example, the commit that adds the file
`rat` is the last commit all our branches have in common.

This means that the rebase source will be the version of 
the "adding rat" commit that is in the old history (the 
history of branch 1 and branch 3) - commit `859fb5d`.

The rebase destination will be the version of the "adding
rat" commit that is in the new history of branch 2,
which corresponds to commit `6ec8be5`.

Mark these two commits as the `rebase_src` and `rebase_dest`
branches:

```
git checkout 859fb5d
git checkout -b rebase_src

git checkout 6ec8be5
git checkout -b rebase_dest
```

Now here is the state of the repo:

```
$ git lg1
* 4221760 - (5 hours ago) adding branch3_data - C Reid (branch3)
| * 5e2b657 - (5 hours ago) adding branch2_data - C Reid (branch2)
| * 6ec8be5 - (8 hours ago) adding data2/rat - C Reid (HEAD -> rebase_dest)
| * 15ec3be - (8 hours ago) adding data2/fat - C Reid
| * 03a671e - (8 hours ago) adding data1/dat - C Reid
| * 3cda75c - (8 hours ago) adding data1/cat - C Reid
| | * 70b607b - (5 hours ago) adding branch1_data - C Reid (branch1)
| |/
|/|
* | 859fb5d - (8 hours ago) adding data2/rat - C Reid (rebase_src)
* | 7d104ee - (8 hours ago) adding data2/fat - C Reid
* | ddf2903 - (8 hours ago) adding data1/dat - C Reid
* | 765708d - (8 hours ago) adding data1/cat - C Reid
|/
* 4c9f26f - (8 hours ago) adding data1/bat - C Reid
* 3b92007 - (8 hours ago) adding bar.txt - C Reid
* c2daf61 - (8 hours ago) adding foo.txt - C Reid
```


### perform rebase operation

Start the rebase operation by checking out branch 2:

```
$
git checkout branch2
```

Now rebase branch 1 from the old history onto the new history:

```
$ 
git rebase --onto rebase_dest rebase_src branch1
```

You should see output like this:

```
First, rewinding head to replay your work on top of it...
Applying: adding branch1_data
```

Verify that branch 1 and branch 2 now share a commit history:

```
$ git lg1
* 2af1225 - (5 hours ago) adding branch1_data - C Reid (HEAD -> branch1)
| * 4221760 - (5 hours ago) adding branch3_data - C Reid (branch3)
| * 859fb5d - (8 hours ago) adding data2/rat - C Reid (rebase_src)
| * 7d104ee - (8 hours ago) adding data2/fat - C Reid
| * ddf2903 - (8 hours ago) adding data1/dat - C Reid
| * 765708d - (8 hours ago) adding data1/cat - C Reid
| | * 5e2b657 - (5 hours ago) adding branch2_data - C Reid (branch2)
| |/
|/|
* | 6ec8be5 - (8 hours ago) adding data2/rat - C Reid (rebase_dest)
* | 15ec3be - (8 hours ago) adding data2/fat - C Reid
* | 03a671e - (8 hours ago) adding data1/dat - C Reid
* | 3cda75c - (8 hours ago) adding data1/cat - C Reid
|/
* 4c9f26f - (8 hours ago) adding data1/bat - C Reid
* 3b92007 - (8 hours ago) adding bar.txt - C Reid
* c2daf61 - (8 hours ago) adding foo.txt - C Reid
```

Repeat the process with branch 3:

```
$
git rebase --onto rebase_dest rebase_src branch3
```

You will see output like this:

```
First, rewinding head to replay your work on top of it...
Applying: adding branch3_data
```

Verify that all three branches now share a commit history:

```
$ git lg1
* b5642eb - (5 hours ago) adding branch3_data - C Reid (HEAD -> branch3)
| * 2af1225 - (5 hours ago) adding branch1_data - C Reid (branch1)
|/
| * 5e2b657 - (5 hours ago) adding branch2_data - C Reid (branch2)
|/
* 6ec8be5 - (8 hours ago) adding data2/rat - C Reid (rebase_dest)
* 15ec3be - (8 hours ago) adding data2/fat - C Reid
* 03a671e - (8 hours ago) adding data1/dat - C Reid
* 3cda75c - (8 hours ago) adding data1/cat - C Reid
| * 859fb5d - (8 hours ago) adding data2/rat - C Reid (rebase_src)
| * 7d104ee - (8 hours ago) adding data2/fat - C Reid
| * ddf2903 - (8 hours ago) adding data1/dat - C Reid
| * 765708d - (8 hours ago) adding data1/cat - C Reid
|/
* 4c9f26f - (8 hours ago) adding data1/bat - C Reid
* 3b92007 - (8 hours ago) adding bar.txt - C Reid
* c2daf61 - (8 hours ago) adding foo.txt - C Reid
```

Last, delete the rebase source and destination branches, so nothing points
at the problematic commits or the old history:

```
$
git branch -D rebase_src rebase_dest
```

Check the log to make sure we only see the new history:

```
$ git lg1
* b5642eb - (5 hours ago) adding branch3_data - C Reid (HEAD -> branch3)
| * 2af1225 - (5 hours ago) adding branch1_data - C Reid (branch1)
|/
| * 5e2b657 - (5 hours ago) adding branch2_data - C Reid (branch2)
|/
* 6ec8be5 - (8 hours ago) adding data2/rat - C Reid (rebase_dest)
* 15ec3be - (8 hours ago) adding data2/fat - C Reid
* 03a671e - (8 hours ago) adding data1/dat - C Reid
* 3cda75c - (8 hours ago) adding data1/cat - C Reid
* 4c9f26f - (8 hours ago) adding data1/bat - C Reid
* 3b92007 - (8 hours ago) adding bar.txt - C Reid
* c2daf61 - (8 hours ago) adding foo.txt - C Reid
```

