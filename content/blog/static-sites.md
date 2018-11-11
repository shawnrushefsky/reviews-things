---
title: "Building and Deploying Static Sites with Hugo, Circle CI, and Google Cloud Storage"
date: 2018-11-10T12:45:18-06:00
draft: true
---

I built this website because I wanted to play with a static site builder, and set up a pipeline to easily deploy it to the web. I'll go over the tools I used, why I chose them, and all of the steps I took with each. 

## Tools

Basically everything I used here is free and open-source, except for the cloud storage provider, which is still incredibly cheap at $0.026/GB/month. So, basically free, but you will need to provide a credit card.

* [Hugo](https://gohugo.io) is a static site builder that presents a more performant alternative to [Jekyll](https://jekyllrb.com/) with some extra features.  This website is built with Hugo.
* [GitHub](https://github.com) is a code repository. You're probably familiar with it.
* [Circle CI](https://circleci.com) is a continuous integration and deployment platform.
* [Google Cloud Storage](https://cloud.google.com/storage/) is a cloud-based multi-region blob storage similar to [Amazon S3](https://aws.amazon.com/s3/).

## Hugo

### Why Hugo?

My goal was a site that I could manage via markdown files and git-based version control, so a static site generator was the natural fit.

I was choosing between [Hugo](https://gohugo.io) and [Jekyll](https://jekyllrb.com/) for this project, and ended up with Hugo for a number of reasons:

1. Hugo is much faster than Jekyll, taking an average of <1ms to build a page, meaning all but the largest sites will build in under 1 second.

2. Jekyll requires you to have Ruby installed, and themes are managed via Ruby gems. Hugo is a compiled binary written in Go, and can be installed with the standard package manager for your platform. Themes are managed with .gitmodules. I prefer this, because I don't regularly work in Ruby, but I do use [homebrew](https://brew.sh) and [GitHub](https://github.com) all the time.

3. Quite a few more features out-of-the-box (no plugins required), like comments and dynamic API driven content. I don't currently have plans for API-driven content, but I could get bored and add some. Who knows?

### What to Do

#### Getting Set Up

I found it very easy to get set up with Hugo. I'm using OS X, so it was as easy as

```shell
brew install hugo
hugo new site <site-name>
cd <site-name>
git init
```

#### Choosing a Theme

I went to the [Hugo Themes](https://themes.gohugo.io/) page, and filtered for "responsive" themes. It's 2018, and you can't have a website that doesn't perform gracefully on a mobile device. I ended up with a theme called [mediumish](https://github.com/lgaida/mediumish-gohugo-theme) that resembles a certain popular online publishing platform.

I set up my theme, using gitmodules:

```shell
git submodule add https://github.com/lgaida/mediumish-gohugo-theme themes/mediumish
```

This is a great way to distribute themes, because it relies on no dependencies outside of `git`, and still lets you benefit from bug fixes and updates from the theme author.

Now, add your theme to the config:

```shell
echo 'theme = "mediumish"' >> config.toml
```

Alternatively, you can open this directory in your prefered editor, and add the line to `config.toml` that way.

The final file will look like:

```toml
baseURL = "http://example.org/"
languageCode = "en-us"
title = "My New Hugo Site"
theme = "mediumish"

```

#### Serve it Up

Hugo comes with an auto-reloading server that hosts your site locally, and automatically refreshes when you make changes to any of the files in your project.

```shell
hugo serve -D
```

And then navigate to http://localhost:1313 to see it in action. At this point you may notice that your site is pretty generic, and titled "My New Hugo Site".

#### Customize

My final `config.toml` looks like:

```toml
baseURL = "https://www.shawnreviewsthings.com"
languageCode = "en-us"
title = "Shawn Reviews Things"
theme = "mediumish"
enableEmoji = true
copyright = "Shawn Rushefsky - All rights reserved"
pygmentsCodeFences = true # This enables syntax highlighting in code blocks like this one

[params]
    description = "I have a lot of opinions."

[params.author]
    name = "Shawn Rushefsky"
    thumbnail = "/images/shawn-profile.jpg"
    description = "A grump old man in a grumpy young man's body."

[params.index]
  picture = "/images/shawn-profile.jpg"
  title = "Shawn Rushefsky"
  subtitle = "I have a lot of opinions."
  mdtext = "I review all kinds of things. Gadgets, restuarants, hotels, open source projects, enterprise services, dogs, etc."
  alertbar = false

[[menu.main]]
    name = "Blog"
    identifier = "blog"
    weight = 200
    url = "/blog"

[[menu.main]]
    name = "About Me"
    identifier = "about"
    weight = 201
    url = "/"
```

> **NOTE**
> 
In `[params.author]`, 
```toml
thumbnail = "/images/shawn-profile.jpg"
```
references the path `static/images/shawn-profile.jpg` in the project directory.

This sets up the menu and some basic information about me that the theme generates into a home page. Once I actually start reviewing things, I'll add another menu block:

```toml
[[menu.main]]
    name = "Reviews"
    identifier = "reviews"
    weight = 199
    url = "/reviews"
```

#### Add Content

Ok, now you have a home page, let's add some content. 

I added this page with

```shell
hugo new blog/static-sites.md
```

Which creates a new file at `content/blog/static-sites.md`

And then wrote this blog post.

You can see the original markdown file [here](https://github.com/shawnrushefsky/reviews-things/blob/master/content/reviews/Hugo.md)

If you go back to your site in your browser, you'll see your site working. You can click links in the menu, and they'll take you to a page with entries of that type. For instance, in this menu block:

```toml
[[menu.main]]
    name = "Blog"
    identifier = "blog"
    weight = 200
    url = "/blog"
```

```toml
url = "/blog"
```

will go to a directory page for each page you put in the path `content/blog/`. For instance, this page is stored at `content/blog/static-sites.md`.

### Summary

Ok, you've got a basic hugo site working. Now, how do you get this thing on the internet?

## GitHub

### Why GitHub?

As a software engineer, I really like storing as much as possible in GitHub. It's a natural fit for asyncronous collaboration on text-oriented projects, even if those projects aren't code. Storing this uncompiled site in GitHub also makes it very easy to automatically build it and upload it to our production storage, Google Cloud Storage in this case.

Additionally, it means you can have new articles submitted via the Pull Request mechanism, which provides an opportunity for peer or editor review before publishing. These benefits don't necessarily apply to my project, since this is just a personal blog and I am currently the only contributor. Still, it never hurts to plan for scale.

I'm aware there are other code repositories, and any git-compatible repository you choose will work for the purposes of this tutorial.

### What to Do

Commit all the changes you've made to your site, and publish your repository. If you make your repository "Public", it is free to store it on GitHub, and free to use CircleCI to build and deploy it. If you prefer for it to be private, you can pay a small monthly fee ($7/mo for unlimited private repos on GitHub).

## Circle CI

[Circle CI](https://circleci.com) is a [Continuous Integration](https://en.wikipedia.org/wiki/Continuous_integration) and [Continuous Deployment](https://en.wikipedia.org/wiki/Continuous_deployment) platform. The idea behind such platforms is that you can automatically run jobs every time code is committed to GitHub, or similar code repository. Jobs can include testing, builds, and deployments.

### Why Circle CI?

Circle CI has a lot of benefits:

1. It integrates automatically with GitHub
2. It is free for open source projects (which this is one)
3. It is well documented, and easy to get started with

These benefits aren't unique to Circle CI, and can also be found in products like [Travis CI](https://travis-ci.org/). I really chose it because I have used it for work and was already familiar with their interface. My walkthrough will be specific to Circle CI, but the concepts apply equally well to any similar service.

### What to Do

#### Getting Started

Go to [Circle CI](https://circleci.com) and click Sign Up. This is easiest if you choose "Sign Up with Github", because it will automatically link your GitHub account.

#### Create a New Project

1. Click "Add Projects" in the left-hand tool bar. 
2. Find your new repository, and click "Set up Project".
3. Choose "Linux" for Operating System
4. Choose "Other" for Language

#### Configure your Project

Back in your editor, make a new directory in the repo called `.circleci`, and create a file in that directory called `config.yml`. This file is tells Circle CI what to do with your code on new commits.  In our case, we are going to describe 2 `jobs`: `build`, and `deploy`. We're also going to define a `workflow` that connects those `jobs`.

First, we set up some boilerplate stuff in the configuration:

```yml
version: 2

jobs:
  build:
    docker:
      - image: cibuilds/hugo:latest
    working_directory: ~/hugo
    environment:
      HUGO_BUILD_DIR: public/site
```

This means our build environment is going to use the docker image [cibuilds/hugo](https://hub.docker.com/r/cibuilds/hugo/), which comes with all the hugo commands we need to build our project.

However, this image is based on alpine linux, and is missing 