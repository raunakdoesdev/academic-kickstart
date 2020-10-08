---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "COVID Danger Meter"
subtitle: "Use your head and stop the spread!"
summary: "Project submission for HackMIT 2020."
authors: ['admin']
tags: []
categories: []
date: 2020-09-20T13:24:04-07:00
lastmod: 2020-09-20T13:24:04-07:00
featured: true
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: "Smart"
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

This will be a short post! I wanted to share the project my team (Sarah Zhao, [Mitali Chowdhury](https://www.linkedin.com/in/mitali-chowdhury-ba6b311b8/), and I) completed in ~24 hours for HackMIT 2020. It's called COVID-Meter.

## Source Code
https://github.com/sauhaardac/covid-danger

## Project
This project is our submission for HackMIT 2020.

We developed a web application to analyze crowdsourced, geotagged images to determine whether people are following COVID-19 public safety guidelines at different locations. Based on metrics detected from the image including how many people are wearing masks, the app provides a score for how safe a location is. The map view shows the distribution of where there is the highest risk of spreading COVID-19, and provides an intuitive visualization for infraction hotspots. This helps better inform both the public and law enforcement of public health risks. 

## Technology Stack
The image processing aspect of this project uses PyTorch FasterRCNN to find masks, TensorFlow implementation of YOLO to identify faces (pretrained), and OpenCV. We then pinpoint overlap between faces and masks as identified by the artifial intelligence algorithms and use this, along with the environment, to generate a COVID safety score. In the future, this safety score will be written to a database along with the user's latitude and longitude.

Using the data stored in the database, we also have a visualization component to help users figure out whether a location is safe. There is both a clustered leaflet plot to show the scores of individual locations, built with Folium, while the overall concentration of infractions can be seen in the 3-D bar graph, which uses PyDeck.

Finally, the web app for the project was created with a StreamLit backend. 

## Instructions

1. Upload a picture of a specific location in order to update the COVID Danger Scale application.
2. View the aggregated statistics on a map of your area in the View menu.

## Video Demo
{{< youtube p3TMY83KXPY>}}
