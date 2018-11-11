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
    thumbnail = "/images/shawn-profile.JPG"
    description = "A grump old man in a grumpy young man's body."

[params.index]
  picture = "/images/shawn-profile.JPG"
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

This sets up the menu and some basic information about me that the theme generates into a home page.

Then, I added this page with

```shell
hugo new blog/static-sites.md
```

You can see the original markdown file [here](https://github.com/shawnrushefsky/reviews-things/blob/master/content/reviews/Hugo.md)