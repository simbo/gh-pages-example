gh-pages-example
================

  > Example project for creating and maintaining github pages

---

<!-- TOC -->

- [Goals of this project](#goals-of-this-project)
- [Setup](#setup)
  - [Preconditions](#preconditions)
  - [Initialize git](#initialize-git)

<!-- /TOC -->

---


## Goals of this project

Find and document a clean and convenient strategy to create and maintain a
github pages project where sources and generated contents are separate
branches.

Publishing updated contents should be handled in a way that is easy to
use and understand - preferably by just pushing updated sources to master and
letting travis build and deploy contents.


## Setup


### Preconditions

You have just created your new website project and your project root looks
something like this:

```
./
├─ dist/          # generated contents
├─ src/           # sources and templates
├─ .gitignore     # dist folder is ignored
├─ package.json   # project metadata and tasks
└─ README.md      # project information
```


### Initialize git

If not done already, initialize git, add github as remote and commit everything
except the `dist/` folder, which is added to `.gitignore`.

``` sh
git init
git remote add origin git@github.com:simbo/gh-pages-example.git
git add src .gitignore package.json README.md
git commit -m "initial commit"
```
