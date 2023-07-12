---
title: "Share With Thy Neighbors: Single-View Reconstruction by Cross-Instance Consistency"
layout: post
date: 2023-06-16
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Single View Reconstruction
- 3D Computer Vision
- Unsupervised learning
star: true
category: blog
author: Zihao Cao
description: Introduction of the first unsupervised SVR method in the world
---

## Background

It's quite amazing that human eyes and brains can work together and directly understand the 3D structure of the objects that we see in 2D images. However, it's a hard task for the computer to do so ———— reconstructuring the object from single view. 

Recent advancements in deep learning methods have dramatically improved results in SVR(single view reconstruction), however, the best methods still require costly supervision at training time. Although crucial to achieve reasonable results, priors like silhouettes and symmetry can also harm the reconstruction quality: silhouette annotations are often coarse and small symmetry prediction errors can yield unrealistic reconstructions.

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/9ae48935-3c33-44e2-b73b-7fa92d0030da)



Then it comes to the thinking: is there an unsupervised learning method for SVR? 

Before this paper, the answer was no. 

In this paper, the authors present UNICORN, a framework leveraging UNsupervised cross-Instance COnsistency for 3D ReconstructioN. It is the most unsupervised approach to single-view reconstruction and their main contributions are: 1) the most unsupervised SVR system; 2) two data-driven techniques to enforce cross-instance consistency

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/d3d56cb4-c25c-4345-831b-16eb0aaeb258)



## Related works
### Mesh-based differentiable rendering
![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/f541f889-ec22-4f18-b6f2-3fbd8aa3e8b5)


The authors represent 3D models as meshes with parametrized surfaces, as introduced in AtlasNet[1]. They optimize the mesh geometry, texture and camera parameters associated to an image using differentiable rendering. 
Loper and Black[2] introduce the first generic differentiable renderer by approximating derivatives with local filters, and Kato[3] proposes an alternative approximation more suitable to learning neural networks.
Another set of methods instead approximates the rendering function to allow differentiability, including SoftRasterizer[4].

### Cross-instance consistency
![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/1962cacf-562e-44ab-8889-dc349810498f)

Inspired by [5], the SVR system of [6] is learned by enforcing consistency between the interpolated 3D attributes of two instances and attributes predicted for the associated reconstruction. 

Closer to this paper’s approach, [7] introduces a loss enforcing cross-silhouette consistency. Yet it differs from this paper in two ways:
(i) the loss operates on silhouettes, whereas our loss is adapted to image reconstruction by modeling background and separating two terms related to shape and texture, 
(ii) the loss is used as a refinement on top of two cycle consistency losses for poses and 3D reconstructions, whereas we demonstrate results without additional self-supervised losses.

### Curriculum learning

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/d021fb8c-1172-45f8-9228-5889478046e7)

We, humanbeings, always start from simple incrementally to complex problem when we try to learn something. It also makes sense for the Machine Learning.
The idea of learning networks by “starting small” dates back to Elman [8] where two curriculum learning schemes are studied: 
(i) increasing the difficulty of samples, 
(ii) increasing the model complexity.
The authors call them curriculum sampling and curriculum modeling. 

Known to drastically improve the convergence speed, curriculum sampling is widely adopted across various applications. On the contrary, curriculum modeling is typically less studied although crucial to various methods.We propose a new form of curriculum modeling dubbed progressive conditioning which enables us to avoid bad minima



## Model
### Overview

The approach can be seen as a structured autoencoder: it takes an image as input, computes parameters with an encoder, and decodes them into explicit and interpretable factors that are composed to generate an image.

The image I is fed to convolutional encoder networks eθ which output parameters eθ(I) = {zsh, ztx, a, zbg} used for the decoding part. In the following, we describe the decoding modules using these parameters to build the final image by generating a shape, adding texture, positioning it and rendering it over a background.

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/bb71524f-9e91-4c03-92ba-9e8048f844ed)

### 1.Shape deformation
![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/f7aa2641-1235-43ab-a934-f33ffc245ba4)

The authors follow [9] and use the parametrization of AtlasNet [1] where different shapes are represented as deformation fields applied to the unit sphere. 

They apply the deformation to an icosphere slightly stretched into an ellipsoid mesh E using a fixed anisotropic scaling. Because they found that using an ellipsoid instead of a raw icosphere could be very effective in encouraging the learning of objects aligned w.r.t. the canonical axes

### 2.Texturing
![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/ea20c50a-b301-4017-9eeb-e8200b98f894)

Following the idea of CMR [10], the authors model textures as an image UV-mapped onto the mesh through the reference ellipsoid. 

They provide a texture code ztx, a convolutional network tθ is used to produce an image tθ(ztx), which is UV-mapped onto the sphere using spherical coordinates to associate a 2D point to every vertex of the ellipsoid, and thus to each vertex of the shaped mesh.

### 3.Affine transformation
![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/7ccdaee3-2a99-4553-b820-8a213ed2a612)

The authors found it beneficial to explicitly model an anisotropic scaling of the objects. They predict K poses candidates, defined by rotations r1:K and translations t1:K, and associated probabilities p1:K.  Then they select the pose with highest probability, combine the scaling and the most likely 6D pose in a single affine transformation module 


### 4.Rendering with background
![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/413256d7-eafd-4d6e-9b03-89ea02dc1139)

The final step of the process is to render the mesh over a background image. The background image bθ(zbg) is generated from a background code zbg by a convolutional network bθ.


## Learning Methods
![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/0db2c3d4-46e1-4f4b-8028-ff8698d378e4)

