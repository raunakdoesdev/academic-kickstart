---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Seeing What a Gan Cannot Generate"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-06-28T13:24:04-07:00
lastmod: 2020-06-28T13:24:04-07:00
featured: true
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

## GANs
Generative adversarial networks, or GANs, can be used to generate new images by taking advantage of the interplay between two neural networks: a generator and a discriminator. The generator is like an art counterfeiter, trying to produce realistic "fake" images, and the discriminator is like an art critic, trying to determine if a given image is real or fake.
## Mode Collapse
The problem is, the generator can get kind of lazy and start outsmarting the discriminator. Let's say your dataset has an equal number of images of people, fountains, and churches. Instead of creating fakes of all of these kinds of images, the generator might learn it's much easier to just create images of fake fountains and ignore the rest.

This is called mode collapse, because certain modes of generation (the people and the churches in this example) are no longer created by the generator. If we think of the generator as a function $$ G(x) $$, then the range of this function (the possible values it can produce) is reduced.

This is a real problem, because it's hard to detect when mode collapse is happening since the main metric, the discriminator, is fooled by the limited modes the generator produces.
## Inception Score and Fréchet Inception Distance
One of the first papers to quantify the problem of mode collapse was *Improved Techniques for Training GANs* [^1]. It proposes a metric which optimizes two factors:
1. Overall image variety (i.e. there are many different types of images in the dataset)
2. Individual image specificity (i.e. a particular image is distinctly one thing, not confused with other classes)

This is accomplished with the following metric:
$$\operatorname{IS}(G)=\exp \left(\mathbb{E}_{\mathbf{x} \sim p_{g}} D_{K L}(p(y \vert \mathbf{x}) \| p(y))\right)$$

{{< video library="1" src="https://revresearch.s3.us-east-2.amazonaws.com/InceptionScore.mp4" controls="yes" >}}

Here's an equation breakdown: 

[^1]: Salimans, T., Goodfellow, I., Zaremba, W., Cheung, V., Radford, A., & Chen, X. (2016). Improved techniques for training gans. In *Advances in neural information processing systems* (pp. 2234-2242).

