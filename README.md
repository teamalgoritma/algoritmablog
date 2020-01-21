# :star2: Overview Algotech

*Algotech* is a Website for technical blog Algoritma. To serve the website locally and to view a live version run the following command:

```
blogdown::serve_site()
```

![](public/img/main/ss1.png)


***

# :memo: Contributing Articles

Want to contribute an article? Please follow the submission guidelines below:

1. Fork the repository.
2. Make a some development version or added a new article.
3. Submit a *pull request*.

If the project owner agrees with your work, they might merged your request into the original repository.

## 1. Fork a Repo 

**Step-1**:

1. On github, navigate to the [teamalgoritma/algoritmablog](https://github.com/teamalgoritma/algoritmablog) repository.
2. On the top-right corner of the page, click *fork*.

**Step-2**: Keep your fork synced using Git. If you haven't yet, you should first [set up Git](https://help.github.com/en/articles/set-up-git#setting-up-git).

1. On github, to **your fork** of the *algoritmablog* repository.
2. Open Git Bash
3. Type `git clone`, and then paste the URL of repository. It will look like this, with your GitHub username instead of `YOUR-USERNAME`:

```
$ git clone https://github.com/YOUR-USERNAME/algoritmablog
```
4. Press **Enter**. Your local clone will be created.

```
$ git clone https://github.com/YOUR-USERNAME/algoritmablog
Cloning into 'algoritmablog'...
remote: Enumerating objects: 1592, done.
remote: Counting objects: 100% (1592/1592), done.
remote: Compressing objects: 100% (731/731), done.
remote: Total 1592 (delta 770), reused 1495 (delta 679), pack-reused 0
Receiving objects: 100% (1592/1592), 13.38 MiB | 1.05 MiB/s, done.
Resolving deltas: 100% (770/770), done.
```

5. Type `git remote -v` and press Enter. You'll see the current configured remote repository for your fork.

```
$ git remote -v
> origin  https://github.com/YOUR_USERNAME/algoritmablog.git (fetch)
> origin  https://github.com/YOUR_USERNAME/algoritmablog.git (push)
```

## 2. Create an Article

### YAML Options

Use the following template:

```
title: Creating Choropleth with Mapshaper and R
author: Ardhito Utomo
github: https://github.com/ardhitoutomo
date: '2019-08-18'
slug: creating-choropleth-with-mapshaper-and-r
categories:
  - R
tags: 
  - Geocoding
  - Mapshaper
  - Leaflet
  - Map
  - Data Visualization
description: ''
featured: ''
featuredalt: ''
featuredpath: ''
linktitle: ''
type: post
```

**OR**:

Create a new article by using `New Post` addins on your Rstudio (make sure you already installed *blogdown* packages first).

![](public/img/main/newpost.png)


`Tags` are keywords or terms that describe your article. Here are some tags that already exist: 

* `tidyverse`
* `tidymodels`
* `Data Visualization`
* `Machine Learning`
* `Data Manipulation`
* `Capstone ML`

*Note*: Please add `Capstone ML` tags on your YAML options, if you are making an article that aims to help student on capstone project.

### Setup Chunk

Use the following setup chunk options on your Rmd files.

```
# clean up the environment
rm(list = ls())

# setup chunk options
knitr::opts_chunk$set(
  message = FALSE,
  warning = FALSE,
  fig.align = "center",
  comment = "#>"
)
```

### Insert Images

1. Create new folder *your topic article* on `/public/img/`.
2. Save your images on `/public/img/ARTICLE-TOPIC` with a template `imgname.png`.
3. Embed the images with the following command, and options `echo = FALSE`.

```
knitr::include_graphics('/img/article-topic/imgname.png')
```

## 3. Submit Pull Request! :grin:

