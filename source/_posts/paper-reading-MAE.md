---
title: 'paper reading: MAE'
author: daydreamatnight
toc: true
declare: true
date: 2022-04-27 02:00:38
tags:
  - paper reading
  - deep learning
---

> This is a [series of paper reading notes](https://daydreamatnight.github.io/2022/04/02/paper-reading-start/), hopefully, to push me to read paper casually and to leave some record of what I've learned.

> Published in Dec 2021, this new work by Kaiming He draws a lot of attention from the community. The astounding result of unsupervised transfer learning and the capability of reconstructing highly masked (up to 90%) images might herald a new era in CV. 

<!-- more -->

Paper: [Masked autoencoders are scalable vision learners](https://arxiv.org/abs/2111.06377)

Useful link: https://www.bilibili.com/video/BV1sq4y1q77t/

### Abstract

Inspired by BERT and ViT, this paper proposes an asymmetric, transformer-based, denoising auto-encoder architecture. The unsupervised pre-training task is to reconstruct highly masked input images. The pre-training time is reduced by 3 times with competitive accuracy. The transfer performance is even better than the supervised pre-training models.

### Conclusion

The results show that MAE makes scaleable unsupervised pre-training in CV applicable, a similar route to that of NLP. 

They claim the semantic density difference between text and image leads to different masking operations. Besides, the patch masking operation does not separate semantic entities, meaning one masked patch may include more than one piece of semantic information, unlike language. However, the reconstruction results show that the model manages to learn from complex semantics.

### Introduction

Although yielding excellent success in NLP, applying scalable unsupervised models in CV is still a challenging problem. But why? i.e. **what makes masked autoencoding different between vision and language? **3 reasons are discussed:

- The architecture difference between convolution and transformer: it's hard to integrate masked embedding or positional embedding to the convolution layer. --addressed by ViT.

- The information density difference between text and image: Unlike high-semantic text, natural signals in images possess heavy spatial redundancy. --addressed by masking a very high portion of random patches.
- Decoder difference: in NLP, take BERT as an example, a simple linear projection is used as a decoder, while in vision, a simple decoder is not powerful enough to reconstruct the semantic level information -- addressed by substituting linear projection with transformer layers.

Then, the idea of MAE is on the front door. The encoder processes only the unmasked patches, while the lightweight decoder reconstructs the whole image from the encoded latent representation and the [mask] tokens. With a very high masking ratio(e.g. 75%), the pre-training time can be reduced by 3 times.

Besides, the data capacity and generalisation performance are great. SOTA accuracy is achieved with fine tuning on a medium-sized dataset.

### Relate work

Works in 4 areas are briefly reviewed. 

{% markmap 300px %}

- **Masked language modelling:**

  - [BERT](https://arxiv.org/abs/1810.04805)

  - [GPT](https://www.cs.ubc.ca/~amuham01/LING530/papers/radford2018improving.pdf)
- **Auto-encoding:** 

  - [classic autoencoders](https://proceedings.neurips.cc/paper/1993/hash/9e3cfc48eccf81a0d57663e129aef3cb-Abstract.html): PCA, k-means 

  - [denoising autoencoders(DAE)](https://dl.acm.org/doi/abs/10.1145/1390156.1390294)
- **Masked image encoding:** 

	- classic
		- pioneer work [SDAE](https://www.jmlr.org/papers/volume11/vincent10a/vincent10a.pdf?ref=https://githubhelp.com)
		-  [context encoder](http://openaccess.thecvf.com/content_cvpr_2016/html/Pathak_Context_Encoders_Feature_CVPR_2016_paper.html)

	- transformer based: 
		- [iGPT](http://proceedings.mlr.press/v119/chen20s.html)
		- [ViT](https://arxiv.org/abs/2010.11929)
		- [BEiT](https://arxiv.org/abs/2106.08254)
- **Self-supervised learning:** 
  - CNN based: 
    - [Unsupervised learning of visual representations using videos](http://openaccess.thecvf.com/content_iccv_2015/html/Wang_Unsupervised_Learning_of_ICCV_2015_paper.html)
    - [CFN](https://link.springer.com/chapter/10.1007/978-3-319-46466-4_5)
    - [Colorful Image Colorization](https://link.springer.com/chapter/10.1007/978-3-319-46487-9_40)
    - [Unsupervised representation learning by predicting image rotations](https://arxiv.org/abs/1803.07728)
  - Transformer based: 
    - [ViT](https://arxiv.org/abs/2010.11929)
  - Contrastive learning based: 
    - [Unsupervised feature learning via non-parametric instance discrimination](http://openaccess.thecvf.com/content_cvpr_2018/html/Wu_Unsupervised_Feature_Learning_CVPR_2018_paper.html)
    - [Representation learning with contrastive predictive coding](https://arxiv.org/abs/1807.03748)
    - [MOCO](http://openaccess.thecvf.com/content_CVPR_2020/html/He_Momentum_Contrast_for_Unsupervised_Visual_Representation_Learning_CVPR_2020_paper.html)

{%endmarkmap%}

### Approach

The architecture and the training approach is briefly covered in the sketch below:

<img src="MAE architecture.png" alt="MAE architecture" style="zoom:40%;" />

Additional details about the architecture: For per-processing, non-overlapped patching and random uniform masking are adopted. The [mask] token is shared, and another position embedding is introduced to the decoder input so that the inputs are different on different masked area. But it is unclear whether the positional embedding is performed only on the [mask] token or on the whole input, i.e. encoded patches + [mask] token. Furthermore, the default decoder has <10% computation per token compared with the encoder.

Reconstruction target: The decoder aims to recreate the pixels of masked patches. The loss function is the mean squared error (MSE) between the output and the original image, only on the masked region of course. Besides, a variation reconstructing the normalised pixels shows an improvement in representation quality.

### ImageNet experiments

First, the most astonish reconstruction results are shown below:

<img src="paper-reading-MAE/MAE result.png" alt="MAE result" style="zoom:50%;" />

<img src="paper-reading-MAE/MAE result2.png" alt="MAE result2" style="zoom:30%;" />

Mask ratio is higher than [BERT](https://arxiv.org/abs/1810.04805)(15%) and [iGPT](http://proceedings.mlr.press/v119/chen20s.html), [ViT](https://arxiv.org/abs/2010.11929) and [BEiT](https://arxiv.org/abs/2106.08254)(25%-50%), 

<img src="paper-reading-MAE/MAE mask ratio.png" alt="MAE mask ratio" style="zoom:60%;" />



### Transfer learning experiments



### Reference

[Devlin, J., Chang, M. W., Lee, K., & Toutanova, K. (2018). Bert: Pre-training of deep bidirectional transformers for language understanding. *arXiv preprint arXiv:1810.04805*.](https://arxiv.org/abs/1810.04805)

[Radford, A., Narasimhan, K., Salimans, T., & Sutskever, I. (2018). Improving language understanding by generative pre-training.](https://www.cs.ubc.ca/~amuham01/LING530/papers/radford2018improving.pdf)

[Hinton, G. E., & Zemel, R. (1993). Autoencoders, minimum description length and Helmholtz free energy. *Advances in neural information processing systems*, *6*.](https://proceedings.neurips.cc/paper/1993/hash/9e3cfc48eccf81a0d57663e129aef3cb-Abstract.html)

[Vincent, P., Larochelle, H., Bengio, Y., & Manzagol, P. A. (2008, July). Extracting and composing robust features with denoising autoencoders. In *Proceedings of the 25th international conference on Machine learning* (pp. 1096-1103).](https://dl.acm.org/doi/abs/10.1145/1390156.1390294)

[Vincent, P., Larochelle, H., Lajoie, I., Bengio, Y., Manzagol, P. A., & Bottou, L. (2010). Stacked denoising autoencoders: Learning useful representations in a deep network with a local denoising criterion. *Journal of machine learning research*, *11*(12).](https://www.jmlr.org/papers/volume11/vincent10a/vincent10a.pdf?ref=https://githubhelp.com)

[Pathak, D., Krahenbuhl, P., Donahue, J., Darrell, T., & Efros, A. A. (2016). Context encoders: Feature learning by inpainting. In *Proceedings of the IEEE conference on computer vision and pattern recognition* (pp. 2536-2544).](http://openaccess.thecvf.com/content_cvpr_2016/html/Pathak_Context_Encoders_Feature_CVPR_2016_paper.html)

[Chen, M., Radford, A., Child, R., Wu, J., Jun, H., Luan, D., & Sutskever, I. (2020, November). Generative pretraining from pixels. In *International Conference on Machine Learning* (pp. 1691-1703). PMLR.](http://proceedings.mlr.press/v119/chen20s.html)

[Dosovitskiy, A., Beyer, L., Kolesnikov, A., Weissenborn, D., Zhai, X., Unterthiner, T., ... & Houlsby, N. (2020). An image is worth 16x16 words: Transformers for image recognition at scale. *arXiv preprint arXiv:2010.11929*.](https://arxiv.org/abs/2010.11929)

[Bao, H., Dong, L., & Wei, F. (2021). Beit: Bert pre-training of image transformers. *arXiv preprint arXiv:2106.08254*.](https://arxiv.org/abs/2106.08254)

[Wang, X., & Gupta, A. (2015). Unsupervised learning of visual representations using videos. In *Proceedings of the IEEE international conference on computer vision* (pp. 2794-2802).](http://openaccess.thecvf.com/content_iccv_2015/html/Wang_Unsupervised_Learning_of_ICCV_2015_paper.html)

[Noroozi, M., & Favaro, P. (2016, October). Unsupervised learning of visual representations by solving jigsaw puzzles. In *European conference on computer vision* (pp. 69-84). Springer, Cham.](https://link.springer.com/chapter/10.1007/978-3-319-46466-4_5)

[Zhang, R., Isola, P., & Efros, A. A. (2016, October). Colorful image colorization. In *European conference on computer vision* (pp. 649-666). Springer, Cham.](https://link.springer.com/chapter/10.1007/978-3-319-46487-9_40)

[Gidaris, S., Singh, P., & Komodakis, N. (2018). Unsupervised representation learning by predicting image rotations. *arXiv preprint arXiv:1803.07728*.](https://arxiv.org/abs/1803.07728)

[Wu, Z., Xiong, Y., Yu, S. X., & Lin, D. (2018). Unsupervised feature learning via non-parametric instance discrimination. In *Proceedings of the IEEE conference on computer vision and pattern recognition* (pp. 3733-3742).](http://openaccess.thecvf.com/content_cvpr_2018/html/Wu_Unsupervised_Feature_Learning_CVPR_2018_paper.html)

[Oord, A. V. D., Li, Y., & Vinyals, O. (2018). Representation learning with contrastive predictive coding. *arXiv preprint arXiv:1807.03748*.](https://arxiv.org/abs/1807.03748)

[He, K., Fan, H., Wu, Y., Xie, S., & Girshick, R. (2020). Momentum contrast for unsupervised visual representation learning. In *Proceedings of the IEEE/CVF conference on computer vision and pattern recognition* (pp. 9729-9738).](http://openaccess.thecvf.com/content_CVPR_2020/html/He_Momentum_Contrast_for_Unsupervised_Visual_Representation_Learning_CVPR_2020_paper.html)

