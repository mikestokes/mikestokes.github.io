---
layout: post
title: Creating-a-Light-Weight-Blog-with-Github-Pages-and-Jekyll
comments: True
category: blogging
tags: [github-pages, jekyll, blogging, markdown blogging]
---


As a developer I find the typical flow ant tools used for blogging to be less than ideal. I want a flow that embraces the tools I use on a daily basis and makes it as easy as possible to blog...

> What I want is an optimized blogging platform and experience targetted to developers.

## What about Wordpress?

I could have used [Wordpress](https://wordpress.org/) and [WP Engine](http://wpengine.com/), an excellent solution to be sure. But, it doesn't really tick the developer flow or simplicity requirement.

## GitHub to the Rescue

Github has a great feature called **Github Pages** that allows us devs to host static content as a web site - they even allow us to associate a custom domain name (e.g. example.com) with the web site. Could this be suitable?

GitHub Pages offers the following capabilities that are relevant to us:

 * Pre-processing and templating support via Jekyll
 * Quick start blogging templates like [JekyllNow](https://github.com/barryclark/jekyll-now) and [Poole](http://getpoole.com/) (this blog is based on Poole / [Hyde](https://github.com/poole/hyde))
 * Markdown support as the blogging format
 * Git commit publishing (even publish directly from GitHub)
 * Custom domain names can be pointed to your static pages
 * Supports commenting, categories, tags etc easily or with little change
 * Easy to set up and get going with
 * It's free and fast!

### Building the GitHub Pages Blog

So with this information, I decided to MVP blog using GitHub Pages and see how it went.

I decided to use the Hyde repository as a starting point as it looked quite beautiful and was simple to understand and enhance.

#### Cloning the Base Repository

If you don't already have a GitHub account, jump over there now and [create one for free](https://github.com/).

Once you're logged into your GitHub account, the quickest way to get started is to clone an existing Jekyll repository to modify. As stated, I decided to use Hyde, so to do the same, visit the [Hyde repository](https://github.com/poole/hyde) and click the Fork button ![Hyde Repository Fork](http://mikestokes.co/public/img/fork-hype-repo.png)

> There are other Jekyll repos you can clone to get started quickly. Check out [JekyllNow](https://github.com/barryclark/jekyll-now) and [Poole](https://github.com/poole/) for tsraters.

#### Rename your Cloned Repository

This step is critical! It's also easy :)

In the cloned repository, click the Settings button and then rename the repository so it has exactly the same name as your GitHub home. For me that's "mikestokes.github.com", for you, it'll follow the pattern "yourgithubname.github.io".

![Hyde Repository Fork](http://mikestokes.co/public/img/rename-repo.png)

#### Visit your Cloned Blog

Your should now be able to visit your blog website at "yourgithubname.github.io".

![Hyde Repository Fork](http://mikestokes.co/public/img/hype-cloned-blog.png)

## Customising your Blog

Now that you have a static blog, we'll customise it over the next few blog posts.

Things we'll be looking at customising:

 * Title, blog description and other basics (hint: look at the _config.yml file)
 * Links in the left hand navigation (hint: sidebar.html and head.html templates)
 * Adding new posts (/posts folder)
 * Adding Disqus comments to your blog posts
 * Changing the Favicon (look in the /public folder)
 * Adding Google Analytics (put it in the default.html template)
 * Ideal Mac and Windows setups (I use [Hooroopad](http://pad.haroopress.com/) on Windows)
