---
layout: post
title:  "Do you wanna technical blog too?"
date:   2015-03-29 20:00:20
update: 2015-09-12 23:22:00
categories: Common Beginners
comments: true
---

Look at [this interesting article](http://blog.vjeux.com/2011/analysis/start-a-technical-blog-its-worth-it.html), IMHO it's a good summary why you should consider to create your own blog.

* TOC
{:toc}

So let me explain what you need to create a blog. Most of you are experienced engineers, so this post is not for you, but for beginners.

You can use existing platforms like [blogspot](http://blogspot.com) or [wordpress](http://wordpress.com).

I decided to go with simple static site and external systems for comments etc. So it can be placed on [GitHub Pages][pages-gh] or any hosting. I've used [Jekyll][jekyll] as the template processor with [Kasper theme][kasper]. A bit more themes are available [here](http://jekyllthemes.org/)

This blog is available in my [GitHub](https://github.com/sotnikdv/sotnikdv.github.io) repository, feel free to fork and change to start your own blog. 

**Don't forget to:**

- delete my posts from `_posts` directory
- change `_config.yml` (use `_config.yml.sample`)
- change `about.md` (use `about.md.sample`)

### How to write posts

See this article [How to write and edit posts with Markdown, Jekyll and GitHub pages](http://rndblog.github.io/common/2015/09/12/how-to-write-posts-with-jekyll.html)

### OMG! I wanna write posts and not scripts!

No worries, now let's talk about [GitHub Pages][pages-gh]. I've used [Jekyll Now README](https://github.com/barryclark/jekyll-now/blob/master/README.md) as a basis for this section.

#### Step 1) Fork my blog repository to your User Repository

Fork `github.com/sotnikdv/sotnikdv.github.io`, then rename the repository to [yourgithubusername].github.io.

Your blog will often be viewable immediately at <http://[yourgithubusername].github.io> (if it's not, you can often force it to build by completing step 2)

#### Step 2) Customize and view your site

Change your site name, description, avatar and many other options by editing the _config.yml file.

Making a change to _config.yml (or any file in your repository) will force GitHub Pages to rebuild your site with jekyll. Your rebuilt site will be viewable a few seconds later at <http://[yourgithubusername.github.io]> - if not, give it ten minutes as GitHub suggests and it'll appear soon

> There are 3 different ways that you can make changes to your blog's files:

> 1. Edit files within your new username.github.io repository in the browser at GitHub.com (shown below).
> 2. Use a third party GitHub content editor, like [Prose by Development Seed](http://prose.io). It's optimized for use with Jekyll making markdown editing, writing drafts, and uploading images really easy.
> 3. Clone down your repository and make updates locally, then push them to your GitHub repository.

#### Step 3) Publish your first blog post

Use any existing file in `/_posts/` as a template for your new post.

> You can add additional posts in the browser on GitHub.com too! Just hit the + icon in `/_posts/` to create new content. Just make sure to include the [front-matter](http://jekyllrb.com/docs/frontmatter/) block at the top of each new blog post and make sure the post's filename is in this format: year-month-day-title.md

[pages-gh]: http://pages.github.com
[jekyll]: http://jekyllrb.com
[kasper]: https://github.com/rosario/kasper
