---
title: "Updating Debian PEAR packages"
author: ar
layout: post
permalink: /archives/631
sharing_disabled:
  - 1
sharing: false
tags:
  - debian
  - howto
  - tegnament
---
After preparing my first Debian packages this summer (some dependencies for
Composer, which in turn is a dependency for wp-cli, which is the software i'm
really after), i have been postponing updating them with new (minor so far)
upstream releases mainly as i kinda dreaded having to work with PEAR's tools to
figure out how to best import new upstream releases.

It turns out it's extremely simple and quick - and i'm writing it up here
mainly as a reminder to myself. I am updating these packages now to sync them
with current upstream releases, but also to remove the Build-Depends on the
pear-symfony2-channel which never made it to the official Debian repositories
and [is being superseded by pear-channels like all other legacy Debian packages
for individual PEAR
channels](https://lists.alioth.debian.org/pipermail/pkg-php-pear/2013-October/001782.html).

What i am doing, basically, is:

    git checkout upstream-sid
    pear download <Package>

(Package-x.y.N.tgz is downloaded)

    git mv <Package-x.y.M> <Package-x.y.N>
    tar xvzf Package-x.y.N

(this overwrites files as needed; now we check what the new
code is like, if there is anything needing attention regarding
license, etc.)

    git diff
    git add <new files if ok and changed files if ok>

    git commit -m 'Import of x.y.N'
    git tag -u <signing_key> -s upstream/x.y.N

If using a pristine-tar branch (as for php-symfony-console), the process is
slightly different (and possibly, this should be the way to do things in any
case, but i have only figured this out after having tagged all the new
releases, so i will check this on the next update): once the
`Package-x.y.N.tgz` file is downloaded, we can import its delta against the
previous version via `git-import-orig`:

    mv Package-x.y.N.tgz ..
    git-import-orig -u x.y.N ../Package-x.y.N.tgz

(again, i need to check this on next update, but i believe that all it takes is
actually a single `git-import-orig --uscan`, which should take care of pretty
much everything up to now).

Now the Debian fun starts:

    git checkout debian-sid
    git merge upstream-sid

(the upstream code is merged)

    gbp dch

(a new changelog entry is generated; this needs to be checked and updated as
needed; any updated copyright information or other changes to Debian packages
should be done at this stage and ideally committed individually - otherwise
we just commit the updated changelog)

    git add debian/changelog

and finally

    git-buildpackage

That's it - if everything is working as expected, assuming nothing has been
pushed anywhere yet, we can make a final update to changelog by switching
release from UNRELEASED to whatever appropriate and amend the previous commit
on the debian-sid branch.

Once everything is finalised, we can tag the Debian release:

    git tag debian/x.y.N-Z

and push to git.debian.org.
