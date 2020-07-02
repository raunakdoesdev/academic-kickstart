---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Seeing What a Gan Cannot Generate"
subtitle: ""
summary: "This is an illustrated summary of the paper *Seeing What a GAN Cannot Generate*"
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

{{% toc %}}

[^2]: [Bau, D., Zhu, J. Y., Wulff, J., Peebles, W., Strobelt, H., Zhou, B., & Torralba, A. (2019). Seeing what a gan cannot generate. In *Proceedings of the IEEE International Conference on Computer Vision* (pp. 4502-4511).](http://ganseeing.csail.mit.edu/)



## Background

### GANs

Generative adversarial networks, or GANs, can be used to generate new images by taking advantage of the interplay between two neural networks: a generator and a discriminator. The generator is like an art counterfeiter, trying to produce realistic "fake" images, and the discriminator is like an art critic, trying to determine if a given image is real or fake.
### Mode Collapse
The problem is, the generator can get kind of lazy and start outsmarting the discriminator. Let's say your dataset has an equal number of images of people, fountains, and churches. Instead of creating fakes of all of these kinds of images, the generator might learn it's much easier to just create images of fake fountains and ignore the rest.

This is called mode collapse, because certain modes of generation (the people and the churches in this example) are no longer created by the generator. If we think of the generator as a function $G(z) $, then the range of this function (the possible values it can produce) is reduced.

It’s hard to detect when mode collapse is happening because the main metric, the discriminator, is fooled by the limited modes the generator produces.
## Prior Work
### Inception Score (IS)

One of the first papers to quantify the quality of a GAN’s generated samples was *Improved Techniques for Training GANs* [^1]. It proposes a metric, called the inception score, which optimizes two factors:

1. Overall image variety (i.e. there are many different types of images in the dataset). 
2. Individual image specificity (i.e. a particular image is distinctly one thing, not confused with other classes).

This is accomplished with the following metric: 

<video autoplay loop muted playsinline>
<source src="https://revresearch.s3.us-east-2.amazonaws.com/BlogInceptionScore.mp4" type="video/mp4"></video>

Maximizing the inception score means maximizing the KL Divergence between the distributions and improving image variety while reducing individual image specificity.

This method has some drawbacks, though. By encouraging individual image specificity, this metric penalizes scenes with multiple objects in them, as might be desired from the generative model. The inception score also operates “in a vacuum” because it only considers the distribution of generated images, ignoring the training data completely.

[^1]: Salimans, T., Goodfellow, I., Zaremba, W., Cheung, V., Radford, A., & Chen, X. (2016). Improved techniques for training gans. In *Advances in neural information processing systems* (pp. 2234-2242).

### Fréchet Inception Distance (FID)

The Fréchet Inception Distance [^frech] attempts to counter some of the shortcomings of the inception score by directly comparing the statistics of the GAN generated samples to the real samples. It also operates on an embedding extracted from an intermediate layer of the Inception Net network, rather than final predictions, allowing it to examine and optimize images in which multiple classes are present.

Here’s the equation:

 <video autoplay loop muted playsinline>
<source src="https://revresearch.s3.us-east-2.amazonaws.com/BlogInceptionDistance.mp4" type="video/mp4"></video>

Since it’s original publication, this metric has become the standard for measuring the performance of GANs. Yet, this metric also has its shortcomings, the biggest being its interpretability. The Fréchet Inception Distance operates on a “latent space” of the embeddings created Inception network, which have no inherent meaning in themselves. While it may be able to measure mode collapse, we can’t see which modes are collapsing or how much. That’s where *Seeing What a GAN Cannot Generate* comes in.

## Methodology

### Segmentation

 <video autoplay loop muted playsinline>
<source src="https://revresearch.s3.us-east-2.amazonaws.com/Segmentation.mp4" type="video/mp4"></video>

To understand what information is being dropped from generated images, it would be important to understand what “stuff” is in an image in the first place. Thankfully, this is already a field of extensive study: image segmentation. The authors use the Unified Perceptual Parsing[^unified] network for this purpose. It’s been trained to do pixel-based segmentation (that’s where you classify every pixel into some category) for a huge variety of labels. The authors then 

<iframe width="900" height="500"  frameborder="0" scrolling="no" src="//plotly.com/~sauhaarda/1.embed"></iframe>

[^unified]: Xiao, T., Liu, Y., Zhou, B., Jiang, Y., & Sun, J. (2018). Unified Perceptual Parsing for Scene Understanding. In V. Ferrari, M. Hebert, C. Sminchisescu, & Y. Weiss (Eds.), *Computer Vision – ECCV 2018* (pp. 432–448). Springer International Publishing
[^frech]: Martin Heusel, Hubert Ramsauer, Thomas Unterthiner, Bernhard Nessler, and Sepp Hochreiter. 2017. GANs trained by a two time-scale update rule converge to a local nash equilibrium. In *Proceedings of the 31st International Conference on Neural Information Processing Systems* (*NIPS’17*). Curran Associates Inc., Red Hook, NY, USA, 6629–6640.





[^ unified]: 