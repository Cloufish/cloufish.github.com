---
title: How to create your own blog - Jekyll
date: 2020-05-27 06:46:00 +0100
categories: [Blogging, Jekyll]
tags: [blog, growth, jekyll, free]
lang: en
---

**In this post, I will cover how to create a blog. We will use the Jekyll static page generator for this. This is a free option and much more efficient, safer. than CMS (including WordPress). However, this post will not go into detail on how Jekyll works or how to code your own page theme**


# Why do we use a static page generator for this?

| Advantages | Disadvantages |
| - You are able to use Markdown (the language for formatting text) | - There are difficulties in running a blog with many authors |
| - It's quick to configure | |
| - Does not require huge VPS resources. | |
| - It is much safer due to the lack of a database. | |
| - It's free | |
| - You are able to edit the page code (if you can) | |

That it wasn't. Creating a blog using CMS (WordPress or Drupal) is not a bad thing. However, I am presenting a way more budgetary here.

## STEP 1 - Choosing the theme of our website

Here are a few pages to help us with this:
- https://jamstackthemes.dev/ssg/jekyll/
- http://jekyllthemes.org/
- https://jekyllthemes.io/
- https://jekyllthemes.io/

Some themes are paid on these sites, many of them are free. In the search engine, we are able to enter keywords that are more interesting in the appearance of the page.
However, there are many more factors to consider than just whether the look * "I like" *:

We should check the theme design on GitHub too:
- See if the creator is responding to * issues *
- whether the configuration files are readable.
I say this because most of the creators of these themes are Chinese and not all of them make a blog for international users, and can only do for China.

When we manage to choose a theme, we enter its project in GitHub and fork it to our account.

Now it remains for us to clone the project from our repository
``` git clone https://github.com/<our-username>/<our-repository>.git ```

## STEP 2 - Get familiar with the file structure

```
├── _date
├── _includes
├── _layouts
├── _posts # Place for our posts.
├── _scripts
├── assets
├── tabs
│ └── about.md # "About me" page
├── .gitignore
├── 404.html
├── Gemfile
├── LICENSE
├── README.md
├── _config.yml # Configuration file
├── docs
├── feed.xml
├── index.html
├── robots.txt
└── sitemap.xml

``

It should be mentioned right away that this is an example of the file structure I work on. It doesn't have to match 100% of yours. However, these folders with the comment next to them must always exist, and most of them will be the ones we will work on.

## STEP 3 - Supplementing the configuration file config.yml


There is no deeper philosophy here. We just need to complete it. Each creator here creates a differently arranged file, but the variables also remain unchanged.

## STEP 4 - Local site hosting

### Installing Jekyll

In general, we should always host the site locally first. Finally, I have to bring up the Jekyll installation. Follow the instructions in Jekyll's documentation https://jekyllrb.com/docs/installation/

### Alternative - Docker container with Jekyll

Installing Jekyll and its dependencies are sometimes troublesome. Containerization will solve this problem for us.
For this, I assume you already have Docker installed. To containerize Jekyll, go to the main theme folder and enter the command
``` bash
docker run --rm --volume = "$ PWD: / srv / jekyll" -p 4000: 4000 jekyll / jekyll: 4.0 jekyll serve
```
> --rm - removes the container when we are done hosting locally
> --volume = "$ PWD: / srv / jekyll" - We put the contents of the directory we are in to the directory / srv / jekyll where we expect files.
> -p 4000: 4000 - We open the port 4000 through which we can connect to the site
> jekyll / jekyll: 4.0 - define the application image here. The value after the colon represents the version (we should never use the newest)
> jekyll serve - Specify the command to be executed after starting the container.

And that's practically it. However, it would be troublesome to type this command every time. Docker-compose is the solution! For this we create the docker-compose.yml file and give it the content:
``` yaml
# docker run --rm --volume = "$ PWD: / srv / jekyll" -p 4000: 4000 jekyll / jekyll: 4.0 jekyll serve

version: '3'
services:
  jekyll serve:
    image: jekyll / jekyll: 4.0
    volumes:
      - '.: / srv / jekyll'
    ports:
      - 4000: 4000
    command: 'jekyll serve'

```
Now, thanks to this, we can turn the application on and off with one command :)
``` bash
 docker-compose up
```
`` bash
 docker-compose down
```

## Writing your first post

 Most of your posts will end up in the _posts folder. However, the text file must meet several conditions:
 - Its name must begin with a date in the American syntax, eg "2020-05-31". So the final name must be YEAR-MONTH-DAY-NAME-OF YOUR-FAST.md
 - Must have so-called "front matter". You can read about it here -> https://jekyllrb.com/docs/step-by-step/03-front-matter/
  - And of course, we write its content in markdown. It is not as difficult a matter as it seems. It is even more convenient;)

  **That's it for the first part about creating a blog. In the next one I will try to discuss how to host your blog on GitHub Pages - also free, non-scary hosting. I encourage you to review on your own and complete your knowledge:
  - Technical documentation [ENG]
  - Youtube course from giraffe.academy [ENG] https://www.youtube.com/watch?v=T1itpPvFWHI&list=PLLAZ4kZ9dFpOPV5C5Ay0pHaa0RJFhcmcB**
