# Using the Template to Make a New Course

In this guide, I'll go through the steps of making a new course using our customized [template](https://github.com/gamma-opt/jb_course_template). 
A separate guide on how to pull the changes from an updated template to your new course is [here](./update_from_template.md).

## Template contents

The template provides the basic `jupyter-book` skeleton along with some customization.
The biggest of these are 
- various settings like LaTeX macros and using the [QuantEcon theme](https://github.com/QuantEcon/quantecon-book-theme) (found in `course/_config.yml`),
- tweaks to the theme that changes the toolbar logo and removes the "Download PDF" button (found in `course/_static`), and
- GitHub workflows to compile the PDF and update it or deploy GitHub Pages whenever something is pushed to `course/` (found in `.github/`).

## Making a New Course

### 1. Copy the template and clone it into your machine.

You can either fork the template repository, or clone it and push to a new repository you created. The difference between these two is that forking will leave you with messages like this:
```{image} ../resources/x_commits_ahead.png
:name: x commits ahead
```

````{tip}
When updating, we will need the template to be among the remotes of the local repository. So it is a good idea to add it now, for example with
```bash
git remote add template https://github.com/gamma-opt/jb_course_template.git
```
````

### 2. (Optional) Prepare the programming environments. 

The template uses `jupyter-book`, so you'll at least need `python`. If more packages may be needed, it is a good idea to create a virtual environment. A `conda`/`mamba` environment is provided in `environment.yml`, but you can use some other tools as well, just make sure to install `jupyter-book` and `quantecon-book-theme` (this one is installed from GitHub).

A Julia environment may also be useful. The provided `Project.toml` comes only with `IJulia`, which is essential for making `jupyter-book` execute Julia code. You can (add other packages and) ready the environment with
```julia
using Pkg
Pkg.activate(".")
Pkg.instantiate()
```

### 3. Configure basic settings

Open up `course/_config.yml` and change the basic information such as course title,  author, and the GitHub repository.

Take a look at `.github/workflows`.
If you don't want to use them, you can just delete them now or disable them ([here](https://docs.github.com/en/actions/using-workflows/disabling-and-enabling-a-workflow) is how).
Even if you want to use them, it may be a good idea to disable them on GitHub, until a significant amount of the content is ready.

```{important}
If you'd like to use the PDF workflow and you are not Fabricio, you may want to change the git details at the bottom of the workflow.
```

```{warning}
If you change the latex target name in `_config.yml`, make sure to update the last step in `workflows/jupyterbook-pdf.yml` with the new name.
```

### 4. Prepare the course

You should now be ready to prepare the content of the course, which essentially involves creating files under `course/` and specifying them in `course/_toc.yml`. The rest of this instructions material should be helpful with that.