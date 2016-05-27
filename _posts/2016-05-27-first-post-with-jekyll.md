---
layout: post
title: First post with jekyll on github
tags: [jekyll, jekyll-compose, github]
---

[jekyll](https://jekyllrb.com) seems to be nice tool to create your own pages because it supports markdown which is developers friend.

# Install jekyll

```bash
sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt-get update
sudo apt-get install ruby2.3 ruby2.3-dev

sudo gem install jekyll
```
The dependencies node.js and python were already insalled on my system.

# Create new blog

```bash
jekyll new blog
cd blog
jekyll serve
# open http://127.0.0.1:4000/ in browser
```

# Create new post using jekyll-compose

[jekyll-compose](https://github.com/jekyll/jekyll-compose) is great to create new posts or pages.

```bash
sudo gem install jekyll-compose
sudo gem install bundler
bundle init # to create new Gemfile
```

Add `gem 'jekyll-compose', group: [:jekyll_plugins]` to your `Gemfile`.

After this you can create your first post very easily.

```bash
bundle exec jekyll post "First post with jekyll"
```

# Make it nice by using a theme

Found this nice theme [dbyll](http://dbtek.github.io/dbyll/) on [http://jekyllthemes.io/](jekyllthemes.io).

To use it I have to fork it and execute the commands on the page and copy my existing `_posts` and `config.yml` files.

# Upload to github

First create new `blog` github repository

```bash
git clone https://github.com/ccguyka/blog.git
# add all files to repository
git add .
git commit -m "some message"
git push --set-upstream origin gh-pages
```
After that you have to wait some time until it's published on github.io.
