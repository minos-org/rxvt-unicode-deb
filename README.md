# rxvt-unicode-deb

minos' patch on Ubuntu's own rxvt-unicode package.

urxvt 9.31 anchors the rows to the bottom when a terminal grows, even with no scrollback to pull into view,
so a terminal that a tiling window manager resizes right after it opens shows its first prompt in the middle
of the window. `resize-keeps-top.patch` brings back 9.30's behaviour for that case only. Ubuntu 22.04 ships
9.30 and needs nothing.

## Branches

One branch per Ubuntu release that needs the patch: `noble` (24.04) and `resolute` (26.04).

Each one is that release's `ubuntu/<series>-devel` branch from Ubuntu's git of the package
(https://git.launchpad.net/ubuntu/+source/rxvt-unicode), with its full history, plus one commit that adds
`debian/patches/resize-keeps-top.patch` and its line at the start of `debian/patches/series`. Nothing else
changes, not even `debian/changelog`.

The line goes first because Ubuntu adds its new patches at the end of `series`: a new Ubuntu patch then
never touches the lines around ours, and the recipe's merge of this branch onto Ubuntu's stays clean. The
whole series applies in that order with no fuzz, which is how `dpkg-source` applies it.

## How it is built

One Launchpad recipe per release, building into the minos PPA (`ppa:minos-archive/main`), starts from
Ubuntu's branch and merges this one:

    # git-build-recipe format 0.4 deb-version {debversion}+minos{time}
    lp:ubuntu/+source/rxvt-unicode ubuntu/noble-devel
    merge minos lp:~minos-archive/minos/+git/rxvt-unicode-deb noble

The version is Ubuntu's followed by `+minos` and the time of the build: it stays just above Ubuntu's, every
build gets a new one, and any newer Ubuntu upload sorts above it. A version taken from the branches alone
repeats when the same commits are built twice, and the PPA refuses the second upload, whose changelog entry
carries another date. The recipes build
daily when a branch they read has changed, so a new Ubuntu upload is rebuilt with the patch; if that build
fails, Ubuntu's newer package wins over the old minos one.

## When Ubuntu uploads a new version

Rebase the one commit onto Ubuntu's branch and push it:

    git remote add ubuntu https://git.launchpad.net/ubuntu/+source/rxvt-unicode
    git fetch ubuntu
    git checkout noble
    git rebase ubuntu/ubuntu/noble-devel
    git push --force-with-lease origin noble

Our line stays the first one of `debian/patches/series`. Before pushing, check that the whole series still
applies in order with `patch -p1 -F0`, ours first: a new Ubuntu patch may touch the same lines of
`src/screen.C`.

A new Ubuntu release that ships a version with the regression gets its own branch, made the same way, and
its own recipe.
