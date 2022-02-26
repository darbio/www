---
title: 'Mercurial Branching'
date: 2012-07-11 00:08
draft: false
tags: ['Mercurial', 'Source control']
summary: 'Notes about migration to Mercurial'
---

We are currently going down the path of switching from our huge monolithic SVN repository, to a number of product based Mercurial repositories. Historically, I have used FogBugz and Kiln, but this time we have decided to give the JIRA stack a whirl - mainly for GrassHopper, as one of the project managers here wanted to give it a go.

In the old SVN repo, instead of branching and tagging, we copied files to "Release folders", and ended up with a bunch of duplicated code, which was pretty hard to manage. One problem was making a bug fix in a release directory, then having to either manually implement that into the development directory, or do an onerous merge of the 2 directories.

To cut a long story short... We were using SVN like a file store, not a source control system.

For my personal projects in KilnHg I have always had 2 repositories per project - one for "Release" and one for "Development". Why? Because that is how Fog Creek's guides showed me. It works, but I wanted to take advantage of Mercurial's named branches for our work projects. It produces some nice graphics, and (whilst I prefer the terminal to work with hg) works better with the TortoiseHg GUI - which is what my developers are used to, albeit TortoiseSVN.

So, I armed myself with some questions:

- How to do this?
- What is the best way?
- Am I mad?

And went to the holy grail of Google to have a look.

After some to-ing and fro-ing, trying and failing, I came up with the following "demo" scenario. What follows is simple, and possibly does not deserve a blog post - but it can provide some point of reference for me (and my team!)...

# Initial repository

    - Create repo in source control system
    - Clone repo to my machine

`hg clone repoURL`

    - Add my code to the repo

`hg add`

    - Commit code to the repo

`hg commit -m "Initial commit"`

    - Push to the remote repo

`hg push`

# Make the release branch

    - Make the release branch

`hg branch "Release-1.0"`

    - Commit the change to the repo

`hg commit -m "Added branch Release-1.0"`

    - Push to the remote repo

`hg push --new-branch`

# Bug fix on the Release-1.0 branch

    - Switch working directory to the branch you want to edit in your repo

`hg update "Release-1.0"`

    - Add your bugfix to the repository
    - Add your changes to the repo

`hg add`

    - Commit your changes

`hg commit -m "Added BUGFIX1 to Release-1.0"`

    - Push

`hg push`

# Feature added on the default branch

    - Switch working directory to the default branch

`hg update "default"`

    - Add your feature to the repo
    - Add your changes to the repo

`hg add`

    - Commit your changes

`hg commit -m "Added FEATURE1 to default"`

    - Push

`hg push`

# Merge the bugfixes on Release-1.0 into default

    - Switch working directory to the default branch

`hg update "default"`

    - Merge the Release-1.0 branch into the default branch

`hg merge "Release-1.0"`

    - Commit your changes

`hg commit -m "Merged Release-1.0 into default"`

    - Push

`hg push`

Which results in the following graphlog (from BitBucket):

![](http://i.imgur.com/wyhXO.png)
