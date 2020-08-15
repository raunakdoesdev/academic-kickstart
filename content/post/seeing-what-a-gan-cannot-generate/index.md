---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Seeing What a Gan Cannot Generate"
subtitle: ""
summary: "This is an illustrated summary of the paper *Seeing What a GAN Cannot Generate*"
authors: ['admin']
tags: []
categories: []
date: 2020-08-14T13:24:04-07:00
lastmod: 2020-08-14T13:24:04-07:00
featured: true
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: "Smart"
  preview_only: true

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---


{{% toc %}}

[^2]: [Bau, D., Zhu, J. Y., Wulff, J., Peebles, W., Strobelt, H., Zhou, B., & Torralba, A. (2019). Seeing what a gan cannot generate. In *Proceedings of the IEEE International Conference on Computer Vision* (pp. 4502-4511).](http://ganseeing.csail.mit.edu/)

## Intro
This is a summary of the research paper, *Seeing What a GAN Cannot Generate*.[^2] The authors of this paper have full rights over this work. I’ll be breaking down their paper for a more lay audience in this post.

### Video Overview
As an experiment, this blog post was adapted into a video format, with narration accompanied by live annotation. Feedback is welcome.
{{< youtube J9k83wShQ7c >}}

## Background

<video autoplay loop muted playsinline>
<source src="https://revresearch.s3.us-east-2.amazonaws.com/Background.mp4" type="video/mp4"></video>

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

[^1]: [Salimans, T., Goodfellow, I., Zaremba, W., Cheung, V., Radford, A., & Chen, X. (2016). Improved techniques for training gans. In *Advances in neural information processing systems* (pp. 2234-2242).](https://papers.nips.cc/paper/6124-improved-techniques-for-training-gans)

### Fréchet Inception Distance (FID)

The Fréchet Inception Distance [^frech] attempts to counter some of the shortcomings of the inception score by directly comparing the statistics of the GAN generated samples to the real samples. It also operates on an embedding extracted from an intermediate layer of the Inception Net network, rather than final predictions, allowing it to examine and optimize images in which multiple classes are present.

Here’s the equation:

 <video autoplay loop muted playsinline>
<source src="https://revresearch.s3.us-east-2.amazonaws.com/BlogInceptionDistance.mp4" type="video/mp4"></video>

Since its original publication, this metric has become the standard for measuring the performance of GANs. Yet, this metric also has its shortcomings, the biggest being its interpretability. The Fréchet Inception Distance operates on a “latent space” of the embeddings created Inception network, which have no inherent meaning in themselves. While it may be able to measure mode collapse, we can’t see which modes are collapsing or how much. That’s where *Seeing What a GAN Cannot Generate* comes in.



## Quantifying Mode Collapse with Segmentation

 <video autoplay loop muted playsinline>
<source src="https://revresearch.s3.us-east-2.amazonaws.com/Segmentation.mp4" type="video/mp4"></video>

To understand what information is being dropped from generated images, it would be important to understand what “stuff” is in an image in the first place. Thankfully, this is already a field of extensive study: image segmentation. The authors use the Unified Perceptual Parsing[^unified] network for this purpose. It’s been trained to do pixel-based segmentation (that’s where you classify every pixel into some category) for a huge variety of labels. 

When you examine the differences in the distribution of pixels across these labels for both the real and generated image sets, you get the following plot[^ reldiff]: 

<div class='embed-container'><iframe frameborder='0' scrolling='no' src='//plotly.com/~sauhaarda/1.embed'></iframe></div>
**By plotting against segmented categories, the authors are able to show exactly which modes the GAN system is dropping in its computation and which modes it’s overrepresenting to compensate.** More difficult classes like hands and people are underrepresented in generated images, while simple textures like rocks, ceiling, and floor are overrepresented.

The authors go on to propose a new metric, called **Fréchet Segmentation Distance (FID)** for quantifying the problem of mode collapse. It’s exactly analogous to the Fréchet Inception Distance (FID) score (uses the same equation), but replaces the uninterpretable Inception embedding with the pixel distribution across the Unified Perceptual Parsing system’s labels. This confers two advantages:

1. The distribution being analyzed has a human-interpretable meaning.
2. The system can handle multiple classes within one image, and better quantify the problem of mode collapse.

## Visualizing Mode Collapse in Individual Images

{{< figure src="ablation-01.png" caption="Approach Comparison" lightbox="true" >}}

In the next section, the authors propose a method of visualizing mode collapse for individual images.  If we want to find out what the GAN can’t generate, one thing we can do is to try to get it to replicate an image. The parts of the image it can’t replicate will correspond with the modes that have collapsed.

Before exploring this concept, it’s important to take a closer look at one element of how a generative adversarial system works. In order to generate random images, the generator acts as a function which takes in a source of entropy, or randomness, denoted $z$. For random values of $z$, the network will produce completely different images, but a specific input $z$ will always produce the same result. In this way, getting the GAN to replicate an image can be reduced to the problem of finding the best $z$, which when fed into the generator, produces a result closest to the desired image (minus any mode collapse of course).

The authors tried six different ways of approaching this problem of finding the right $z$:

{{< figure src="ablation-01.png" lightbox="true" >}}

1. **Baseline - Optimize $\textbf{z}$** - this is by far the simplest method, and it involves starting with a random $z$ and slowly tweaking it to get the output of the generator to look more like the desired image. This tweaking process is called *gradient descent*. Unfortunately, the neural networks commonly used as generators are complex and have too many parameters.  With this approach, the generated images are very different from the “goal” image the authors attempt to replicate, as seen in the figure above.
2. **Baseline - Learn $\textbf{E}$ directly** - Another approach is to train a deep neural network, E, to take an image as input and output a $z$, which, when plugged into the generator, creates a similar image. This method does slightly better than the direct optimization because it uses a deep neural network to learn more generalized rules, but the results are still pretty poor.
3. **Baseline - direct $\textbf{E}$ then $\textbf{z}$** - A different approach[^savage] is to use the trained $E$ from #2 to generate an initial guess for $z$ . This initial guess can then be optimized further using the gradient descent described in #1. Again, this does better, but to quote the authors, “the reconstructions are far from satisfactory.” Savage!
4. **Ablation - layered $E$ alone** - This is where we get into the authors’ novel contributions. Instead of training the entire $E$ deep neural network at once, they train the layers of $E$ separately to invert each individual layer of $G$. Once this is done, they assemble these separately trained layers of $E$ and finetune them to invert $G$ as a whole. This layer-wise approach reduces the number of parameters being optimized at each step and significantly improves the overall quality the outputted $z$.
5. **Ablation - layered $E$ alone** - Analogous to #3, this approach uses the trained E from #4 to generate an initial guess for $z$ and optimizes it further using gradient descent. This marginally improves the prediction.
6.  **Final - layered $E$ then $r$** - This is the final approach the authors use. Instead of directly taking the outputted $z$ and sticking it into the generator network, the authors split the generator, $G$ into two halves.  Plugging $z$ into the first half, they end up with an intermediate output, $r$. They then run gradient descent to optimize $r$ in order to make the image outputted by the second half of the generator closer to the desired image. The key point here, though, is that the authors are solving a simpler subproblem with this approach. <u>They are identifying the mode collapse in the second half of the generator G, claiming that this is an approximation of the mode collapse in the generator as a whole.</u>

With this method, we are finally able to “See What a GAN Cannot Generate.” The authors run their visualization algorithm on two standard datasets: LSUN bedrooms and LSUN churches. They also visualize their trained models with unrelated images to confirm that unseen objects are dropped in the final visualization. Not only are they dropped, the network often converts one mode to another. In one case, a dining room table with a white tablecloth is turned into a bed with a white bed sheet.

{{< figure src="results-01.png" lightbox="true" >}}

## Conclusion

To sum it all up, the authors provide two ways to measure mode collapse:

1. Identify which modes are collapsing by quantifying the relative difference in the distribution of segmented classes over real and generated images.
2. Visualize the mode drop by inverting the generator, revealing exactly which part of a given image the GAN cannot generate.

Notably, the authors don’t attempt to solve the problem of mode collapse, just characterize it. If you’d like to know how mode collapse can mitigated, look into recent techniques such as Veegan[^veegan] and PresGAN[^presgan].

[^savage]: [Zhu, J. Y., Krähenbühl, P., Shechtman, E., & Efros, A. A. (2016, October). Generative visual manipulation on the natural image manifold. In *European conference on computer vision* (pp. 597-613). Springer, Cham.](https://link.springer.com/chapter/10.1007/978-3-319-46454-1_36)
[^unified]: [Xiao, T., Liu, Y., Zhou, B., Jiang, Y., & Sun, J. (2018). Unified Perceptual Parsing for Scene Understanding. In V. Ferrari, M. Hebert, C. Sminchisescu, & Y. Weiss (Eds.), *Computer Vision – ECCV 2018* (pp. 432–448). Springer International Publishing.](https://openaccess.thecvf.com/content_ECCV_2018/papers/Tete_Xiao_Unified_Perceptual_Parsing_ECCV_2018_paper.pdf)
[^frech]: [Heusel, M., Ramsauer, H., Unterthiner, T., Nessler, B., & Hochreiter, S. (2017). Gans trained by a two time-scale update rule converge to a local nash equilibrium. In *Advances in neural information processing systems* (pp. 6626-6637).](https://dl.acm.org/doi/10.5555/3295222.3295408)

[^ reldiff]: The data in this figure were extracted from http://ganseeing.csail.mit.edu/ using WebPlotDigitizer, and then replotted. Due to this process, there may be some noise in the relative difference values.

[^presgan]: [Dieng, A. B., Ruiz, F. J., Blei, D. M., & Titsias, M. K. (2019). Prescribed generative adversarial networks. *arXiv preprint arXiv:1910.04302*.](https://arxiv.org/abs/1910.04302)
[^veegan]: [Srivastava, A., Valkov, L., Russell, C., Gutmann, M. U., & Sutton, C. (2017). Veegan: Reducing mode collapse in gans using implicit variational learning. In *Advances in Neural Information Processing Systems* (pp. 3308-3318).](http://papers.nips.cc/paper/6923-veegan-reducing-mode-collapse-in-gans-using-implicit-variational-learning)
