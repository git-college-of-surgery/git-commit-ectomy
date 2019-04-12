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

git has the ability to create patches using
`git-diff` and the `--patch` flag, which is the
default behavior.

```
git-diff --patch
```

