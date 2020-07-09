---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "PDFPlot"
subtitle: "Organize your plots in one line."
summary: "A tour of my first python package designed to help people better organize plots for their big projects."
authors: []
tags: ["python", "plotting"]
categories: ["coding"]
date: 2020-07-08T16:32:40-07:00
lastmod: 2020-07-08T16:32:40-07:00
featured: true
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: "Center"
  preview_only: true

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

## Package

GitHub: [https://github.com/sauhaardac/pdfplot](https://github.com/sauhaardac/pdfplot)

PyPI: [https://pypi.org/project/pdfplot/](https://pypi.org/project/pdfplot/)

Install instructions: `pip install pdfplot`

## Background

When evaluating a machine learning model or analyzing a shifting dataset, you'll often need to regenerate a set of figures multiple times. If you have many figures, this means saving them all into separate files, manually updating file names, etc. As a project grows, this can become pretty cumbersome.

That's what my lightweight python package, PDFPlot, is for! It takes care of all of the boilerplate plotting so you can focus on *making* cool graphs instead of organizing them.

The best part is, you can create your plots **anywhere** in your code. As long as you call `pdfplot.save_figs` before the run is completed, all of your plots will be saved and organized.

## Usage

```python
import pdfplot

fig, ax = pdfplot.make_fig()
ax.plot([0, 1, 2, 3], [4, 3, 2, 1])
ax.set_title('First Plot plot!')

fig, ax = pdfplot.make_fig()
ax.plot([0, 1, 2, 3], [1, 2, 3, 4])
ax.set_title('Second plot!')

pdfplot.save_figs(folder='figures')  # magic
```

The `save_figs` method has a lot of built in magic. It does a few things by default:

- Automatically timestamps your figures with the date and time (sortable and human readable of course!)
- Creates a shortcut to the most recent file, named "latest.pdf"
- Opens the created pdf file in your default PDF viewer

These features can be toggled via arguments to the save_figs method.



You can also overload the `make_fig` method  your plots to adjust all of your plots, simultaneously. If you want a grid, just update `make_fig`:

```python
def make_fig():
    fig, ax = plt.subplots()
    fig.tight_layout()
    ax.grid(True)
    return fig, ax

pdfplot.make_fig = make_fig
```



## Credits

Thanks to [Benjamin Rivière](https://www.linkedin.com/in/benjamin-rivi%C3%A8re-442419a2/) for introducing me to the PDF generation functionality in Matplotlib!
