# GitHub Pages Example

<div class="intro">

An example project and tutorial for creating and deploying *GitHub Pages*
projects using custom build tools and *Travis CI*.

</div>

<div class="author">
  Published by <a href="https://github.com/simbo" rel="author">Simon Lepel</a>
  in May 2018
</div>


---

- [About this project](#about-this-project)
  - [Types of GitHub Pages](#types-of-github-pages)
- [Assumed preconditions](#assumed-preconditions)
- [Deployment strategy](#deployment-strategy)
  - [Deploying using Travis CI](#deploying-using-travis-ci)
  - [Deploying manually](#deploying-manually)
- [Good to know](#good-to-know)
- [Feedback](#feedback)


## TL;DR

  - push your pages project sources to `master` branch while generated files are
    added to `.gitignore`

  - enable Travis CI using
    [pages deployment](https://docs.travis-ci.com/user/deployment/pages/) to
    build from `master` branch and push only generated files to `gh-pages`
    branch (see final [`.travis.yml`](#deployment-config))

  - [GitHub personal access token](https://github.com/settings/tokens) with
    access scope `public_repo` is required for pushing from Travis CI


## About this project

This example project demonstrates a clean and convenient strategy to create and
deploy *GitHub Pages* projects.

It focuses on setups, where pages are generated by a custom build tool (like
webpack, gulp or whatever) and not automatically generated by GitHub using
Jekyll or markdown. Although those type of projects could also benefit from
shown workflows.

If you are not familiar with *GitHub Pages*' core concepts, you may want to head
over to the [official documentation](https://pages.github.com/) first.


### Types of GitHub Pages

There are two types of GitHub pages: *project pages* and *user or organization
pages*.

Both offer the same features, while there are some important differences:


  - *Project Pages* (i.e. `your-username.github.io/your-project`)  
    …for creating pages for your project.  
    can be published from:
      - `gh-pages` branch
      - `master` branch
      - `master` branch `docs/` folder


  - *User/Organization Pages* (i.e. `your-username.github.io`)  
    …for creating your personal user pages or organization pages.  
    can *only* be published from `master` branch


<div class="alert--info">

This document focuses on *project pages* that are published from a `gh-pages`
branch, as this is *imho* the most usual and convenient usecase.

</div>

I suppose that, after reading this tutorial, you will be able to change
configuration settings according to your needs to publish your *user pages* or
whatever you want to do.


## Assumed preconditions

You have just created your new website project and your project root looks
somehow like this:

``` text
./
├─ dist/          # generated pages
├─ src/           # sources and templates
├─ .gitignore     # `dist/` folder is ignored
├─ package.json   # project metadata and tasks
└─ README.md      # project information
```

The `dist/` folder should be added to `.gitignore` as we do not want to commit
generated files to our source branches.

Everything else should be committed to your git repository's `master` branch and
pushed to GitHub.


## Deployment strategy

The concept is simple:

<div class="alert--info">

While your generated `dist/` folder is ignored for the `master` branch, you
have another branch `gh-pages` that only contains the generated contents of
`dist/`, which will be automatically published at your GitHub pages URL.

</div>

This is easy to achieve using a CI/CD service like *Travis CI* but can also be
done manually.


### Deploying using Travis CI


#### Preparations


##### Enable Travis CI support

If not done already, register at [Travis CI](https://travis-ci.org/) using your
GitHub account and install the
[travis client](https://github.com/travis-ci/travis.rb) on your machine.

Enable Travis support for your project by running `travis init` in your project
root. When asked for main language, choose what fits your needs (i.e. `node`).

Afterwards, there should be a fresh generated `.travis.yml` in your project
root.


##### GitHub personal access token

Go to GitHub and get a
[personal access token](https://github.com/settings/tokens), so Travis will be
authorized to push changes back to GitHub. Give it a useful description like
`my-project travis deploy` and select `public_repo` as access scope.

After generating the token, encrypt it and add it to your Travis config:

``` bash
travis encrypt GITHUB_TOKEN=your-personal-access-token --add
```


##### Deployment config

Travis offers the
[GitHub Pages deployment provider](https://docs.travis-ci.com/user/deployment/pages/),
which fullfills all our needs automagically.

With deployment options, your final `.travis.yml` should look like this:

``` yaml
language: node_js
node_js:
- '8.11.*'

env:
  global:
    secure: PDS4pIbrZhQaB…  # encrypted github token

script:
- npm run build             # build script

deploy:
  provider: pages
  github-token: $GITHUB_TOKEN
  on:
    branch: master          # trigger deploys only from master branch
  local-dir: dist           # path to generated pages
  target-branch: gh-pages   # target branch for deploy
  skip-cleanup: true        # keep generated files from build script
  keep-history: true        # keep git history in gh-pages branch
```

If not already existing, the `gh-pages` branch will be automatically created on
the first deploy.

Afterwards, you can enable *GitHub Pages* support in your repository settings
on GitHub. Make sure to set the build source to `gh-pages` branch.

*You're done!*


#### Triggering deploys

Every pushed commit to `master` should now automatically trigger a build and
push updates to `gh-pages`, which will be published at the respective URL.


### Deploying manually

<div class="alert--warning">

I do not recommend to use manual deployment as a common strategy. Although it
seems simple, it's prone to human error. Nevertheless, it's good to know…

</div>

We're assuming the same preconditions as for deploying with Travis:
`dist/` folder is ignored while everything else is pushed to `master` with no
current changes in the working tree.


#### Preparations for manual deploy workflow


##### Create `gh-pages` branch

You can skip branch creation if you already have a branch `gh-pages` on your
GitHub remote.

``` bash
# create a new orphan branch `gh-pages` and clear its working tree
git checkout --orphan gh-pages
git rm -rf .

# create an `index.html` and commit it
echo "Hello World" > index.html
git add index.html
git commit -m "initial content"

# push `gh-pages` branch to github
git push origin gh-pages

# switch back to `master` branch and delete `gh-pages` locally
git checkout master
git branch -d gh-pages
```

Afterwards, you can enable *GitHub Pages* support in your repository settings
on GitHub. Make sure to set the build source to `gh-pages` branch.


##### Clone `gh-pages` to `dist/`

Now clone the `gh-pages` branch to the `dist/` folder inside of your repo.
This way your build script will generate files directly into `gh-pages`.

``` bash
# remove generated dist folder if existing
rm -rf dist
# clone only `gh-pages` to `dist/`
git clone git@github.com:USER/PROJECT.git --branch gh-pages --single-branch dist
```


#### Manual deployment steps

``` bash
# run build script
npm run build
# go to `dist/` to commit and push generated changes
cd dist
git add -r .
git commit -m "update content"
git push origin gh-pages
```

When using manual deployment like this, make sure that your build script
doesn't touch the repository inside of `dist/`. You may want to build a little
script for the deployment steps (or better
[use Travis CI for deployment](#deploying-using-travis-ci) directly).


## Good to know

  - `index.html` vs. `README.md`

    There will be a conflict if you have both of these in your `gh-pages`
    root, as GitHub seems indecisive which one of them to use per default…


## Feedback

Feel free to use
[project issues](https://github.com/simbo/gh-pages-example/issues) for
discussion and comments.