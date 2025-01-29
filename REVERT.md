## Revert Example
This example outlines the steps needed to revert a specific tag, in most cases this would be the most recent tag containing the last published changes.

Most often a revert is not going to be the best way to undo a change - you may be better off moving forward by making changes to the `production` branch to "undo" the previously published changes.

It is important that you know exactly which tag you want to revert, so double check that before proceeding.

Using the terminal, navigate to the directory of your choosing and clone the repository to your computer (replace `[YOUR_PROJECT]` with the correct path):
```shell
git clone git@github.com:searchspring-implementations/[YOUR_PROJECT].git
```

If you already have a checkout of the repository ensure that the `production` branch is up to date before branching.

```shell
git checkout production
git pull
```

Verify the tag version by taking a look at the project tags. The following command will list the tags in order, from newest to oldest.

```shell
git tag -l --sort=-version:refname
```

Create a new branch (replace `my-reversion-branch` with branch name) for the reversion branch:

```shell
git checkout -b my-reversion-branch
```

Revert the previous tag (replace `the-tag-to-rollback` with the tag that you want to roll back _from_; typically the newest one):
```shell
git revert --no-commit -m 1 the-tag-to-rollback
```

Verify that that changes have rolled back as expected by viewing the file diffs. If there are any merge conflicts you will need to resolve them now and then stage those changes.

Commit changes:
```shell
git add -A
git commit -m 'a nice commit message talking about the reversion'
```

Push the new branch to Github (replace `my-reversion-branch` with branch name):
```shell
git push -u origin my-reversion-branch
```

Check the repository action status and verify that it completes successfully. https://github.com/searchspring-implementations

Verify that the changes fixed the problem by using a branch preview. Navigate to the website where Snap is integrated:  
`http://www.yoursite.com/collection/things?branch=my-reversion-branch`

Once satisfied with the changes, the branch can be merged into `production`. This must be done via a Pull Request in Github. This extra step clearly identifies the incomming changes about to be merged so that any last minute issues can be resolved. Any merge conflicts are immediately identified and can even be resolved within Github. Pull requests also provide a method for requesting a peer review of your work before merging it in.

After the branch is merged into `production`, another Github action will kickoff, and if it succeeds, the changes will be published to the CDN and be live on the site.

Congratulations, you have reverted a tag for your Snap project.
