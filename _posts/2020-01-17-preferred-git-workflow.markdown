---
layout: post
title:  "My preferred git workflow for small teams"
date:   2020-01-23 15:00:00 -0400
categories: git collaboration teams opinion
---

git has many useful features for tracking collaborative changes. Two common patterns for using git collaboratively are 

* One central repository where all changes end up, it is the source of "truth"
* Multiple repositories that pull in changes from each other, there isn't one source of "truth"

My experience is primarily with the first case, and I'd like to outline a few guidelines for managing changes for this central repository scheme. Many teams miss an opportunity to maintain a pattern of organization and labelling that can save you headaches down the line. It also limits the amount of complicated merges you have to make, and provides additional tracability with whichever issue tracker you use.

# Fork and modify your own copy of the repository

In this article, I'll explain the workflow we used at one of my internships. I'll assume you have a basic understanding of what the git model looks like (i.e. what branches are, how to check out branches, fetch, push, and pull changes from the command line). I'll refer to the branch where we would like to push changes as `develop`. The repository name will be `TeamName/OurProject`.

The first antipattern that sticks out to me is when individual team members clone and modify the organization's (or individual owner's) repo. In GitHub and GitLab, you can fork the project to your own account, so that way you don't clutter the repo that should be the source of "truth". Once you do that, you should have your own repository called `<yourName>/OurProject`, which you should `git clone` onto your local machine.

# Using Remotes to stay synchronized

When you want to interact with multiple repositories, you use [git remotes](https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes). You can add a remote to your local repository with `git remote add`:
```
git remote add <alias> <url>
```

When you first clone a project from GitHub/GitLab, you're actually adding the remote repository with the alias `origin`, which you can see when you do something like `git push origin <branch>`. The alias we'll use for the central repository going forward is `upstream`. You can add the central repository:

```
git remote add upstream <git url for TeamName/OurProject>
```

Now you can interact with the upstream repository from the command line.



# Check out the most recent version before you start working

When you're ready to start working on a new feature, bug fix, etc., in most cases you'll want to start with the most up-to-date version of the repository. To do this, you move to the `develop` branch and then you can pull in changes from the `upstream` repository:

```
git checkout develop
git pull upstream develop
```

Now your local develop branch is up-to-date. From here, you can check out a new branch to get started on the work you have to do.

```
git checkout -b <newBranchName>
```

## Branch naming conventions

If your team uses an issue tracker (e.g. GitHub or GitLab issues, JIRA, Trello) to assign and monitor all of the work you do, I recommend naming the new branch after the ID of the issue itself. When you go to merge these changes back, the request will include the branch name, which will provide context for anyone who ends up looking at the change.

As an example, suppose you were assigned to issue #349 on your team's Trello board. You should work on a branch named something like `issue-349`. When a reviewer goes to look at your changes, they can instantly see that this merge request should be addressing issue #349, which they can then review in the issue tracker for context. In cases where branch names collide (for example, you want to have two branches where you work on this issue), one option is to work on branches named `issue-#-1`, `issue-#-2`, etc.

## Getting your changes to the central repository

As you make commits on your feature branch, you should push them to an identically named branch on your `origin` repository. To follow the above example, you would push your changes like so:

```
git push origin issue-349
```

When you're finished working on the issue, you can make a pull/merge request on Github or GitLab to merge `issue-349` on `YourName/OurProject` to `develop` on `TeamName/OurProject`, which can be merged after your team's code review, CI, etc. procedures.

## Getting back up to date with rebase

If you're still working on your new branch and changes have been made to `upstream/develop` since you started working, you can bring in the changes to your current branch with `git rebase`. The most straightforward way to do this is to

1. Make sure your version of develop is up-to-date. If not, do
```
git checkout develop
git pull upstream develop
git push origin develop
git checkout <my-existing-branch>
```
2. Now you can bring in the changes from develop and then place your feature branch changes on top of them:
```
git rebase develop
```
3. If there are merge conflicts, you must resolve them. I assume that you can handle this, but if you need to cancel the rebase operation, you can do `git merge --abort`.
4. Your branch is now up-to-date and only includes forward changes from `upstream/develop`, so you don't have to worry about existing conflicts. If you've previously pushed changes before rebasing, keep in mind that you will either have to create a new branch, or (very carefully) force push.

Note `git rebase` is super powerful, and it can be helpful to look at the [official documentation](https://git-scm.com/docs/git-rebase) to learn more about what you can do with it.
