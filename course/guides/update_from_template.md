# Updating a Course to the Newest Template

In this guide, I'll go through the steps of pulling the changes made to our [template](https://github.com/gamma-opt/jb_course_template) into your new course repo.

We will do so by rebasing, i.e. going back in _git history_ to when the course was cloned/forked, seamlessly applying the changes to the template, and putting back the course-specific commits.

This is probably not the only way to sync the changes made to the template repo, but I think it may be the cleanest.

## Preparation

Before starting, make sure your repo doesn't contain uncommitted work.
If it does, you can save them with
```bash
git stash
```
and then restore them once you are done with
```bash
git stash pop
```

## Rebasing

Assuming the template is already added to the remotes of your course repository, fetch the latest changes from the template and initiate the rebase process.

```bash
git fetch template
git rebase template/main
```

`git` will automatically apply all the course-specific commits on top of the latest template.
For example, in `_config.yml`, if you renamed the course title from "Course template" to "Underwater Basket Weaving", `git` should have no problem resolving that.

However, you may end up with some conflicts, for example if the course added a specific LaTeX macro in `_config.yml` and the template had a different change to the same file.
However, due to the template being very bare-bones, these conflicts should be minimal and easy to resolve.

Resolve the conflicts by editing those files, and then mark them as ready with
```bash
git add {changed files}
```
then continue the rebase with
```bash
git rebase --continue
```

Be careful with the `add` command, don't accidentally introduce new files.
For example, if the folder contains untracked files, `git stash` won't get rid of them, so if you were to do `git add .`, you will end up commiting them as well.

If something goes wrong, you can abort the process with
```bash
git rebase --abort
```
and go back to the start.

You may need to repeat the above resolve-and-continue process multiple times, but at the end your local repository should be updated.

## Done

Push the changes with
```bash
git push origin main --force-with-lease
```
and you should be good to go. Here the force flag is needed as we _rewrote history_ in `git`'s eyes, but `-with-lease` part should prevent problems if someone else happened to push a change while you were busy with the updating process.
