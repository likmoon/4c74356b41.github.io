---
id: 5772
title: Moved to Github Pages and Jekyll
date: 2017-06-11T21:40:00+00:00
author: rootilo
layout: post
guid: http://4c74356b41.com/post5772
permalink: /post5772
categories:
  - random
---

I was using Project Nami and Azure SQL for my wordpress, but I decided against that as I don't need all that mess, so I moved to Github Pages and Jekyll. I now have a static webpage, less management and, theoretically, people can submit PR's to fix my stuff ;)

Process\Issues:  
1. Project Nami and Jekyll exporter do not play well together (at all), so I had to export all the data using wordpress export and import it to a fresh wordpress install ([using docker-compose](https://docs.docker.com/compose/wordpress/))
2. Installed and used Jekyll exporter plugin (which didn't exactly work, but it created temp files in `/tmp`, so I went into the container and tared them and downloaded over the http server)
3. Follow [this article](https://www.smashingmagazine.com/2014/08/build-blog-jekyll-github-pages/)
4. Find lots of small bugs\inconsistencies and fix them for several hours (and probably next several days)

And a really big issue - my blog used permalinks like this: `?p=xxxx` and Jekyll fails to use that. No ideas about how to fix this, so I worked around by searching\replacing `?p=` to `post`. Kinda works. I hope google catches up at some point. Maybe worth looking at [this](https://github.com/jekyll/jekyll-redirect-from) later.

Meanwhile, if you found this in google and it redirects you to the main page instead of the post you are looking for just do a search for the post name on the main page with `Ctrl+F`.

So I guess I'm happier now. Best of luck folks.
