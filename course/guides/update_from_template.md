# Updating a Course to the Newest Template

In this guide, I'll go through the steps of pulling the changes made to our [template](https://github.com/gamma-opt/jb_course_template) into your new course repo.

We will do so by merging, i.e. by treating the course repo as if it is a branch diverged from the template, and joining them together.

## Preparation

Before starting, make sure your repo doesn't contain uncommitted work.
If it does, you can save them with
```bash
git stash
```
and then once you are done, restore them with
```bash
git stash pop
```

## Merging

Assuming the template is already added to the remotes of your course repository, fetch the latest changes from the template and initiate the rebase process.

````{tip}
You can check the remotes of your repository with
```bash
git remote -v
```
If the template is not there, add it with
```bash
git remote add template <url>
```
````

```bash
git fetch template
git merge template/main
```

`git` will automatically try to combine all commits made since the latest common ancestor.
For example, in `_config.yml`, if you renamed the course title from "Course template" to "Underwater Basket Weaving", `git` should have no problem resolving that.

However, you may end up with some conflicts, for example if the course added a specific LaTeX macro in `_config.yml` and the template had a different change to the same file.
However, due to the template being very bare-bones, these conflicts should be minimal and easy to resolve.

While `git` will instruct you on what to do, in most cases conflicts can be resolved by editing those files, and then marking them as ready with
```bash
git add {changed files}
```
then continuing the merge with
```bash
git commit
```

Be careful with the `add` command, don't accidentally introduce new files.
For example, if the folder contains untracked files, `git stash` won't get rid of them, so if you were to do `git add .`, you will end up commiting them as well.

If something goes wrong, you can often abort the process with
```bash
git merge --abort
```
and go back to the start.
But this may not work 100% of the time, so do be careful.

At the end, you'll have a merge commit.
Give it an informative message, like "Merge: Updated to latest template".

## Done

Push the changes with
```bash
git push origin main
```
and you should be good to go.
