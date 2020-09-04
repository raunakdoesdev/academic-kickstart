---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Segmenting Aerial LIDAR Data"
subtitle: ""
summary: "I segment some LIDAR data from an interview with MIT's ESI Group."
authors: ['admin']
tags: []
categories: []
date: 2020-09-03T13:24:04-07:00
lastmod: 2020-09-03T13:24:04-07:00
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


This is a short notebook outlining my procedure on the "interview project" for the MIT ESI's Nature Based Solutions group to train a model to segment aerial lidar data.

# Data Processing

### External (QGIS3) Work

There was only one provided image of training data, which was a subset of the larger test data image. I crop the "test" image to include only the portion where label data was available using **QGIS3** and load this image alongside the label data, GRSS_DFC_GT_TR.tif. I saved this to my google drive so I can easily load it within colab.


### Split Training and Validation.


```python
img = open_image('/content/drive/My Drive/geotiff/training_data/cropped_training_data.tif')
labels = open_mask('/content/drive/My Drive/geotiff/GRSS_DFC_GT_TR.tif')
img.show(y=labels, alpha=0.5, figsize=(25, 25))
```

{{< figure src="featured.png">}}


The data is pretty uniformly distributed vertically. To get a roughly uniform distribution of classes between training and validation, I decide to have the validation set be the bottom 1/10 of the image. **This is DEFINITELY not the best approach to take, but it is simple to implement. In the future, splitting the image into grids and then sampling the grids to ensure the training and validation have similar distributions is the way to go.**


```python
plt.style.use(['dark_background'])

img_data = img.data
c, h, w = img_data.shape
train_size = round(0.9 *h)
train_img = Image(img.data[:, :train_size, :])
val_img = Image(img.data[:, train_size:, :])
train_labels = ImageSegment(labels.data[:, :train_size, :])
val_labels = ImageSegment(labels.data[:, train_size:, :])

_,axs = plt.subplots(2,2, figsize=(15,8))
train_img.show(ax=axs[0][0], y=train_labels, title='Training Set')
val_img.show(ax=axs[1][0], y=val_labels, title='Validation Set')

axs[0][1].hist(train_labels.data.flatten())
axs[0][1].set_title('Training Label Distribution')
axs[0][1].set_xticks(list(range(21)))
axs[0][1].ticklabel_format(style='plain') # disable scientific notation

axs[1][1].hist(val_labels.data.flatten())
axs[1][1].set_title('Validation Label Distribution')
axs[1][1].set_xticks(list(range(21)))
```

   
{{< figure src="Segmenting_Aerial_LIDAR_6_1.png">}}


As you can see, the distribution of labels in the training and validation sets is at least roughly similar, though it can definitely be optimized with some smarter methods in the future.

### Save Data


```python
!mkdir -p images
train_img.save('images/train.png')
val_img.save('images/val.png')
!mkdir -p labels
train_labels.save('labels/train.png')
train_labels.save('labels/val.png')

codes = [str(i) for i in range(21)] # label ids, since these were not provided in the training set
```

### Load Data


```python
get_label_image = lambda x: f'labels/{x.stem}{x.suffix}'
src = (SegmentationItemList
       .from_folder('images')
       .split_by_files(ItemList(['val.png']))
       .label_from_func(get_label_image,  classes=codes))
```

### Transformations

I encode random transformations such as vertical and horizontal flipping as well as random cropping of the image to a 200x200 size. I also normalize the input data to match the imagenet statistics, as I'm going to use a ImageNet pretrained Resnet for the UNet segmentation method.



```python
data = (src.transform(get_transforms(), size=(200,200),
        tfm_y=True, resize_method=ResizeMethod.CROP)
data = data.databunch(bs=1, num_workers=0).normalize(imagenet_stats))
```
    

### Metrics

Here I defined the acc_camvid metric:

