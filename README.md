### LaTeX + Git + Travis &rightarrow; release pdf

[![Build Status](https://travis-ci.org/PHPirates/travis-ci-latex-pdf.svg?branch=master)](https://travis-ci.org/PHPirates/travis-ci-latex-pdf)
[![Gitter](https://badges.gitter.im/travis-latexbuild/community.svg)](https://gitter.im/travis-latexbuild/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
![](https://github.com/PHPirates/travis-ci-latex-pdf/workflows/Github%20Action%20for%20LaTeX%20by%20xu-cheng/badge.svg)
![](https://github.com/PHPirates/travis-ci-latex-pdf/workflows/Compile%20a%20ConTeXt%20document/badge.svg)
![](https://github.com/PHPirates/travis-ci-latex-pdf/workflows/dxjoke%20Tectonic%20Docker/badge.svg)
![](https://github.com/PHPirates/travis-ci-latex-pdf/workflows/setup-tectonic%20action/badge.svg)
<!-- ![](https://github.com/PHPirates/travis-ci-latex-pdf/workflows/Compile%20LaTeX%20by%20vinay0410/badge.svg) -->

Write LaTeX, push to git, let Travis automatically build your file and release a pdf automatically to GitHub releases when the commit was tagged.

This repository contains an overview of the methods you could use to build your LaTeX on a remote server (continuous integration server).
You want that because then every time you push it will automatically check if you pushed valid LaTeX.

If you are looking for instructions to build LaTeX locally, look [here](https://github.com/Ruben-Sten/TeXiFy-IDEA#installation-instructions).
If you are looking for instructions to build LaTeX on GitLab CI, have a look at the [GitLab](#gitlab) section.
If you are looking for instructions to build ConTeXt, have a look at the [ConTeXt](#context) section.

## Quickstart

This repository contains a huge amount of information, and as you can see I have worked with a lot of different options.
These are my two favourites:

* For simple documents, I use [Tectonic via Docker](#gh-actions-docker).
* For documents which Tectonic can't handle (think external programs like biblatex, minted, pythontex, ...) I use a [cached basic TeX Live installation with texliveonfly](#gh-actions-texlive).

## Table of Contents

- [GitHub Actions](#github-actions)
- [Choose your build method](#choose-your-build-method)
  - [Tectonic](#tectonic)
  - [1. Docker image with Tectonic](#1-docker-image-with-tectonic)
  - [2. Miniconda with Tectonic](#2-miniconda-with-tectonic)
  - [3. Docker image with TeX Live](#3-docker-image-with-tex-live)
  - [4. TeX Live with pdflatex/lualatex/latexmk/xelatex/texliveonfly/etc](#4-tex-live-with-pdflatexlualatexlatexmkxelatextexliveonflyetc)
  - [5. TeX Live and pdflatex via tinytex with R.](#5-tex-live-and-pdflatex-via-tinytex-with-r)
- [Instructions for each build method](#instructions-for-each-build-method)
  - [Instructions for building with Docker and Tectonic](#tectonic-docker)
  - [Instructions for building with Miniconda and Tectonic](#tectonic)
    - [Separate instructions for adding biber to your Miniconda and Tectonic setup](#biber)
  - [Instructions for building with TeX Live from a Docker image](#texlive-docker)
    - [Using a docker image with texlive and texliveonfly](#texlive-docker-texliveonfly)
  - [Instructions for building with pdflatex/lualatex/latexmk/xelatex/texliveonfly/etc and TeX Live](#pdflatex)
  - [Instructions for building with TeX Live and pdflatex via tinytex with R](#tinytex)
  - [To automatically deploy pdfs to GitHub release](#deploy)
    - [First time setup](#first-time-setup)
    - [For every new project](#for-every-new-project)
  - [Tips](#tips)
  - [Troubleshooting](#troubleshooting)
  - [Notes](#notes)
- [Contributing](#contributing)
- [GitLab CI](#gitlab)
- [ConTeXt](#context)

# GitHub Actions

In 2019 GitHub introduced GitHub Actions, which is essentially an alternative to Travis.

- GitHub Actions is integrated into GitHub so you do not have to leave GitHub to see build logs
- GitHub Actions can also publish to GitHub releases, and upload artifacts
- GitHub Actions also has a caching mechanism
- GitHub Actions has a marketplace from which it is very easy to copy yaml configuration files

To use it:
* Go the Actions tab in your repository
* Click on Set up this workflow
* Search in the Marketplace for LaTeX
* Choose an action, see below for an overview.
* Copy the yaml shown to _under_ 'steps' in the file shown
* Commit

For more information, see https://help.github.com/en/actions/automating-your-workflow-with-github-actions

In the following sections we will give an overview and comparison of available GitHub Actions to build LaTeX.
Note however, that you can adapt any of the methods for using Travis CI to work with GitHub Actions, because the syntax for the yml configuration files is very similar.

## Docker image with full TeX Live

Currently we know the following Actions.

### Github Action for LaTeX by xu-cheng

Advantages:
* You can specify file to compile, working directory, compiler, compiler arguments and extra packages to install in the alpine image used
* Supports minted, biber, custom fonts (using local paths)

Disadvantages:
* It downloads a full TeX Live image on every build, which takes almost three minutes

Example in this repo: [xu-cheng-docker-full-texlive.yml](.github/workflows/xu-cheng-docker-full-texlive.yml)

Build time example files: 3-4 min.

### LaTeX compilation by dante-ev

This is a fork of Github Action for LaTeX by xu-cheng, of which the only changes are now also in the original Action by xu-cheng.

### Latex-multicompiler by Jatus93

Fork of 'LaTeX compilation by dante-ev' but reads the file to compile from a `.fileToCompile` file in your repository so this does not seem to have any advantages.

### Github Actions for build LaTeX and release pdf by MaineK00n

Uses https://github.com/Paperist/docker-alpine-texlive-ja which installs the TeX Live package `collection-langjapanese` by default and otherwise does not seem to have any advantages (or documentation at all).

### Uberblatt by ottojo

debian-based Docker image with a full texlive installed using the debian package.
Includes pygments and pandoc, for source see https://hub.docker.com/r/aergus/latex/dockerfile.
No known advantages currently.

## Docker image with Tectonic

The following Actions use Tectonic, which means they have the advantages and disadvantages of Tectonic as described [below](#tectonic)

### <a name="gh-actions-docker">Using the Docker image directly</a>

Essentially this just executes `docker run --mount src=$GITHUB_WORKSPACE/src,target=/usr/src/tex,type=bind dxjoke/tectonic-docker /bin/sh -c "tectonic main.tex"` to compile `main.tex` in `src`.

* Copy [.github/workflows/dxjoke-tectonic-docker.yml](.github/workflows/dxjoke-tectonic-docker.yml) to your repository, changing the source path and file to compile
* Commit and push

Build time example file: 30s

### setup-tectonic by WtfJoke

This Github Action downloads tectonic (and optionally biber) from the latest official releases.
It has a significant advantage over using the Docker image directly: it supports caching of installed LaTeX packages so it is significantly faster for documents with many dependencies.
For the documentation and more explanation (including one about the cache) see https://github.com/wtfjoke/setup-tectonic

Example in this repo: [.github/workflows/setup-tectonic-action.yml](.github/workflows/setup-tectonic-action.yml)

Build time example file: 20s

### Compile LaTeX by vinay0410

Uses a [fork](https://github.com/vinay0410/tectonic-docker) from https://github.com/WtfJoke/tectonic-docker with no significant changes.

Advantages:
* You can specify the path to the file to compile
* Supports biber

Disadvantages:
* Packages are not cached, downloading them all every time costs time
* No way to disable the automatic upload of pdfs to the master branch (and because of that it has been commented out in this repo)

Example in this repo: [vinay04010-docker-tectonic.yml](.github/disabled-workflows/vinay04010-docker-tectonic.yml)

Build time example files: 4-5 min.

## TeX Live

### <a name="gh-actions-texlive">Installing TeX Live directly</a>

You can do basically the same as [4-texlive/.travis.yml](4-texlive/.travis.yml) for GitHub Actions.
The workflow [.github/workflows/texlive.yml](.github/workflows/texlive.yml) contains a full example of installing a basic TeX Live installation and using texliveonfly and caching to install the rest.
It also contains the steps needed to automatically release a pdf to GitHub if you tag a commit, and also shows that it is easy to extend with other programs like pythontex.
The complete time the GitHub Actions needs for an empty LaTeX file is only 20 seconds.

The steps to use this are then:
* Copy the texlive folder in [4-texlive](4-texlive) to your project
* Copy the workflow file [.github/workflows/texlive.yml](.github/workflows/texlive.yml), read it through and change all the paths

### paper-maker by andycasey using Ubuntu texlive packages

This actions installs a default set of texlive packages from Ubuntu, at the moment `texlive-publishers texlive-latex-recommended texlive-latex-extra texlive-fonts-recommended texlive-fonts-extra`.

Then it compiles using pdflatex and bibtex, using `-shell-escape`.

By default, you don't specify any options, but for customizations you can copy the source yaml from https://github.com/andycasey/paper-maker/blob/master/action.yml and modify it.

Advantages:
* It is simple and might be close to what you would do on Ubuntu yourself, if you would install TeX Live from Ubuntu packages.

Disadvantages:
* You have to specify all the packages you want manually.
* Installing TeX Live using the Ubuntu packages is not something that is recommended, because these packages can be older.

Because it would be so much work figuring out what packages are needed, especially when TeX Live was installed using the Ubuntu packages, no example file is provided with this repository.


## LaTeX linting

### LaTeX linter (chktex) by j2kun

## Uploading the pdf to a GitHub Release when a tag is pushed

Basically see https://github.com/actions/upload-release-asset#example-workflow---upload-a-release-asset, example in this repo in [release-pdf-on-tag.yml](.github/workflows/release-pdf-on-tag.yml).

# Overview and comparison of methods to build LaTeX using Travis CI

## Tectonic

Thanks to [Dan Foreman-Mackey](http://dfm.io/posts/travis-latex/) for writing about Tectonic.
The first two methods do not use the pdflatex engine to compile, but [Tectonic](https://tectonic-typesetting.github.io) which is a fork of XeTeX (thanks to [ShreevatsaR](https://tex.stackexchange.com/users/48/shreevatsar) for pointing this out). 
Please leave a comment to vote for adding documentation about using Tectonic at https://github.com/tectonic-typesetting/tectonic/issues/373

Important note: there have been recurring problems with the persistent urls from which the packages are downloaded, if you encounter this (e.g. 'PURL setup link has expired SSL certificate'  or 'couldn't probe ...') then you can try the workaround described at https://github.com/tectonic-typesetting/tectonic/issues/131#issuecomment-435792227

#### Pro:
* automatically loops TeX and BibTeX as needed, and only as much as needed
* automatically downloads LaTeX packages which are needed
* can generate an index
* fastest build time
* also can use biber

#### Con:
* Tectonic does not support the `-shell-escape` flag at the moment (see [tectonic/#38](https://github.com/tectonic-typesetting/tectonic/issues/38)), which is required for example by the minted package. The pdflatex way (below) has been tested to work with the minted package.

We will quickly compare two methods to use Tectonic, after that we will discuss more conventional methods which can compile with pfdlatex, lualatex etc.

Note that since May 2018 Travis moved open source to travis-ci.com (instead of travis-ci.org). Travis is also now available as GitHub App in the Marketplace, but old repositories (like this one) are still on travis-ci.org.

## 1. Docker image with Tectonic

Thanks to [Norbert Pozar (@rekka)](https://tectonic.newton.cx/t/small-docker-image-for-tectonic/133?u=phpirates) for providing the original Docker image.
[Manuel (@WtfJoke)](https://github.com/WtfJoke) extended it by integrating biber.
Docker provides the ability to download a pre-installed Tectonic and then run it on your LaTeX files.

#### Pro:
* Tectonic is ready really fast, because downloading the image is all it takes
* The Docker image is really small, around 75MB
* Works with bibtex automatically
* Can automatically deploy pdfs
* Works with biber

#### Con:
* Slower than the other methods because all packages have to be reinstalled every time

Build time example file: two minutes

Want this? Instructions [below](#tectonic-docker).

## 2. Miniconda with Tectonic

#### Pro:

* It is also fast because tectonic and packages are cached
* Can automatically deploy pdfs
* Can work with Biber, thanks to [Malcolm Ramsay](https://github.com/PHPirates/travis-ci-latex-pdf/pull/11) who made it work.

#### Con:
* Cache is bigger, although still small enough to not really be a disadvantage

Build time example file: 1-2 minutes

Want this? Instructions [below](#tectonic-miniconda).

## 3. Docker image with TeX Live

This method downloads a docker image which contains a small TeX Live installation. Thanks to [Andreas Strauman](https://github.com/Strauman/travis-latexbuild/) for figuring this out, give his [tex.stackexchange.com](https://tex.stackexchange.com/a/450256/98850) answer an upvote if you like it!
At the moment it uses `latexmk` to compile pdfs which is configured to run `pdflatex` by default, but it should be easy to configure it for other tex engines.

#### Pro:

* Very clean build files, only a very short `.travis.yml` file is needed.
* You can specify [options](https://github.com/Strauman/travis-latexbuild/tree/master#configuration-options) in your `.travis.yml`, for example which files to build and which packages to install.
* You can also provide extra `latexmk` [flags](https://man.cx/latexmk#heading4), for example `-dvi` to generate a dvi file.

#### Con:

* Packages are (not yet) cached, so the build time is long
* You still need to specify which packages you require to be installed (unless you use the texliveonfly image, see [the instructions below](texlive-docker-texliveonfly))

Build time example file: 4 minutes (9 minutes when using texliveonfly)

Want this? Instructions [below](#texlive-docker).

## 4. TeX Live with pdflatex/lualatex/latexmk/xelatex/texliveonfly/etc

Thanks to [Joseph Wright](https://tex.stackexchange.com/users/73/joseph-wright) who pointed out that they use something based on this setup for LaTeX3 development.

#### Pro:
* Uses any compiler you want, this can be useful for example if you need pdflatex for some cases like the `minted` package.
* Fast, because of caching
* Almost all required packages are downloaded automatically
* Tested to work with the `minted` package
* Tested to work with [custom fonts](#texlive-custom-fonts)

#### Con:
* You need to copy extra build files to each repository, besides the `.travis.yml`.

Build time example file: 1-2 minutes

Want this? Instructions [below](#pdflatex).

## 5. TeX Live and pdflatex via tinytex with R.

Thanks to [Hugh](https://tex.stackexchange.com/users/18414/hugh) for pointing out this option.

#### Pro:

* Uses pdflatex
* Automatically installs packages needed
* Works with Biber

#### Con:
* You need to specify how much times to compile
* Build time is very long

Build time example file: 5-8 minutes

Want this? Instructions [below](#tinytex).


# Instructions for each build method

## <a name="tectonic-docker">Instructions for building with Docker and Tectonic</a>

* Install the Travis GitHub App by going to the [Marketplace](https://github.com/marketplace/travis-ci), scroll down, select Open Source (also when you want to use private repos) and select 'Install it for free', then 'Complete order and begin installation'. 
* Now you should be in Personal settings | Applications | Travis CI | Configure and you can allow access to repositories, either select repos or all repos.
* Copy [`1-tectonic-docker/.travis.yml`](1-tectonic-docker/.travis.yml) and specify the right tex file in the line with `docker run`. If your tex file is not in the `src/` folder, you also need to change the path in that line after `$TRAVIS_BUILD_DIR`.
* If you want to compile multiple files, you can replace `tectonic main.tex` by `tectonic main.tex; tectonic main2.tex`.
* If you want to use biber, you can use `tectonic --keep-intermediates --reruns 0 main.tex; biber main; tectonic main.tex`
* Commit and push, you can view your repositories at [travis-ci.com](https://travis-ci.com/).
* For deploying to GitHub releases, see the notes [below](#deploy).
* Have a look at the [Tips](#tips).
* If your build doesn't start, see [Troubleshooting](#troubleshooting).

## <a name="tectonic-miniconda">Instructions for building with Miniconda and Tectonic</a>

* Install the Travis GitHub App by going to the [Marketplace](https://github.com/marketplace/travis-ci), scroll down, select Open Source (also when you want to use private repos) and select 'Install it for free', then 'Complete order and begin installation'. 
* Now you should be in Personal settings | Applications | Travis CI | Configure and you can allow access to repositories, either select repos or all repos.
* Copy [`2-tectonic-miniconda/.travis.yml`](2-tectonic-miniconda/.travis.yml) and specify the right tex file in the `script` section in `.travis.yml`. You can uncomment the `makeindex` line and the extra `tectonic` call if you want to use an index.
* Commit and push, you can view your repositories at [travis-ci.com](https://travis-ci.com/).
* For deploying to GitHub releases, see the notes [below](#deploy).
* Have a look at the [Tips](#tips).
* If your build doesn't start, see [Troubleshooting](#troubleshooting).
* If you get the error `This means that your biber (2.xx) and biblatex (3.yy) versions are incompatible` that means that Tectonic has a different biblatex version than the biber version specified in the config. To fix this, open your `.travis.yml` and change `biber==2.xx` to `biber==2.yy`, for example if biblatex 3.11 is present then you need to install biber 2.11. This trick works for any recent version of biblatex, as can be seen in the [biber docs](http://mirrors.ctan.org/biblio/biber/documentation/biber.pdf).

### <a name="biber">Separate instructions for adding biber to your Miniconda and Tectonic setup</a>

These changes have already been added to the `.travis.yml`, but to be clear here are the separate instructions if you already have Miniconda and Tectonic running:
* Install the correct biber version (see above) either from [sourceforge](https://sourceforge.net/projects/biblatex-biber/files/biblatex-biber), or with conda `conda install -c malramsay biber==2.xx` 
* Run tectonic once to create intermediate files, followed by biber, and finally complete compilation of the document with tectonic.

```shell
tectonic --keep-intermediates --reruns 0 ./main.tex
biber main
tectonic ./main.tex
```

* Following this project, Malcolm Ramsay has a great [blog post](https://malramsay.com/post/compiling_latex_on_travis/) documenting this use case in a very complete way.

## <a name="texlive-docker">Instructions for building with TeX Live from a Docker image</a>

* Install the Travis GitHub App by going to the [Marketplace](https://github.com/marketplace/travis-ci), scroll down, select Open Source (also when you want to use private repos) and select 'Install it for free', then 'Complete order and begin installation'. 
* Now you should be in Personal settings | Applications | Travis CI | Configure and you can allow access to repositories, either select repos or all repos.
* Copy [`3-texlive-docker/.travis.yml`](3-texlive-docker/.travis.yml) and specify which tex files you want to build in the `build-pattern` option.
* Add all the required LaTeX packages to the `packages` option, by checking at https://www.ctan.org/pkg/some-package to see in which TeX Live package it is contained (which may be different than the LaTeX package name). Alternatively, use the texliveonfly images as [below](#texlive-docker-texliveonfly).
* Commit and push, you can view your repositories at [travis-ci.com](https://travis-ci.com/).
* For deploying to GitHub releases, see the notes [below](#deploy).
* Have a look at the [Tips](#tips).
* If your build doesn't start, see [Troubleshooting](#troubleshooting).

### <a name="texlive-docker-texliveonfly">Using a docker image with texlive and texliveonfly</a>

You can also use a docker image which uses texliveonfly, it is based on the above-mentioned docker image.
It is somewhat experimental in the sense that sometimes texliveonfly seems to hang (for longer than 10 mins), although usually it seems to work fine.
Texliveonfly calls pdflatex by default, but you can change this and more options in arguments.
Some documentation can be found at https://tex.stackexchange.com/a/463842/98850 which also includes a little list of packages which texliveonfly does not download.

The only change in the instructions is that you use the docker image as specified in [`texlive-docker-texliveonfly/.travis.yml`](texlive-docker-texliveonfly/.travis.yml), and that you don't need to specify packages manually in that file.

The docker source file of this image can be found in https://github.com/Strauman/travis-latexbuild/tree/master/docker/texliveonfly

#### Github Package Registry

Instead of downloading from Docker Hub, you can also download the image from the Github Package Registry.
You can see it at https://github.com/Strauman/travis-latexbuild/packages/92067?version=texliveonfly-2019

This requires you to upload your username and a github token as Travis environment variables.

Just replace the `script:` block with

```yml
script:
  - docker login -u $GH_REGISTRY_USERNAME -p $GH_REGISTRY_TOKEN docker.pkg.github.com
  - docker run --mount src=$TRAVIS_BUILD_DIR/,target=/repo,type=bind docker.pkg.github.com/strauman/travis-latexbuild/texlive-latexbuild:texliveonfly-2019
```

## <a name="pdflatex">Instructions for building with pdflatex/lualatex/latexmk/xelatex/texliveonfly/etc and TeX Live</a>

If for some reason you prefer the pdflatex (or any other) engine with the TeX Live distribution, read on.
This is based on the [LaTeX3 build file](https://github.com/latex3/latex3/blob/master/support/texlive.sh).

This method installs an almost minimal TeX Live installation on Travis, and compiles with pdflatex.
This repo contains:
- The TeX Live install script `texlive_install.sh` including profile `texlive/texlive.profile` (specifies for example the TeX Live scheme)
- A Travis configuration file
- Demonstration LaTeX files in `src/`

### Instructions

* Install the Travis GitHub App by going to the [Marketplace](https://github.com/marketplace/travis-ci), scroll down, select Open Source (also when you want to use private repos) and select 'Install it for free', then 'Complete order and begin installation'. 
 * Now you should be in Personal settings | Applications | Travis CI | Configure and you can allow access to repositories, either select repos or all repos.
* Copy the files in the folder [`4-texlive`](4-texlive) to your repo, so `.travis.yml` and the `texlive/` folder.
* Specify the right tex file in the `.travis.yml` under the `script` section. Possibly you also need to change the folder in `before_script` if not using `src/`.
* Commit and push, you can view your repositories at [travis-ci.com](https://travis-ci.com/).
(Thanks to [@jason-neal](https://github.com/PHPirates/travis-ci-latex-pdf/pull/6) for improving this)
* For deploying to GitHub releases, see the notes [below](#deploy).
* If the build fails because some package is missing you have to add it manually in `texlive_packages`. This configuration uses the `texliveonfly` package which tries to download missing packages but sometimes the error message is non-standard and that fails. 

  In that case, put the package you want to install in `texlive/texlive_packages`, by checking at https://www.ctan.org/pkg/some-package to see in which TeX Live package it is contained (which may be different than the LaTeX package name). If you find a package which is not automatically downloaded, it would be great if you could let us know by submitting an issue.

  You can find required LaTeX packages of your document (say main.tex) by installing the snapshot package and then running `pdflatex -draftmode -interaction=batchmode "\RequirePackage{snapshot}\input{main}"` (thanks to [Pablo Gonzalez](https://github.com/PHPirates/travis-ci-latex-pdf/pull/24#issuecomment-521780469) for finding this).
   This will produce a file `snapshot.dep` with dependencies. Note this requires that you have all required packages installed on your system.
   
* If you use custom fonts, [read on](#texlive-custom-fonts).
* Tip from [gvacaliuc](https://github.com/gvacaliuc/travis-ci-latex-pdf): In order to maintain the install scripts in a central repo and link to them, you could also just copy `.travis.yml` and replace
```yaml
install:
 - source ./texlive/texlive_install.sh
```
with
```yaml
install:
  - mkdir ./texlive/
  - 4-texlive
  - 4-texlive
  - 4-texlive
  - source ./texlive/texlive_install.sh
```
* Preferably you fork this repo so you can maintain your own build files with the right packages.

Note that sometimes `tlmgr` selects a broken mirror to download TeX Live from, so you get an error like `Output was gpg: verify signatures failed: eof`. Restarting the build will probably fix this, it will auto-select a different mirror. (Thanks to [@jason-neal](https://github.com/jason-neal/travis-ci-latex-pdf-texlive/commit/d48a5f92d2394f27371dd32c94a16415de499058) for testing this.)

You can also read a nice blog post by Jeremy Grifski ([@jrg94](https://github.com/jrg94)) about using the setup from this repo including the minted package at https://therenegadecoder.com/code/how-to-build-latex-with-travis-ci-and-minted/

### <a name="texlive-custom-fonts">Using custom fonts</a>

In order to enable Travis to use custom fonts, i.e. those fonts not provided by Ubuntu or TeX Live packages, it is probably easiest to push them with git.
An example is in this repo, with the file [src/fonts.tex](src/fonts.tex) and the fonts in [src/fonts](src/fonts).
You can do the following to use them on Travis:
* Make sure you have the fonts pushed to you GitHub repository
* Make sure that your `texlive_packages` file contains the packages `collection-fontsrecommended` and `luaotfload` 
* If you are starting from scratch, you can adapt [4-texlive/fonts-travis-yml/.travis.yml](4-texlive/fonts-travis-yml/.travis.yml). If you want to change your current .travis.yml, do the following:
    * In the `.travis.yml`, uncomment the following lines (make sure to change src/fonts to your own path if it is different)
    ```yaml
    before_install:
      - mkdir $HOME/.fonts
      - cp -a $TRAVIS_BUILD_DIR/src/fonts/. $HOME/.fonts/
      - fc-cache -f -v
    ```
    * Make sure to compile with lualatex or xelatex, for example if you previously had `texliveonfly main.tex` and `latexmk -pdf main.tex` in your `.travis.yml`, then change them to `texliveonfly main.tex --compiler=lualatex` and `latexmk -pdflua main.tex`.

## <a name="tinytex">Instructions for building with TeX Live and pdflatex via tinytex with R</a>

* * Install the Travis GitHub App by going to the [Marketplace](https://github.com/marketplace/travis-ci), scroll down, select Open Source (also when you want to use private repos) and select 'Install it for free', then 'Complete order and begin installation'. 
  * Now you should be in Personal settings | Applications | Travis CI | Configure and you can allow access to repositories, either select repos or all repos.
* Copy the files in the folder `5-tinytex` to your repo, so `.travis.yml` and `install_texlive.R`.
* Specify the right tex file in `.travis.yml`.
* Commit and push, you can view your repositories at [travis-ci.com](https://travis-ci.com/).
* For deploying to GitHub releases, see the notes [below](#deploy).
* Have a look at the [Tips](#tips).
* If your build doesn't start, see [Troubleshooting](#troubleshooting).

## <a name="deploy">To automatically deploy pdfs to GitHub release</a>
### First time setup

We will add a configuration to the `.travis.yml` such that a pdf will be automatically uploaded to GitHub releases when you tag a commit, also see the [Travis documentation](https://docs.travis-ci.com/user/deployment/releases/).

* We will generate a GitHub OAuth key so Travis can push to your releases, with the important difference (compared to just gettting it via GitHub settings) that it's encryped so you can push it safely.
* (Windows) [Download ruby](https://rubyinstaller.org/downloads/) and at at end of the installation make sure to install MSYS including development kit.
* Run `gem install travis --no-document` to install the Travis Command-line Tool.
### For every new project
* Go to the directory of your repository, open the command prompt (Windows: <kbd>SHIFT</kbd>+<kbd>F10</kbd> <kbd>W</kbd> <kbd>ENTER</kbd>) and run `travis login --pro --github-token mytoken` where `mytoken` is a token from https://github.com/settings/tokens (or leave the `--github-token` flag out if you want to use your password). 
* Run `travis setup releases --pro` and leave File to Upload empty.
* Replace everything below your encryped api key with (changing the path to your pdf file, probably the same folder as your tex file is in)
```yml
  file: 
  - ./src/nameofmytexfile.pdf
  - ./otherfile.pdf
  skip_cleanup: true
  on:
    tags: true
    branch: master
```
* Commit and push.
* If you are ready to release, just tag and push.

## <a name="tips">Tips</a>
* You can tell Travis to skip the build for a certain commit by prefixing the commit message with `[ci skip]`.
* If you want the badge in your readme, just copy the code below to your readme and change the links.

Markdown:
```markdown
[![Build Status](https://api.travis-ci.com/username/reponame.svg)](https://travis-ci.com/username/reponame)
```
reStructuredText:
```rst
.. image:: https://travis-ci.com/username/reponame.svg?branch=master
    :target: https://travis-ci.com/username/reponame
    :alt: Build Status
```
* You may want to edit settings on Travis to not build both on pull request and branch updates, and cancel running jobs if new ones are pushed.
* You can automatically add a footnote, tag and/or git hash to the document by adding the following lines to your `.travis.yml`:
```
before_install:
  - chmod +x .travis/git-info-2.sh
  - ".travis/git-info-2.sh"
```
Also copy the [`.travis/git-info-2.sh`](.travis/git-info-2.sh) to your repository.
Then in your LaTeX file add `\usepackage{gitinfo2}`.

Now you can for example get the hash of the commit with `\gitHash`.
An example can be found in [`src/git-info-2-mwe.tex`](src/git-info-2-mwe.tex).
For more info, see the documentation at http://mirrors.ctan.org/macros/latex/contrib/gitinfo2/gitinfo2.pdf 
(Thanks to [@shabbychef](https://github.com/Strauman/travis-latexbuild/issues/27) for mentioning this)

## <a name="troubleshooting">Troubleshooting</a>

* When trying to fix your `.travis.yml`, it may be helpful to trigger custom builds instead of creating a new commit every time, as explained in https://blog.travis-ci.com/2017-08-24-trigger-custom-build

* If your build doesn't start you should first look at Travis (so on https://travis-ci.com/username/reponame) under More Options | Requests, it might for example be that the `.travis.yml` could not be parsed, for example because your indentation is wrong.
You can also manually trigger a build there.

* If you do not understand why your build is failing, it may help to run the relevant commands on a local Ubuntu system, if you have one.

## Notes
* There are much more CI services than just Travis, for example CircleCI or SemaphoreCI and [much more](https://github.com/ligurio/awesome-ci). If you manage to use one of them, it would be great if you could report back!

I also put some of these instructions on the [TeX Stackexchange](https://tex.stackexchange.com/questions/398830/how-to-build-my-latex-automatically-with-pdflatex-using-travis-ci/398831#398831).

Some original thoughts from [harshjv's blog](https://harshjv.github.io/blog/setup-latex-pdf-build-using-travis-ci/), and thanks to [jackolney](https://github.com/jackolney/travis-ci-latex-pdf) for all his attempts to put it into practice.
Also see harshjv's original [blog post](https://harshjv.github.io/blog/document-building-versioning-with-tex-document-git-continuous-integration-dropbox/).

# Contributing

If you want to add/update a method to build LaTeX, look in the [contributing guidelines](.github/contributing.rst).

# <a name="gitlab">GitLab CI</a>

At the moment, only the Texlive method explained [above](#pdflatex) was tested in GitLab for this repo.
The instructions are similar but with of course a different configuration file, which can be found in [gitlab/texlive](gitlab/texlive) for now. This still has to be moved to GitLab.
The only disadvantage is that the caching of TeX Live does not work, because it is not installed in the project, but artifacts have to be in the project directory for GitLab.
The build time for the example file is around four minutes.

A more extensive overview of configurations for GitLab CI will come in the future, either in this or an other repo.
Please contribute if you can help with this!
Some links to get started:

https://tex.stackexchange.com/questions/459484/compiling-latex-files-automatically-with-gitlab-ci

https://www.vipinajayakumar.com/continuous-integration-of-latex-projects-with-gitlab-pages.html

https://sayantangkhan.github.io/latex-gitlab-ci.html

https://tex.stackexchange.com/questions/412740/gitlab-ci-runner-with-relative-paths-in-main-tex

https://tex.stackexchange.com/questions/437553/gitlab-ci-using-miktex-docker-image

<!-- https://miktex.org/howto/miktex-docker -->

# <a name="context">ConTeXt</a>

Following a [TeX Stackexchange answer from the TeXnician](https://tex.stackexchange.com/a/459487/98850) and the official docs at https://wiki.contextgarden.net/ConTeXt_Standalone#Single_user_installation it is not too difficult to install ConTeXt (and cache the installation).

As an alternative, you could use a Docker image, for example one from https://gitlab.com/islandoftex/images/context

In this repository there is an example for Github Actions, see [.github/workflows/context.yml](.github/workflows/context.yml), but it should be easy to extend to other build systems as well.

## Usage of the Github Action

* Copy [.github/workflows/context.yml](.github/workflows/context.yml) to your repository in `.github/workflows`
* Adjust the path to your tex file and possibly other things you want customized
* Commit and push
* On GitHub, check the Actions tab

This actions caches the ConTeXt installation.
Build time example file: around 30-40 sec.
