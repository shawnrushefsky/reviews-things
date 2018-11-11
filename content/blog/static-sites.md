---
title: "Building and Deploying Static Sites with Hugo, Circle CI, and Google Cloud Storage"
date: 2018-11-11T12:35:18-06:00
draft: false
tags: ['Static Sites', 'CI/CD', 'Circle CI', 'Cloud', 'Google Cloud Storage', 'Markdown', 'Hugo', 'GitHub', 'Tutorial']
---

I built this website because I wanted to play with a static site builder, and set up a pipeline to easily deploy it to the web. I'll go over the tools I used, why I chose them, and all of the steps I took with each. While anyone familiar with markdown could probably make their way through this tutorial, it is definitely aimed at developers who want a simple but developer-friendly workflow for administering a website.

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
baseURL = "http://www.shawnreviewsthings.com"
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

Which creates a new file at `content/blog/static-sites.md`.

This post comes from that file.

You can see the original markdown file [here](https://github.com/shawnrushefsky/reviews-things/blob/master/content/blog/static-sites.md)

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

## Google Cloud Storage

[Google Cloud Storage](https://cloud.google.com/storage/) is a cloud-based multi-region blob storage similar to [Amazon S3](https://aws.amazon.com/s3/).

### Why Google Cloud Storage

My initial instinct was to use S3, but it turns out I owed AWS $60, and while I was happy to pay it, it was going to take a day or so to restore access to my account.

Next I went to Azure, because we use them at work, and their bucket storage is a service I hadn't had a chance to play with yet. Alas, my work uses single sign-on, so it was going to be a big hassle to log in with anything other than my work credentials, and this is definitely a personal project.

So, by process of elimination, I ended up with Google Cloud Storage. All of these services are fairly similar, and fairly cost comparable, so ultimately it doesn't really matter what you choose here.

### What to Do

This is probably the most complex part, but it's really not too bad, and you can definitely do it.

1. Create a google account if you somehow don't have one already
2. Navigate to the [storage console](https://console.cloud.google.com/storage/)
3. Create a new project. Mine is called "Shawn Reviews Things".
4. You will need to authorize payments in order to create a Storage bucket. Go ahead and follow their directions for doing so. Review their prices if you need to, but it's extremely cheap ($0.026/gb/mo).
5. Create a new Bucket. 
    1. Name your bucket "www.yoursiteurl.com". It's important to note that you do need to own this URL already at this point. I used [Namecheap](https://www.namecheap.com) to purchase my domain name for about $9/year.
    2. Choose "Multi-Regional" for your Storage Class. This will make all your files available in multiple regions simultaneously, for better durability, availability, and read performance.
    3. I chose "United States" for location, because that's where I live. You could choose something else if you want.
    4. Click Create
    5. At this point, it's going to ask you to verify that you own your domain name by adding a `TXT Record` to your DNS. They provide instructions for doing this that are specific to your domain name provider. Sometimes this verification can take a while, but it was essentially real-time for me in this case.
    6. While you're in your DNS settings, make a new `CNAME Record`:
        - Host: "www"
        - Value: "c.storage.googleapis.com." (Note the trailing `.` there. It's important)
    7. Congrats! You made your bucket.
6. Create a new User for Circle CI to use
    1. Go to the [Service Account Console](https://console.cloud.google.com/iam-admin/serviceaccounts)
    2. Click "Create Service Account"
    3. Name your new account "circle-ci". This will make it easy to remember what this account is for.
    4. Add a description if you want more information, then click "Create"
    5. Select the role "Storage Object Admin" for your project. This will let this new Service User have access to create, modify, and destroy resources inside of your storage bucket. It's important to have the full access, because your files will be overriden when changes occur.
    6. Click "Continue".
    7. Give yourself access to this role, and then click "Create Key".
    8. Choose the "JSON" option, which should be selected by default, and click "Create". This will download the key to your computer as a JSON file. Remember where you put it, because you're going to need it again later.
7. Set up access permissions for your bucket
    1. From the [Storage Browser](https://console.cloud.google.com/storage/browser), you should be able to see your new bucket. 
    2. Click the menu button for your bucket, located on on the far right of its row.
    3. Click "Edit Bucket Permissions"
    4. Under the **Add Members** section, type "allUsers", and select the role "Storage Object Viewer". This will set the contents of your bucket to be publically readable. This is important if you want people other than you to be able to see your website. Don't worry, they won't have write access.
8. Configure the Bucket to be a Website
    1. Click the menu button for your bucket
    2. Click "Edit website configuration"
    3. For Main page, put "index.html"
    4. For 404 (Not Found) page, put "404.html"


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

##### Build Job

We're going to start by defining the build job.

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

This means our build environment is going to use the docker image [cibuilds/hugo](https://hub.docker.com/r/cibuilds/hugo/), which comes with all the hugo and git commands we need to build our project.

We're going to be cloning our repo down into `~/hugo`, and building the site into `~/hugo/public/site`

Next, we define the steps to take for this job

```yml
    steps:
      # checkout the repository
      - checkout

      # install git submodules for managing third-party dependencies
      - run: git submodule sync && git submodule update --init

      # build with Hugo
      - run: HUGO_ENV=production hugo -v -d $HUGO_BUILD_DIR
 
      - persist_to_workspace:
          root: public
          paths:
            - site
```

This clones down our repo, installs any gitmodules, like our theme, and then builds the site. When you set `HUGO_ENV=production`, it will ignore any content with the flag `draft: true` at the top. Set that to `draft: false` when you're ready to release a new piece of content.

Finally, it persists our built site to the the `workspace`, which is a concept Circle CI gives us to persist data between different jobs.

`config.yml` should look like this at this point:

```yml
version: 2

jobs:
  build:
    docker:
      - image: cibuilds/hugo:latest
    working_directory: ~/hugo
    environment:
      HUGO_BUILD_DIR: public/site
    steps:
      # checkout the repository
      - checkout

      # install git submodules for managing third-party dependencies
      - run: git submodule sync && git submodule update --init

      # build with Hugo
      - run: HUGO_ENV=production hugo -v -d $HUGO_BUILD_DIR
 
      - persist_to_workspace:
          root: public
          paths:
            - site
```

Go ahead and commit and push this file to your repo at this point.

Back in the Circle CI interface, click "Start Building". This will create your project and run the "build" job.

##### Project Settings

At this point we need to add a couple environment variables to our project so that our Deploy job can make authorized requests to our GCS (Google Cloud Storage) Bucket.

You should be in the "Jobs" panel at this point, and you'll see your project there, with a job status for the "master" branch. Click the gear next to your project title to go to your project settings.

Navigate to "Environment Variables".

You'll need to add 3 variables:

1. `GCLOUD_SERVICE_KEY` should be set to the full text of that JSON key you downloaded earlier
2. `GOOGLE_PROJECT_ID` should be set to the ID of your project, which you can find from the Google Cloud Console. For me it was "shawn-reviews-things", because the plain name was "Shawn Reviews Things".
4. `BUCKET_NAME` should be the name of the bucket you made in GCS. For me it was "www.shawnreviewsthings.com"

Next, go to Advanced Settings, and turn on "Only build pull requests". This will prevent circle from building every commit to every branch, and instead it will only build commits to master, and pull requests.

##### Deploy Job

Ok, now that that is configured, we can go back to our editor and define our "deploy" job.

Our "build" job builds the site, and persists the compiled final version to the workspace. Now we need to pull that data into our deploy container.

```yml
  deploy:
    docker:
      - image: google/cloud-sdk
    steps:
      - attach_workspace:
          # Must be absolute path or relative path from working_directory
          at: /tmp/workspace
```

We're going to use the Docker image [google/cloud-sdk](https://hub.docker.com/r/google/cloud-sdk/), because it has all the dependencies we need to upload our project to our Google Cloud Storage Bucket.

We attach the workspace at `/tmp/workspace`, which means our site will be the contents of `/tmp/workspace/site/`

We need to put our JSON key back in a file that the google cli can use:

```yml
      - run:
          name: Create keyfile from env
          command: echo $GCLOUD_SERVICE_KEY >> /tmp/key.json
```

Next we need to set up the cli to use our configuration:

```yml
      - run:
          name: Set up gcloud
          command: gcloud auth activate-service-account --key-file=/tmp/key.json && gcloud --quiet config set project ${GOOGLE_PROJECT_ID}
```

Finally, we can upload our site to the bucket:

```yml
      - run:
          name: Upload to Storage Bucket
          command: gsutil cp -r /tmp/workspace/site/. gs://${BUCKET_NAME}
```

At this point, your `config.yml` should look like this:

```yml
version: 2

jobs:
  build:
    docker:
      - image: cibuilds/hugo:latest
    working_directory: ~/hugo
    environment:
      HUGO_BUILD_DIR: public/site
    steps:
      # checkout the repository
      - checkout

      # install git submodules for managing third-party dependencies
      - run: git submodule sync && git submodule update --init

      # build with Hugo
      - run: HUGO_ENV=production hugo -v -d $HUGO_BUILD_DIR
 
      - persist_to_workspace:
          root: public
          paths:
            - site

  deploy:
    docker:
      - image: google/cloud-sdk
    steps:
      - attach_workspace:
          # Must be absolute path or relative path from working_directory
          at: /tmp/workspace

      - run:
          name: Create keyfile from env
          command: echo $GCLOUD_SERVICE_KEY >> /tmp/key.json

      - run:
          name: Set up gcloud
          command: gcloud auth activate-service-account --key-file=/tmp/key.json && gcloud --quiet config set project ${GOOGLE_PROJECT_ID}
    
      - run:
          name: Upload to Storage Bucket
          command: gsutil cp -r /tmp/workspace/site/. gs://${BUCKET_NAME}
```

##### Define A Workflow

Now that you have defined your jobs, it's time to define how they go together.

Add this to the end of your `config.yml`:

```yml
workflows:
  version: 2
  build-deploy:
    jobs:
      - build
      - deploy:
          requires:
            - build
          filters:
            branches:
              only: master
```

What this is doing is creating a workflow called `build-deploy` that contains 2 jobs, `build` and `deploy`.

It specifies that `deploy` requires the success of `build`, and should only run for the `master` branch of your repo. This will keep expiriments in other branches from making their way to your live site, until you merge them into `master`.

All together, you config looks like:

```yml
version: 2

jobs:
  build:
    docker:
      - image: cibuilds/hugo:latest
    working_directory: ~/hugo
    environment:
      HUGO_BUILD_DIR: public/site
    steps:
      # checkout the repository
      - checkout

      # install git submodules for managing third-party dependencies
      - run: git submodule sync && git submodule update --init

      # build with Hugo
      - run: HUGO_ENV=production hugo -v -d $HUGO_BUILD_DIR
 
      - persist_to_workspace:
          root: public
          paths:
            - site

  deploy:
    docker:
      - image: google/cloud-sdk
    steps:
      - attach_workspace:
          # Must be absolute path or relative path from working_directory
          at: /tmp/workspace

      - run:
          name: Create keyfile from env
          command: echo $GCLOUD_SERVICE_KEY >> /tmp/key.json

      - run:
          name: Set up gcloud
          command: gcloud auth activate-service-account --key-file=/tmp/key.json && gcloud --quiet config set project ${GOOGLE_PROJECT_ID}
    
      - run:
          name: Upload to Storage Bucket
          command: gsutil cp -r /tmp/workspace/site/. gs://${BUCKET_NAME}

workflows:
  version: 2
  build-deploy:
    jobs:
      - build
      - deploy:
          requires:
            - build
          filters:
            branches:
              only: master
```

Commit these changes to your repo, and push.

Back in your Circle CI Jobs interface, you should be able to see your jobs run, first `build`, then `deploy`, if you've done everything correctly.

Once `deploy` has succeeded, you should be able to navigate to your [URL](www.shawnreviewsthings.com) and see your site.

## Conclusions

If you're like me and you prefer to write everything in markdown, and have it work without a lot of fuss, this combo of `Static Site Generator + CI/CD + Bucket Storage` is really nice. I can use my preferred editor, [VS Code](https://code.visualstudio.com/) to do all of my editing, and my site is automatically deployed when I merge new content into master. It's a workflow that I'm already comfortable and familiar with, and provides a lot of convenience.

Beyond the convenience, it's also incredibly cost effective, especially if you keep your site repo public on github. All in, this project is costing me less than $0.80/mo, and that includes domain name, hosting, bandwidth, everything.

While I don't think it's very likely to occur, this is also a highly scalable setup. GitHub provides an efficient mechanism for very large numbers of contributors to submit new content for a site like this, and Circle CI will automatically build and deploy these changes as they happen. GCS Buckets offer exceptional data durability, and very good performance for serving static sites like this. There's no servers to maintain, and I'm only billed for the amount of data I actually store, and the amount of traffic I actually serve. In theory, a site like this could grow to serve hundreds or thousands of contributors and millions of readers without needing any major infrastructure overhaul.