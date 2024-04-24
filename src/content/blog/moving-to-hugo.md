---
author: Ryan Jordan
pubDatetime: 2017-02-09T21:27:00Z
title: Moving to Hugo
slug: moving-to-hugo
draft: false
tags:
  - hugo
  - wercker
description: I have decided to move my blog from [Ghost](https://ghost.org/) to [Hugo](https://gohugo.io/). I have enjoyed using Ghost, but I wanted to play around with some static site generators and host it with [GitHub Pages](https://pages.github.com/). I have been using a Ghost, [Docker](http://docker.io) and [Digital Ocean](http://digitalocean.com) setup and have enjoyed it, but it was time to break something.
---

I have decided to move my blog from [Ghost](https://ghost.org/) to [Hugo](https://gohugo.io/). I have enjoyed using Ghost, but I wanted to play around with some static site generators and host it with [GitHub Pages](https://pages.github.com/). I have been using a Ghost, [Docker](http://docker.io) and [Digital Ocean](http://digitalocean.com) setup and have enjoyed it, but it was time to break something.

## Hugo

After doing some research and playing around with a few different static site generators ([Cryogen](http://cryogenweb.org/), [GatsbyJS](https://github.com/gatsbyjs/gatsby), [Hexo](https://hexo.io) and [Phenomic](https://phenomic.io), to name a few), I decided Hugo worked best for my needs. It is easy to get set up and there are quite a few good looking [themes](http://themes.gohugo.io/). I was up and running quickly and was able to copy and paste my blog content over from my Ghost blog. I haven't been writing much, therefore I didn't have much to copy over, but if you have a lot of content, they have a few [tools](https://gohugo.io/tools/) that can help ease the pain of transferring your content.

Once I had my blog looking fancy, I decided it was time to get it up and running with GitHub Pages. This is where I ran into some hiccups. Of course, an easy solution would be to checkout my <code class="language-none">gh-pages</code> branch and run <code class="language-none">hugo \-\-themes=MY_THEME</code>, then delete everything except the <code class="language-none">/public</code> directory. I don't like this option and would prefer to just have it deploy when I do a <code class="language-git">git push</code>. So I opened up my trusty browser and started searching for a better solution.

## Wercker

I briefly tried using [Travis CI](https://travis-ci.org/), but I couldn't figure out how to get it to work properly. I have very limited experience with continuous integration tools and this was probably my downfall. However, [Wercker](http://www.wercker.com/) was fairly straightforward once I started figuring things out. I have two pipelines set up to build one after another once I push to the <code class="language-none">master</code> branch. The first pipeline builds the hugo site and the second deploys it to my <code class="language-none">gh-pages</code> branch. I have forked the theme I'm using and made a few small changes to it to suit my needs. So during my build process, I pull that from my forked repository.

### Setting Up Wercker

I created a new application in [Wercker](http://www.wercker.com/) and linked it to my repository that houses the code for my Hugo blog. The first pipeline I created was a build pipeline. I added a <code class="language-none">$GIT_TOKEN</code> environment variable to both the build and deploy pipelines. You will need to generate a token in your [GitHub Developer Settings](https://github.com/settings/tokens) and add it as the value in Wercker. Make sure to check the <code class="language-none">repo</code> checkbox. In the <code class="language-none">Workflows</code> tab, I clicked the <code class="language-none">Add new pipeline</code> and added a deploy pipeline. Then I chained them together by clicking the <code class="language-none">+</code> next to the build pipeline and added my deploy pipeline.

I'm using the <code class="language-none">arjen/hugo-build</code> and <code class="language-none">leipert/git-push</code> scripts to help out with building and deploying my blog. My <code class="language-none">wercker.yml</code> file is as follows:

```yaml
box: debian
build:
	steps:
		- script:
			name: install git
			code: |
				apt-get update
				apt-get -y install git
		- script:
			name: download theme
			code: |
				git clone https://github.com/krjordan/hugo-redlounge.git themes/hugo-redlounge
				rm -rf themes/hugo-redlounge/.git
		- arjen/hugo-build@1.14.1:
			version: "0.18.1" 			# Hugo version
			theme: hugo-redlounge
			disable_pygments: true		# Disable pygments because I'm using Prism.js

deploy:
	steps:
		- script:
			name: install git
			code: |
				apt-get update && apt-get install git -y
		- leipert/git-push:
			gh_oauth: $GIT_TOKEN
			basedir: public
			clean_removed_files: true
			branch: gh-pages
			repo: krjordan/personal-blog
			gh_pages_domain: mycodingblog.com    # Optional Custom Domain
```

## Triggering the Build

Once this is set up, it should automatically build and push the <code class="language-none">/public</code> directory contents to the <code class="language-none">gh-pages</code> branch. It should also set up a CNAME file with the domain name specified with the <code class="language-none">gh_pages_domain</code> variable.

## Conclusion

I'm happy with the overall setup. Setting up Hugo was fairly quick and easy. Deploying it, not so much, but I was finally able to get everything up and running like I wanted. So, I broke some things, fixed some things and got a new theme for my blog.