$$\text{Accuracy}_\text{camvid} = \dfrac{\text{# of Correctly Classified Pixels}}{\text{Total # of Pixels}}$$

This snippet was pasted from: https://towardsdatascience.com/image-segmentation-with-fastai-9f8883cc5b53


```python
name2id = {v:k for k,v in enumerate(codes)}
void_code = 3000

def acc_camvid(input, target):
    target = target.squeeze(1)
    mask = target != void_code
    return (input.argmax(dim=1)[mask]==target[mask]).float().mean()
```

# Training

### Define Model
Here I define the model to use a pretrained Resnet34 and initialize some basic parameters.


```python
metrics=acc_camvid
wd=1e-2
```


```python
learn = unet_learner(data, models.resnet34, metrics=metrics, wd=wd)
```
    

### Auto-Tune Learning Rate
First introduced by Leslie N. Smith in [Cyclical Learning Rates for Training Neural Networks](https://arxiv.org/pdf/1506.01186.pdf), the LR Finder trains the model with exponentially growing learning rates from start_lr to end_lr for num_it and stops in case of divergence (unless stop_div=False) then plots the losses vs the learning rates with a log scale.

A good value for the learning rates is then either:
* one tenth of the minimum before the divergence
* when the slope is the steepest


```python
lr_find(learn)
learn.recorder.plot()
```
{{< figure src="Segmenting_Aerial_LIDAR_19_3.png">}}


### Training Loop

```python
learn.fit_one_cycle(2000, slice(1e-06,1e-03), pct_start=0.9)
```

# Validation
Due to VRAM limitations on Google Colab, I cannot run the model on the full input images. This would be easily possible with access to more VRAM or even RAM if you wanted run the model on a CPU.

Instead, I opt to split the image into smaller grids and infer on each of these images separately. Then I concatenate all of the images back together.

Here I do this for the strip of validation and evaluate the model accuracy.


```python
learn.data.single_dl.dataset.tfmargs['size'] = None
out = learn.predict(val_img)
size = 2000
val_grid = np.array_split(val_img.data, round(im.shape[2]/2000), axis=2)
val_grid = [learn.predict(Image(x))[0].data for x in val_grid]
val_result = torch.cat(val_grid, axis=2)
print(f'Validation Accuracy = {(val_result==val_labels.data).float().mean()*100.0:.2f}%')
```

    Validation Accuracy = 73.67%
    

# Testing
Now, we qualitatively test on the unlableled test dataset (which also includes the trainining/validation dataset). We use the same gridding method here.

### Splitting 


```python
size = 500
im = open_image('/content/drive/My Drive/geotiff/test_data.tif').data

grid = np.array_split(im, round(im.shape[1]/size), axis=1)
grid = [np.array_split(row, round(im.shape[2]/size), axis=2) for row in grid]
```

### Inference


```python
for i in range(len(rows)):
    for j in range(len(rows[0])):
        grid[i][j] = learn.predict(Image(grid[i][j]))
```    

### Concatenation


```python
result = []
for i in range(len(rows)):
    result.append(torch.cat([grid[i][j][0].data for j in range(len(grid[i]))], dim=2))
result = torch.cat(result, dim=1)
```

# Final Result


```python
im = open_image('/content/drive/My Drive/geotiff/test_data.tif').resize((1,2500,8500))
im.show(y=ImageSegment(result), alpha=0.75, figsize=(50, 50))
```


{{< figure src="Segmenting_Aerial_LIDAR_34_0.png">}}


### Compare Distributions


```python
_,ax = plt.subplots(1,2, figsize=(20,6))
ax[0].hist(labels.data.flatten())
ax[0].set_xticks(list(range(21)))
ax[0].ticklabel_format(style='plain')
ax[0].set_title('Train/Validation Label Distribution')

ax[1].hist(result.data.flatten())
ax[1].set_xticks(list(range(21)))
ax[1].ticklabel_format(style='plain')
ax[1].set_title('Output Label Distribution')
```

{{< figure src="Segmenting_Aerial_LIDAR_36_1.png">}}


# Conclusion - Reflection - Future Work
- It's clear that because of the **unequal distribution** of the data provided, the model ignores some of the classes that are represented less often. This could be addressed by changing the weighting/importance of these labels in the loss function. Given time constraints, I will leave this as future work.
- Many of the buildings to the right of the image, which were not included in the training data are not properly identified. Doing more data augmentation/increasing the size of the training set could improve these results.
- I accidentally trained the model to take in three input channels (the image has 3 when loaded) but each channel is the same. This is a total waste, so in the next iteration I would retrain it without the extra channels.
- I doubled the number of training examples (from 1000 --> 2000) and the performance increased significantly. I'm pretty confident that increasing it more would improve accuracy, so I leave this for future work as well.

I did all of the coding for this project in one one evening. Because of this, I didn't have a chance to experiment with the other provided datasets or fine-tune the model as much as I would have liked. This was unavoidable however, as I needed time to adjust to my first two days of classes @ MIT. I'm pretty happy with the project overall.
