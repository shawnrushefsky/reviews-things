---
title: "Building and Deploying Static Sites with Hugo, Circle CI, and Google Cloud Storage"
date: 2018-11-10T12:45:18-06:00
draft: true
---

I built this website because I wanted to play with a static site builder, and set up a pipeline to easily deploy it to the web. I'll go over the tools I used, why I chose them, and all of the steps I took with each. 

## Tools

Basically everything I used here is free and open-source, except for the cloud storage provider, which is still incredibly cheap at $0.026/GB/month. So, basically free, but you will need to provide a credit card.

[Hugo](https://gohugo.io) is a static site builder that presents a more performant alternative to [Jekkyl](https://jekyllrb.com/) with some extra features.  This website is built with Hugo.

[GitHub](https://github.com) is a code repository. You're probably familiar with it.

[Circle CI](https://circleci.com) is a continuous integration and deployment platform.

[Google Cloud Storage](https://cloud.google.com/storage/) is a cloud-based multi-region blob storage similar to [Amazon S3](https://aws.amazon.com/s3/).

## Hugo

I found it very easy to get set up with Hugo. I'm using a mac with [homebrew](https://brew.sh), so it was as easy as

```shell
brew install hugo
hugo new site <site-name>
cd <site-name>
git init
```

I set up my theme:

```shell
git submodule add https://github.com/lgaida/mediumish-gohugo-theme themes/mediumish
```

Then, I added this page with

```shell
hugo new blog/static-sites.md
```

You can see the original markdown file [here](https://github.com/shawnrushefsky/reviews-things/blob/master/content/reviews/Hugo.md)