The authors propose to learn the structured autoencoder without any supervision, by synthesizing 2D images and minimizing a reconstruction loss. Due to the unconstrained nature of the problem, such an approach typically yields degenerate solution. While previous works leverage silhouettes and dataset-specific priors to mitigate this issue, the authors instead propose two unsupervised data-driven techniques, namely progressive conditioning (a training strategy) and neighbor reconstruction (a training loss).


### Progressive conditioning

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/d9a96e9a-ea39-496b-a885-24d393329501)

The goal of progressive conditioning is to encourage the model to share elements (e.g., shape, texture, background) across instances to prevent degenerate solutions.They implement progressive conditioning by masking, stage-by-stage, a decreasing number of values of the latent code. All the experiments share the same 4-stage training strategy where the latent code dimension is increased at the beginning of each stage and the network is then trained until convergence.

### Alternate 3D and pose learning


![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/663d9a9d-7c7a-4ced-a334-3234930a9ac6)

<div align="center">

Previous Method
</div>

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/19614a2c-e1aa-4722-8b3c-04f18726ad2a)





### Training Loss
#### 3D step
![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/f1e8bd1c-c7f4-4b72-9f51-9c3597e0c559)

the 3D-step where shape, texture and background branches of the network are updated by minimizing L3D using the pose associated to the highest probability
##### Neighbor reconstruction
![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/763f745d-9103-4d02-8a05-2c6d8a2298e0)

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/af1f6f6d-218e-4501-9391-8750652ddd05)

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/79cfdca6-f2ad-4292-8648-1fca5b27adf9)


##### Reconstruction losses
![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/71193680-54af-4581-a02e-bfa66477fefd)

##### regularization losses
![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/14a0f375-9f77-451d-beaf-5229701bdcec)




#### P step

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/ab52a57a-c74f-46b3-a154-5762850d4e8d)

the P-step where the branches of the network predicting candidate poses and their associated probabilities are updated by minimizing:


## Evaluation

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/564cf84b-44b2-4bbf-8af3-e66b7ea2b613)

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/4de39b37-1917-4794-8795-2da112bb7cf3)

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/29ddde42-8547-4a80-9779-259e35096f71)

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/7f1189e0-e403-41bb-bd9b-e7dee39fe1ce)


## My run

![image](https://github.com/claude0318/claude0318.github.io/assets/69024793/cb71bc6a-87e3-4c56-8655-4d3922bc0bf2)



## Conclusions



Reference:

[1] Groueix, T., Fisher, M., Kim, V.G., Russell, B.C., Aubry, M.: AtlasNet: A Papier Mˆach´e Approach to Learning 3D Surface Generation. In: CVPR (2018) 2, 4, 6, 24

[2] Loper, M.M., Black, M.J.: OpenDR: An Approximate Differentiable Renderer. In: ECCV 2014, vol. 8695 (2014) 4

[3] Kato, H., Ushiku, Y., Harada, T.: Neural 3D Mesh Renderer. In: CVPR (2018) 2, 4, 10

[4] Liu, S., Li, T., Chen, W., Li, H.: Soft Rasterizer: A Differentiable Renderer for Image-based 3D Reasoning. In: ICCV (2019) 2, 3, 4, 5, 7, 8, 10, 11, 18, 23, 25 

[5] Kulkarni, N., Gupta, A., Tulsiani, S.: Canonical Surface Mapping via Geometric Cycle Consistency. In: ICCV (2019) 5, 12

[6] Hu, T., Wang, L., Xu, X., Liu, S., Jia, J.: Self-Supervised 3D Mesh Reconstruction From Single Images. In: CVPR (2021) 2, 4, 5, 10, 12

[7] Navaneet, K.L., Mathew, A., Kashyap, S., Hung, W.C., Jampani, V., Babu, R.V.: From Image Collections to Point Clouds with Self-supervised Shape and Pose Networks. In: CVPR (2020) 5

[8] Elman, J.L.: Learning and development in neural networks: The importance of starting small. Cognition (1993) 5, 8

[9] Tulsiani, S., Kulkarni, N., Gupta, A.: Implicit Mesh Reconstruction from Unannotated Image Collections. arXiv:2007.08504 [cs] (2020) 2, 4, 6, 9, 11, 12, 21

[10] Kanazawa, A., Tulsiani, S., Efros, A.A., Malik, J.: Learning Category-Specific Mesh Reconstruction from Image Collections. In: ECCV (2018) 2, 4, 6, 11, 12



## Paragraph modifiers

### Quote

> Here is a quote. What this is should be self explanatory. Quotes are automatically indented when they are used.

{% highlight raw %}
> Here is a quote. What this is should be self explanatory.
{% endhighlight raw %}

---

## URLs

URLs can be made in a handful of ways:

* A named link to [Mark It Down][3].
* Another named link to [Mark It Down](https://google.com/)
* Sometimes you just want a URL like <https://google.com/>.

{% highlight raw %}
* A named link to [MarkItDown][3].
* Another named link to [MarkItDown](https://google.com/)
* Sometimes you just want a URL like <https://google.com/>.
{% endhighlight %}

---

## Horizontal rule

A horizontal rule is a line that goes across the middle of the page.
It's sometimes handy for breaking things up.

{% highlight raw %}
---
{% endhighlight %}

---

## Images

Markdown can also contain images. I'll need to add something here sometime.

{% highlight raw %}
![Markdowm Image][/image/url]
{% endhighlight %}

![Markdowm Image][5]

*Figure Caption*?

{% highlight raw %}
![Markdowm Image][/image/url]
<figcaption class="caption">Photo by John Doe</figcaption>
{% endhighlight %}

![Markdowm Image][5]
<figcaption class="caption">Photo by John Doe</figcaption>

*Bigger Images*?

{% highlight raw %}
![Markdowm Image][/image/url]{: class="bigger-image" }
{% endhighlight %}

![Markdowm Image][5]{: class="bigger-image" }

---


