---
layout: post
title:  "How to write and edit posts with Markdown, Jekyll and GitHub pages"
date:   2015-09-12 21:41:00
categories: Common
comments: true
author_name: "Dmytro Sotnyk"
author_url: "http://sotnikdv.github.io"
author_image: "http://sotnikdv.github.io/assets/images/profile.png"
author_bio: 'I`m Grid Architect at GridDynamics in San Francisco, USA. You can find me also in <a href="http://plus.google.com/109421189749606131821">Google+</a> or <a href="https://www.linkedin.com/in/sotnikdv">LinkedIn</a>.'
---

In this article we will be focused on how to write/edit/preview posts with Jekyll usage:

1. How to create Jekyll configuration
1. How to write posts in **Markdown** syntax
1. How to install and run **Jekyll** locally to get preview

We will **not** explain how to run blog on **GitHub Pages** because this is already described in [Do you wanna technical blog too?](http://sotnikdv.github.io/common/beginners/2015/03/29/do-you-wanna-blog-too.html) post.

* TOC
{:toc}


## Who is who in this mistery: Jekyll, Markdown and GitHub Pages?

Before we will start, let's overview what's **Jekyll**, **Markdown** and **GitHub Pages** and how they are connected. 


In general, [Jekyll](https://github.com/jekyll/jekyll) is a static site generator, [Markdown](http://daringfireball.net/projects/markdown/) is a "_(1) a plain text formatting syntax; and (2) a software tool, written in Perl, that converts the plain text formatting to HTML_" and [GitHub Pages](https://help.github.com/articles/what-are-github-pages/) are "_public webpages hosted and published through our site_"

So, the heart of the blog is a **Jekyll** generator which may use **Markdown** formatting syntax and tool to convert your text-formatted posts to html-pages:

1. You are writing text-formatted posts in **Markdown** syntax and **Markdown** tool will convert them to HTML.
2. Jekyll is used (and automatically invoke Markdown tool) to generate the whole site.

This is enough to build your own static site and keep it in a good shape. Nevertheless, you have to maintain VCS, back-ups, run Jekyll and upload generation results to a hosting. This is where **GitHub Pages come to play**.


_"In addition to supporting regular HTML content, GitHub Pages support [Jekyll](https://github.com/jekyll/jekyll), a simple, blog-aware static site generator. Jekyll makes it easy to create site-wide headers and footers without having to copy them across every page. It also offers [some other advanced templating features](http://jekyllrb.com/docs/templates/)."_ (c) [GitHub Pages Basics / Using Jekyll with Pages](https://help.github.com/articles/using-jekyll-with-pages/)

So, **GitHub Pages** it's a Jekyll installation which will be automatically invoked on your repository [under some conditions](https://help.github.com/articles/using-jekyll-with-pages/) and which will will be triggered on every your commit to:

- run Jekyll generation from your repository
- upload results to [name].github.io. 

**That's it**

So, we have 3 "steps":

- Step 1. Write posts in **Markdown** syntax and convert them in HTML with **Markdown** tool. You have to take care about the rest of the site, hosting etc.
- Step 2. Use Jekyll to generate site wrapper and Jekyll will invoke **Markdown** convertor for you. You have to take care about a hosting, run Jekyll manually and upload generation results
- Step 3. Use GitHub Pages and let GitHub take care about the hosting and **Jekyll** invocation. All you need is to create Jekyll configuration (or fork) and then write .markdown posts and push it to repository.


## How to create Jekyll configuration

Well, you can create your own project from scratch by `jekyll new myblogfolder` and [configure it (CSS, templates, structure)](https://jekyllrb.com/docs/templates/) or fork existing. 



I've forked this nice [Kasper theme](https://github.com/rosario/kasper) and made some changes to templates and CSS to add more information about an author, TOC etc. So, If you like congiguration of this blog, [fork it on a GitHub](https://github.com/rndblog/rndblog.github.io) and **don't forget** to [update configuration](https://github.com/rndblog/rndblog.github.io/blob/master/README.md)

## How to write posts in Markdown

Well, let's start with example. [Here is the source (text-formatted post in **Markdown** syntax)](https://raw.githubusercontent.com/rndblog/rndblog.github.io/master/_posts/2015-08-28-welcome.markdown) of ["Welcome to yet another technical blog, this time open-source!"](http://localhost:4000/common/2015/08/28/welcome.html) post from this blog.

The body of this post is in **Markdown** with some html includes (where Markdown can't do something), the header is for Jekyll (some fields from Kasper theme, some was added by me)

{% highlight text %}
---
layout: post
title:  "Welcome to yet another technical blog, this time open-source!"
date:   2015-08-28 23:50:00
update: 2015-08-29 05:30:00
categories: Common
comments: true
author_name: "Dmytro Sotnyk"
author_url: "http://sotnikdv.github.io"
author_image: "http://sotnikdv.github.io/assets/images/profile.png"
author_bio: 'I`m Grid Architect at GridDynamics in San Francisco, USA. You can find me also in <a href="http://plus.google.com/109421189749606131821">Google+</a> or <a href="https://www.linkedin.com/in/sotnikdv">LinkedIn</a>.'
---
{% endhighlight %}

### Markdown Syntax

See instructions how to write posts and list of available plugins on [Writing Posts on Jekyll](http://jekyllrb.com/docs/posts/)

Also **take a look** at the [Markdown Reference Manual](http://daringfireball.net/projects/markdown/syntax#link) and [Markdown Cheatsheet](http://assemble.io/docs/Cheatsheet-Markdown.html)  

### Markdown Extentions

If you need more markup functionality, you can use different **Markdown** convertors.

I've used [Textile](https://github.com/jekyll/jekyll-textile-converter), see [beautiful manual](http://redcloth.org/textile). Then switched to [Kramdown](http://kramdown.gettalong.org/) to get automatic ToC

So this resource currently using **Kramdown**


## How to install and run **Jekyll** locally to get preview


If you want to use your own hosting (run local development and to deploy build result to any server you need) or write article on your local system with preview, then you need local instance of  [Jekyll](http://jekyllrb.com).

### Install Jekyll

For now, we have instructions for Linux only. Please [join editors and improve this article](http://rndblog.github.io/about.html).

#### Install Jekyll on Linux

I've did this on Debian-based distributives, for other systems please re-write command to your package manager (yum etc.)

**Jekyll** is Ruby based, so you need Ruby 2.0 or later AFAIK and **ruby-dev** and compilers to install gems. 

{% highlight bash %}
apt-get install ruby-dev libz-dev git
{% endhighlight %}

Then you have to install few gems.

{% highlight bash %}
gem install jekyll
gem install bundler
gem install execjs
gem install therubyracer
{% endhighlight %}

#### Install Jekyll on Windows

#### Install Jekyll on MacOS

### Run local generation

To run build - use `jekyll build` inside a project's folder. 

So now you can upload `_site` folder  content (after build) to any server.

### Run preview

If you want to ask Jekyll to stay in background and rebuild changes files automatically - use `jekyll build -w`. This is useful to view your changes immediately
{% highlight bash %}
sdv@sdv-dev:~/code/sotnikdv.github.io$ jekyll build -w
Configuration file: /home/sdv/code/sotnikdv.github.io/_config.yml
            Source: /home/sdv/code/sotnikdv.github.io
       Destination: /home/sdv/code/sotnikdv.github.io/_site
      Generating... 
                    done.
 Auto-regeneration: enabled for '/home/sdv/code/sotnikdv.github.io'
      Regenerating: 2 file(s) changed at 2015-03-29 01:49:01 ...done in 0.04097498 seconds.
      Regenerating: 2 file(s) changed at 2015-03-29 01:49:16 ...done in 0.058890892 seconds.
      Regenerating: 2 file(s) changed at 2015-03-29 01:51:35 ...done in 0.042440558 seconds.
      Regenerating: 1 file(s) changed at 2015-03-29 01:57:30 ...done in 0.066120995 seconds.
      Regenerating: 2 file(s) changed at 2015-03-29 01:57:35 ...done in 0.04388916 seconds.
{% endhighlight %}

To start local server, use `jekyll serve` inside a folder, server will be started by default on `http://127.0.0.1:4000/`

