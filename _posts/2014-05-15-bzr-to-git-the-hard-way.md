---
title: "Bazaar to git - the hard way"
author: ar
layout: post
permalink: /archives/632
sharing_disabled:
  - 1
sharing: false
tags:
  - bazaar
  - git
  - scm
  - hacks
  - tegnament
  - xelera
---
While working on the website for a new WordPress product, i was doing some
cleanup of obsolete websites, making sure we have full commit history of past
changes before taking obsolete content offline.

I noticed that the longest-running website (the first iteration was in 1999 or
so) had its history split across three slightly-differently-named *git*
repositories stored on our master *git* server - although on closer inspection,
two of them were actually empty (no commits, no nothing), while the third one
was actually a repository with content that i had been working on around three
years as a redesign experiment which never went past mockup stage.

At least one of the two empty repositories turned out to be a failed attempt
to migrate to *git* an older Bazaar repository, at the time of our
mass-migration to git via [Tailor](http://progetti.arstecnica.it/tailor) a few
years back.

I remember that a handful of Bazaar repositories (some of which had in turn
been migrated over from the glorious Arch SCM) could not be migrated to git,
nor through our mass-migration procedure nor with ad-hoc love, but at that
time we checked that nothing really important had been left behind in
Bazaar, and just moved on with life.

It turns out that although no important *history* was left behind, yet we had
left behind this whole website, which since then has had very few changes,
recorded in a git repo whose history starts where the older Bazaar repo was at
the time of the failed migration.

As i was going to archive all this stuff, i wanted to bring all the past history
into a single git repo, with the more recent redesign work stored as a feature
branch - and here is where trouble started: `bzr fast-export` was failing
with an articulated Python stack trace, and git clone bzr--git://old/repo/ was
failing likewise. When checking the Bazaar repo with `bzr check -v`, it turned
out that a *ghost revision* within the old repo, ancestor of revision 1 (?!),
was the culprit. As i haven't been using Bazaar in ages, i didn't know what to
do about this, although a quick search around the error messages i was getting
seemed to point only to outstanding (some since 2009...) bug reports about
ghost revisions blocking Bazaar exports.

Eventually though, since the "damaged" Bazaar repo only stored nine revisions,
I thought it would be easier to just check out each of them sequentially,
committing into a git repository the status of the working directory at each
(Bazaar) checkout. Still, I wanted to preserve (just for archival purposes,
again - none of this had any real practical relevance) the original timestamps,
rather than making all the commits appear to have happened tonight.

git's brilliance shines through in every detail: it took just a quick
[stackoverflow search](https://stackoverflow.com/a/3896112/550077) to learn
just how to backdate git commits... and a few minutes later, my git repo was
populated with the full past history:

    mkdir /path/to/git/repo
    cd /path/to/bzr/repo
    bzr revert -r1
    rsync -ravz --delete /path/to/bazaar/repo/ /path/to/git/repo
    cd /path/to/git/repo
    git add .
    GIT_AUTHOR_DATE="<original commit timestamp>" GIT_COMMITTER_DATE="<original commit timestamp>" git commit -m <old bzr commit message>

...A so on for each revision. The new repository is now pushed to our master
git server and the old Bazaar repo is nuked - good riddance!
