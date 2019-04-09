This page walks through a demonstration
git-commit-ectomy that you can perform
starting with an empty git repository.
It addresses the single-branch case.

<br />
<br />

# table of contents

* [requirements](#requirements)
* [consult with your doctor](#consult-with-your-doctor)
* [demo surgery: setup](#demo-surgery-setup)
    * [side note: how to make a fat file](#side-note-how-to-make-a-fat-file)
    * [make some text files](#make-some-text-files)
    * [make some fat files](#make-some-fat-files)
    * [commit files](#commit-files)
* [demo surgery: procedure](#demo-surgery-procedure)
    * [prepare tools](#prepare-tools)
    * [the command that doesn't work: git rm](#the-command-that-doesnt-work-git-rm)
    * [the command that does work: git forget blob](#the-command-that-does-work-git-forget-blob)
    * [how it worked](#how-it-worked)
    * [stitch the patient back up](#stitch-the-patient-back-up)
* [tips for surgery](#tips-for-surgery)

<br />
<br />

# requirements

This guide utilizes GNU xargs. 
You should run it on Linux, 
or use Homebrew's gxargs if 
on a Mac.


# consult with your doctor

You should consult with your doctor to determine if a 
git-commit-ectomy is right for you.

This one-liner lists the 40 largest files in the repo
(modify the `tail` line to change the number):

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

When you're ready to perform the surgery, append a `cut` command to get the
relative path to the file _only_, without listing the size of the file, which
is what we will need when we carry out the git-commit-ectomy:

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
     sort -n -r -k1 | \
     cut -f 2 -d' ' 
```


# demo surgery: setup

Clone an example repo for performing surgery. You don't _need_
a remote repository to do the demo surgery, but we will use one
in our walkthrough.

```
$ 
git clone https://github.com/charlesreid1/git-commit-ectomy-example
```

## side note: how to make a fat file

We will use the `dd` command to create files with a specified number
of bits. For example, to create a 10 MB file, we can issue the command:

```text
$ 
dd if=/dev/urandom of=my_big_fat_file bs=1048576 count=10
```

**Important:** You must use `/dev/urandom` with a non-zero block size.
If you use `/dev/zeros` then each file will be identical and git 
will not store them separately. Then your surgery will go very badly.

**Note:** `1048576 = 2^20` bytes comes from
the fact that 1 KB = `2^10` bytes, and 1 MB = `2^10` KB, 
for a total of `2^20` bytes per megabyte.

`count=10` means we make 10 blocks, each of size 1 MB (1048576 bytes).

## make some text files

We start by adding some small boring text files to the repository:

```
$ 
echo "hello foo" > foo.txt; echo "hello bar" > bar.txt
```

Now add them to the repo history:

```
$ 
for item in `/bin/ls -1 *.txt`; do
    git add ${item} && git commit ${item} -m "adding ${item}"
done
```


## make some fat files

To demonstrate the importance of specifying the path to the
large files being removed from the repository, we add several
10 MB files inside of a subdirectory. Start with the directory
structure:

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

Now we have the following directory structure:

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

```
$ 
ls -lhg data1
-rw-r--r--   1 staff    10M Apr 10 18:30 bat
-rw-r--r--   1 staff    10M Apr 10 18:30 cat
-rw-r--r--   1 staff    10M Apr 10 18:30 dat

$ 
ls -lhg data2
-rw-r--r--   1 staff    10M Apr 10 18:30 fat
-rw-r--r--   1 staff    10M Apr 10 18:30 rat
```

Also make sure they are unique (hence `/dev/random` and not `/dev/zero`):

```
$ 
for i in `/bin/ls -1 data1/*at data2/*at`; do
    md5 ${i}
done

MD5 (bat) = 140c7d1e5a12c0eb2fefd5529250a280
MD5 (cat) = 9345ca4033c7e42fb009e3b8014570dc
MD5 (dat) = fadc3114fe9a70f688eba0db4e0dc7a9
MD5 (fat) = 39e98200043e438f9070369989b2d964
MD5 (rat) = 77b1c3077041078fd1f371b1bb7dd6e4
```

## commit files

Add the files to the repo in _separate commits_:

```
$
for item in data1/bat data1/cat data1/dat data2/fat data2/rat; do
    git add ${item} && git commit ${item} -m "adding ${item}"
done
```

Now push all the commits to the remote (this will take a while):

```
$
git push origin master
```

Now you should see everything in the commit history on Github:

![Repo commit history](/img/history.png)

You should also see it locally in the git log:

```
$ 
git log --oneline
902b0d8 adding rat
b3376bd adding fat
e2427de adding dat
25682b5 addding cat
495235a addding bat
2506d38 adding bar.txt
2eb8d13 adding foo.txt
```

# demo surgery: procedure

## prepare tools

Use [git-forget-blob.sh](https://tinyurl.com/git-commit-ectomy)
to forget the blob. Start by downloading it:

```
$ 
wget https://tinyurl.com/git-forget-blob-mac-sh -O git-forget-blob.sh
chmod +x git-forget-blob.sh
```

This script will detect if you are on a Mac,
and if so, will use the GNU `gxargs` instead of 
the BSD `xargs`. This requires GNU tools to be 
installed via Homebrew:

```
$
brew install gnu-xargs
```

When installing `gnu-xargs`, you can also add the `--with-default-names` flag
to `brew` to overwrite the default BSD version of xargs (which is _not_ compatible
with the GNU version of xargs).

```
$
brew install gnu-xargs --with-default-names
```

To use the `git-forget-blob.sh` script:

```
$
./git-forget-blob.sh <relative-path-to-file>
```

(See below for more detail.)

## the command that doesn't work: git rm

Start by checking the size of the repo:

```
$ 
du -hs .git
 50M	.git
```

Now remove `dat` using `git rm`:

```
$ 
git rm dat
git commit dat -m 'Removing dat'
git push origin master
```

This, of course, does not change the size of the repo.
If we clone a fresh copy from Github, the size of the 
repo is still the same:

```
$
du -hs .git
 50M    .git
```

Why? Because git is cursed with perfect memory, and will not
forget a large file that's been added to the repo.

## the command that does work: git forget blob

To force git to forget a large file that's been added to the repo,
use the `git-forget-blob.sh` script to permanently remove it.
Here, we remove the `dat` file from the repo history by modifying
all commits that involve the `dat` file, and rewriting those commits
(and, by consequence, all commits that happened after that commit).

Here is how to permanently remove `dat` from the repo history and rewrite
all commits (we specify `data1/bat` and not `bat`):

```
$ 
./git-forget-blob.sh data1/bat
Enumerating objects: 26, done.
Counting objects: 100% (26/26), done.
Delta compression using up to 4 threads.
Compressing objects: 100% (20/20), done.
Writing objects: 100% (26/26), done.
Total 26 (delta 1), reused 26 (delta 1)
Rewrite 495235a86e70be03ee0749733645615a093b547a (3/7) (0 seconds passed, remaining 0 predicted)    rm 'data1/bat'
Rewrite 25682b53e6c8c88328346fc2e245b5946adec6cb (4/7) (0 seconds passed, remaining 0 predicted)    rm 'data1/bat'
Rewrite e2427def6f9de095928aaecfd9fef892880e6ce8 (5/7) (0 seconds passed, remaining 0 predicted)    rm 'data1/bat'
Rewrite b3376bdc847e26bdb323408afa06112dd4c2b36d (6/7) (0 seconds passed, remaining 0 predicted)    rm 'data1/bat'
Rewrite 902b0d8e46ec8d1487ae3db3b2989dfade5dacbe (7/7) (1 seconds passed, remaining 0 predicted)    rm 'data1/bat'

Ref 'refs/heads/master' was rewritten
Enumerating objects: 23, done.
Counting objects: 100% (23/23), done.
Delta compression using up to 4 threads.
Compressing objects: 100% (18/18), done.
Writing objects: 100% (23/23), done.
Total 23 (delta 1), reused 12 (delta 0)
```

Verify it worked by finding size of `.git` directory

```
$ 
du -hs .git
 40M	.git
```

Success!

Note that if you mistakenly specify the name of the file only,
without the relative path to the file, git will be looking for
the file at the top level of the repository, and the file will
not be found:

```
$ 
./git-forget-blob.sh bat
Enumerating objects: 26, done.
Counting objects: 100% (26/26), done.
Delta compression using up to 4 threads.
Compressing objects: 100% (21/21), done.
Writing objects: 100% (26/26), done.
Total 26 (delta 1), reused 0 (delta 0)
bat not found in repo history
```

## how it worked

If we check the git log we can see what happened - all commits involving 
`bat` were rewritten. It's important to note that when `git` computes the 
hash of each commit, it includes the hash of the prior commit - meaning,
if one commit in a repository's history changes, every commit in a repository's
history changes.

Thus, _we will rewrite every single commit since the very first commit that introduced
the file we removed_.

Compare the old and new logs:

```
# NEW LOG                    # OLD LOG                                          
                                                                                
$ git log --oneline          $ git log --oneline                                
5bc57f6 adding rat           902b0d8 adding rat 
3153621 adding fat           b3376bd adding fat                                 
c456173 adding dat           e2427de adding dat                                 
078a5be addding cat          25682b5 addding cat                                
3cd75ce addding bat          495235a addding bat                                
2506d38 adding bar.txt       2506d38 adding bar.txt                             
2eb8d13 adding foo.txt       2eb8d13 adding foo.txt                             
```

Note that the first two commits, which did not involve
the `bat` file, remain identical, but every commit
after `495235a` (which first introduced bat) is changed.

Each commit hash is computed using the prior commit hash,
so once commit `495235a` changes, it cascades through the
entire history by changing all subsequent commit hashes.

## stitch the patient back up

Of course, for surgeons, as for airline pilots,
if the last step is screwed up, nothing else counts.

We asked git to forget a file, which it did, 
but that required modifying git's entire commit 
history. At this point we have two parallel
`master` branches - the old history and the new
history.

If we simply `git push` our branch to the Github remote,
we will have a huge headache: both histories will end up
on Github, our git history will contain duplicates of every
commit, and the old and new history will show up side by 
side.

To do this _correctly_, we need to use the force
when we push, which tells the Github remote to rewrite
whatever commit history it currently has with the 
commit history that we are pushing.

```
$ 
git push origin master --force
```

This will ensure that Github does not keep duplicate
copies of all commits.

Here is a screenshot of the repo on github before 
we ran `git-forget-blob`:

![Commit history before git-forget-blob](/img/before.png)

And a screenshot of the repo after:

![Commit history after git-forget-blob](/img/after.png)

# tips for surgery

**Size up your patient before you start.**

Use the one-liner in the ["Consult with your Doctor" section](#consult-with-your-doctor)
section to size up your patient before you start
(modify the tail line to change the number of files).

**Get your patient some insurance.** 

Back up any files you want to remove but still want to keep.

**Make sure you specify relative paths to file names.** 

The `git-forget-blob.sh` script requires you to specify the
path to the file you want to remove, _relative to the top
level directory of the repository_.

Like this:

```
# CORRECT
./git-forget-blob.sh data/my_project/phase-1/proprietary/super_huge.file
```

Not like this:

```
# INCORRECT
./git-forget-blob.sh super_huge.file
```

The long one-liner in the ["Consult with your Doctor" section](#consult-with-your-doctor)
will list the largest files in the repository, with the relative
path to that file (relative to the root of the repository).

If you pass it a filename without a path to the file,
the script will most likely complain that the file could
not be found. But it may attempt to remove the file and 
rewrite history _anyway_ without removing any files.

If you are running `git-forget-blob.sh` and the size of the 
`.git` folder is not going down, it may be because you are
specifying an incorrect path to the files you are trying to 
remove.

