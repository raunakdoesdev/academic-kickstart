---
title: "ML Templates"
subtitle: "A project on my mind."
summary: "Musings about making templates for ML projects. "
authors: []
tags: ["python", "machine learning", "templates"]
categories: ["coding"]
date: 2021-02-05T13:24:04-07:00
lastmod: 2021-02-05T13:24:04-07:00
featured: true
draft: false

image:
  caption: ""
  focal_point: "Center"
  preview_only: true

projects: []
---

I was watching an old Thomas Frank video recently that talked about the importance of reproducible templates for many workflows: {{< youtube 01DBbTQwYIE>}}

According to the video, these kinds of templated checklists are critical in organizations from NASA to surgeons, and they’ve shown to be pretty effective. I thought a bit about my own workflow, recently, which has involved making a ton of different machine learning codebases for various ML projects I’m working on.

A ton of the code between these tools is boilerplate, stuff that’s repeated over and over. An example would be my config management system (which is pretty slick by the way – I’ll post about that another time). I end up copy pasting the same code over and over again for these kinds of basic things.

An immediate response might be “create a python library.” The thing is, these are hard to remix and edit on the fly and customize to your project. I know from my experience with my [PDFPlot library](https://sauhaarda.me/post/pdfplot/) – it’s handy but I like to remix and customize it for every project, so the library format isn't optimal. Instead, I’m thinking of going the route of something like [ cookiecutter data science](https://drivendata.github.io/cookiecutter-data-science/), and making my own cookie-cutter template for ML projects. Like a library, this can be remixed and adjusted over time, but also speeds up the process of making an ML project from scratch.
