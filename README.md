# clean-fid for Evaluating Generative Models

<br>

<p align="center">
<img src="https://raw.githubusercontent.com/GaParmar/clean-fid/main/docs/images/cleanfid_demo_folders.gif" />
</p>



[**Project**](https://www.cs.cmu.edu/~clean-fid/) | [**Paper**](https://arxiv.org/abs/2104.11222) | 
[**Colab-FID**](https://colab.research.google.com/drive/1ElGAHvlwTilIf_3D3cw1boirCEkFsAWI?usp=sharing) |
[**Colab-Resize**](https://colab.research.google.com/drive/1Q-N94S2mnLsFLpuT7WwY6d5WxGVWLGpg?usp=sharing) |
[**Leaderboard Tables**](#cleanfid-leaderboard-for-common-tasks) <br>
**Quick start:** [**Calculate FID**](#computing-fid) | [**Calculate KID**](#computing-kid) | [**Leaderboard API**](#cleanfid-leaderboard-api)


The FID calculation involves many steps that can produce inconsistencies in the final metric. As shown below, different implementations use different low-level image quantization and resizing functions, the latter of which are often implemented incorrectly.

<p align="center">
  <img src="https://raw.githubusercontent.com/GaParmar/clean-fid/main/docs/images/resize_circle.png"  width="800" />
</p>


We provide an easy-to-use library to address the above issues and make the FID scores comparable across different methods, papers, and groups.

![FID Steps](https://raw.githubusercontent.com/GaParmar/clean-fid/main/docs/images/fid_steps.jpg)


---

[On Buggy Resizing Libraries and Surprising Subtleties in FID Calculation](https://www.cs.cmu.edu/~clean-fid/) <br>
 [Gaurav Parmar](https://gauravparmar.com/), [Richard Zhang](https://richzhang.github.io/), [Jun-Yan Zhu](https://www.cs.cmu.edu/~junyanz/)<br>
arXiv 2104.11222, 2021 <br>
CMU and Adobe

---

### CleanFID Leaderboard API

We compute the FID scores using the corresponding methods used in the original papers and using the Clean-FID proposed here. 
All values are computed using 10 evaluation runs. <br>
We provide an API to query the results shown in the tables below directly from 
the pip package. The arguments `model_name`, `dataset_name`, `dataset_res`, `dataset_split`, `task_name` can be used to filter
the results.
<p align="center">
<img src="https://raw.githubusercontent.com/GaParmar/clean-fid/main/docs/images/demo_leaderboard_cropped.gif" />
</p>


**Buggy Resizing Operations** <br>


  The definitions of resizing functions are mathematical and <em>should never be a function of the library being used</em>. Unfortunately, implementations differ across commonly-used libraries.  They are often implemented incorrectly by popular libraries. Try out the different resizing implementations in the Google colab notebook [here](https://colab.research.google.com/drive/1Q-N94S2mnLsFLpuT7WwY6d5WxGVWLGpg?usp=sharing).

  <img src="https://raw.githubusercontent.com/GaParmar/clean-fid/main/docs/images/resize_circle_extended.png"  width="800" />
<br>

The inconsistencies among implementations can have a drastic effect of the evaluations metrics. The table below shows that FFHQ dataset images resized with  bicubic implementation from other libraries (OpenCV, PyTorch, TensorFlow, OpenCV) have a large FID score (≥ 6) when compared to the same images resized with the correctly implemented PIL-bicubic filter. Other correctly implemented filters from PIL (Lanczos, bilinear, box) all result in relatively smaller FID score (≤ 0.75). Note that since TF 2.0, the new flag `antialias` (default: `False`) can produce results close to PIL. However, it was not used in the existing TF-FID repo and set as `False` by default.

 <p align="center"><img src="https://raw.githubusercontent.com/GaParmar/clean-fid/main/docs/images/table_resize_sc.png"  width="500" /></p>

**JPEG Image Compression**

  Image compression can have a surprisingly large effect on FID.  Images are perceptually indistinguishable from each other but have a large FID score. The FID scores under the images are calculated between all FFHQ images saved using the corresponding JPEG format and the PNG format.

<p align="center">
  <img src="https://raw.githubusercontent.com/GaParmar/clean-fid/main/docs/images/jpeg_effects.png"  width="800" />
</p>

Below, we study the effect of JPEG compression for StyleGAN2 models trained on the FFHQ dataset (left) and LSUN outdoor Church dataset (right). Note that LSUN dataset images were collected with JPEG compression (quality 75), whereas FFHQ images were collected as PNG. Interestingly, for LSUN dataset, the best FID score (3.48) is obtained when the generated images are compressed with JPEG quality 87.

<p align="center">
  <img src="https://raw.githubusercontent.com/GaParmar/clean-fid/main/docs/images/jpeg_plots.png"  width="800" />
</p>

---

## Quick Start


- install requirements
    ```
    pip install -r requirements.txt
    ```
- install the library
    ```
    pip install clean-fid
    ```
### Computing FID
- Compute FID between two image folders
    ```
    from cleanfid import fid
    score = fid.compute_fid(fdir1, fdir2)
    ```
- Compute FID between one folder of images and pre-computed datasets statistics (e.g., `FFHQ`)
    ```
    from cleanfid import fid
    score = fid.compute_fid(fdir1, dataset_name="FFHQ", dataset_res=1024, dataset_split="trainval70k")
    ```
- Compute FID using a generative model and pre-computed dataset statistics:
    ```
    from cleanfid import fid
    # function that accepts a latent and returns an image in range[0,255]
    gen = lambda z: GAN(latent=z, ... , <other_flags>)
    score = fid.compute_fid(gen=gen, dataset_name="FFHQ",
            dataset_res=256, num_gen=50_000, dataset_split="trainval70k")
    ```

### Computing KID
The KID score can be computed using a similar interface as FID. 
The dataset statistics for KID are only precomputed for smaller datasets `AFHQ`, `BreCaHAD`, and `MetFaces`.

- Compute KID between two image folders
    ```
    from cleanfid import fid
    score = fid.compute_kid(fdir1, fdir2)
    ```
- Compute KID between one folder of images and pre-computed datasets statistics
    ```
    from cleanfid import fid
    score = fid.compute_kid(fdir1, dataset_name="brecahad", dataset_res=512, dataset_split="train")
    ```
- Compute KID using a generative model and pre-computed dataset statistics:
    ```
    from cleanfid import fid
    # function that accepts a latent and returns an image in range[0,255]
    gen = lambda z: GAN(latent=z, ... , <other_flags>)
    score = fid.compute_kid(gen=gen, dataset_name="brecahad", dataset_res=512, num_gen=50_000, dataset_split="train")
    ```

---
### Supported Precomputed Datasets

We provide precompute statistics for the following commonly used configurations. Please contact us if you want to add statistics for your new datasets. 

| Task             | Dataset   | Resolution | Reference Split          | # Reference Images | mode |
| :-:              | :---:     | :-:        | :-:            |  :-:          | :-: |
| Image Generation | [`cifar10`](https://www.cs.toronto.edu/~kriz/cifar.html)     | 32         | `train`        |  50,000       |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Image Generation | [`cifar10`](https://www.cs.toronto.edu/~kriz/cifar.html)     | 32         | `test`         |  10,000       |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Image Generation | [`ffhq`](https://github.com/NVlabs/ffhq-dataset)        | 1024, 256  | `trainval`     |  50,000       |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Image Generation | [`ffhq`](https://github.com/NVlabs/ffhq-dataset)        | 1024, 256  | `trainval70k`  |  70,000       |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Image Generation | [`lsun_church`](https://www.yf.io/p/lsun/) | 256        | `train`        |  50,000       |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Image Generation | [`lsun_horse`](https://www.yf.io/p/lsun/)  | 256        | `train`        |  50,000       |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Image Generation | [`lsun_cat`](https://www.yf.io/p/lsun/)    | 256        | `train`        |  50,000       |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Image Generation | [`lsun_cat`](https://www.yf.io/p/lsun/)    | 256        | `trainfull`    |  1,657,264    |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Few Shot Generation | [`afhq_cat`](https://github.com/clovaai/stargan-v2/)  | 512        | `train`       |  5153         |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Few Shot Generation | [`afhq_dog`](https://github.com/clovaai/stargan-v2/)  | 512        | `train`       |  4739         |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Few Shot Generation | [`afhq_wild`](https://github.com/clovaai/stargan-v2/) | 512        | `train`       |  4738         |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Few Shot Generation | [`brecahad`](https://figshare.com/articles/dataset/BreCaHAD_A_Dataset_for_Breast_Cancer_Histopathological_Annotation_and_Diagnosis/7379186)  | 512        | `train`       |  1944         |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Few Shot Generation | [`metfaces`](https://github.com/NVlabs/metfaces-dataset)  | 1024       | `train`       |  1336         |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Image to Image   | `horse2zebra`  | 256        | `test`        |  140          |`clean`, `legacy_tensorflow`, `legacy_pytorch`|
| Image to Image   | `cat2dog`      | 256        | `test`        |  500          |`clean`, `legacy_tensorflow`, `legacy_pytorch`|



**Using precomputed statistics**
In order to compute the FID score with the precomputed dataset statistics, use the corresponding options. For instance, to compute the clean-fid score on generated 256x256 FFHQ images use the command:
  ```
  fid_score = fid.compute_fid(fdir1, dataset_name="ffhq", dataset_res=256,  mode="clean", dataset_split="trainval70k")
  ```

---

### Create Custom Dataset Statistics
- *dataset_path*: folder where the dataset images are stored
- *custom_name*: name to be used for the statistics
- Generating custom statistics (saved to local cache)
  ```
  from cleanfid import fid
  fid.make_custom_stats(custom_name, dataset_path, mode="clean")
  ```

- Using the generated custom statistics
  ```
  from cleanfid import fid
  score = fid.compute_fid("folder_fake", dataset_name=custom_name,
            mode="clean", dataset_split="custom")
  ```

- Removing the custom stats
  ```
  from cleanfid import fid
  fid.remove_custom_stats(custom_name, mode="clean")
  ```

- Check if a custom statistic already exists
  ```
  from cleanfid import fid
  fid.test_stats_exists(custom_name, mode)
  ```

---

## Backwards Compatibility

We provide two flags to reproduce the legacy FID score.

- `mode="legacy_pytorch"` <br>
    This flag is equivalent to using the popular PyTorch FID implementation provided [here](https://github.com/mseitzer/pytorch-fid/)
    <br>
    The difference between using clean-fid with this option and [code](https://github.com/mseitzer/pytorch-fid/) is **~2e-06**
    <br>
    See [doc](https://github.com/GaParmar/clean-fid/blob/main/docs/pytorch_fid.md) for how the methods are compared


- `mode="legacy_tensorflow"` <br>
    This flag is equivalent to using the official [implementation of FID](https://github.com/bioinf-jku/TTUR) released by the authors.
    <br>
    The difference between using clean-fid with this option and [code](https://github.com/bioinf-jku/TTUR) is **~2e-05**
    <br>
  See [doc](https://github.com/GaParmar/clean-fid/blob/main/docs/tensorflow_fid.md) for detailed steps for how the methods are compared

---

## Building clean-fid locally from source
   ```
   python setup.py bdist_wheel
   pip install dist/*
   ```

---

## CleanFID Leaderboard for common tasks

We compute the FID scores using the corresponding methods used in the original papers and using the Clean-FID proposed here. 
All values are computed using 10 evaluation runs. We provide an [API](#cleanfid-leaderboard-api) to query the results shown in the tables below directly from the pip package.

If you would like to add new numbers and models to our leaderboard, feel free to contact us. 

### CIFAR-10 (few shot)

The `test` set is used as the reference distribution and compared to 10k generated images.

**100% data**
| Model	| Legacy-FID<br>(reported)	| Legacy-FID<br>(reproduced)	| Clean-FID	|
| :---: | :---: | :---: | :---: |
| stylegan2-diff-augment [[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-cifar10.pkl)	| 9.89	| 9.90 ± 0.09	| 10.85 ± 0.10	|
| stylegan2-mirror-flips 	[[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-cifar10.pkl)	| 11.07	| 11.07 ± 0.10	| 12.96 ± 0.07	|

**20% data**
| Model	| Legacy-FID<br>(reported)	| Legacy-FID<br>(reproduced)	| Clean-FID	|
| :---: | :---: | :---: | :---: |
| stylegan2-diff-augment [[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-cifar10-0.2.pkl)	| 12.15	| 12.12 ± 0.15	| 14.18 ± 0.13	|
| stylegan2-mirror-flips [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958)	[[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-cifar10-0.2.pkl)	| 23.08	| 23.01 ± 0.19	| 29.49 ± 0.17	|

**10% data**
| Model	| Legacy-FID<br>(reported)	| Legacy-FID<br>(reproduced)	| Clean-FID	|
| :---: | :---: | :---: | :---: |
| stylegan2-diff-augment [[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf)	[[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-cifar10-0.1.pkl)	| 14.50	| 14.53 ± 0.12	| 16.98 ± 0.18	|
| stylegan2-mirror-flips [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958)	[[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-cifar10-0.1.pkl)	| 36.02	| 35.94 ± 0.17	| 43.60 ± 0.17	|

<br>

### CIFAR-100 (few shot)

The `test` set is used as the reference distribution and compared to 10k generated images.

**100% data**
| Model	| Legacy-FID<br>(reported)	| Legacy-FID<br>(reproduced)	| Clean-FID	|
| :---: | :---: | :---: | :---: |
| stylegan2-mirror-flips	[[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-cifar100.pkl)	| 16.54	| 16.44 ± 0.19	| 18.44 ± 0.24	|
| stylegan2-diff-augment	[[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-cifar100.pkl)	| 15.22	| 15.15 ± 0.13	| 16.80 ± 0.13	|

**20% data**
| Model	| Legacy-FID<br>(reported)	| Legacy-FID<br>(reproduced)	| Clean-FID	|
| :---: | :---: | :---: | :---: |
| stylegan2-mirror-flips [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-cifar100-0.2.pkl)	| 32.30	| 32.26 ± 0.19	| 34.88 ± 0.14	|
| stylegan2-diff-augment [[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-cifar100-0.2.pkl)	| 16.65	| 16.74 ± 0.10	| 18.49 ± 0.08	|

**10% data**
| Model	| Legacy-FID<br>(reported)	| Legacy-FID<br>(reproduced)	| Clean-FID	|
| :---: | :---: | :---: | :---: |
| stylegan2-mirror-flips [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-cifar100-0.1.pkl)	| 45.87	| 45.97 ± 0.20	| 46.77 ± 0.19	|
| stylegan2-diff-augment [[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-cifar100-0.1.pkl)	| 20.75	| 20.69 ± 0.12	| 23.40 ± 0.09	|

<br>

### FFHQ

**all images @ 1024x1024**<br>
Values are computed using 50k generated images
| Model     | Legacy-FID<br>(reported) | Legacy-FID<br>(reproduced)    | Clean-FID  | Reference Split |
| :---:     | :-:          | :-:          | :-:         | :-: |
 | stylegan2 [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://nvlabs-fi-cdn.nvidia.com/stylegan2-ada-pytorch/pretrained/ffhq.pkl) | 2.84 | 2.86 ± 0.025 | 3.07 ± 0.025 | `trainval` |
 | stylegan2 [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://nvlabs-fi-cdn.nvidia.com/stylegan2-ada-pytorch/pretrained/ffhq.pkl) | N/A | 2.76 ± 0.025 | 2.98 ± 0.025 | `trainval70k` |
 
**30k - images @ 256x256 (Few Shot Generation)**<br>
 The 70k images from `trainval70k` set is used as the reference images and compared to 50k generated images.
| Model     | Legacy-FID<br>(reported) | Legacy-FID<br>(reproduced)    | Clean-FID  |
| :---:     | :-:          | :-:          | :-:         |
| stylegan2 [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-ffhq-30k.pkl) | 6.16 | 6.14 ± 0.064 | 6.49 ± 0.068 |
| DiffAugment-stylegan2 [[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-ffhq-30k.pkl) | 5.05 | 5.07 ± 0.030 | 5.18 ± 0.032 |

**10k - images @ 256x256 (Few Shot Generation)**<br>
The 70k images from `trainval70k` set is used as the reference images and compared to 50k generated images.
| Model     | Legacy-FID<br>(reported) | Legacy-FID<br>(reproduced)    | Clean-FID  |
| :---:     | :-:          | :-:          | :-:         |
| stylegan2 [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-ffhq-10k.pkl) | 14.75 | 14.88 ± 0.070 | 16.04 ± 0.078 |
| DiffAugment-stylegan2 [[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-ffhq-10k.pkl) | 7.86 | 7.82 ± 0.045 | 8.12 ± 0.044 |


**5k - images @ 256x256 (Few Shot Generation)**<br>
The 70k images from `trainval70k` set is used as the reference images and compared to 50k generated images.
| Model     | Legacy-FID<br>(reported) | Legacy-FID<br>(reproduced)    | Clean-FID  |
| :---:     | :-:          | :-:          | :-:         |
| stylegan2 [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-ffhq-5k.pkl) | 26.60 | 26.64 ± 0.086 | 28.17 ± 0.090 |
| DiffAugment-stylegan2 [[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-ffhq-5k.pkl) | 10.45 | 10.45 ± 0.047 | 10.99 ± 0.050 |

**1k - images @ 256x256 (Few Shot Generation)** <br>
The 70k images from `trainval70k` set is used as the reference images and compared to 50k generated images.
| Model     | Legacy-FID<br>(reported) | Legacy-FID<br>(reproduced)    | Clean-FID  |
| :---:     | :-:          | :-:          | :-:         |
| stylegan2 [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-ffhq-1k.pkl) | 62.16 | 62.14 ± 0.108 | 64.17 ± 0.113 |
| DiffAugment-stylegan2 [[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-ffhq-1k.pkl) | 25.66 | 25.60 ± 0.071 | 27.26 ± 0.077 |

 <br>

 ### LSUN Categories
 
 **100% data**<br>
 The 50k images from `train` set is used as the reference images and compared to 50k generated images.
| Category | Model     | Legacy-FID<br>(reported)  | Legacy-FID<br>(reproduced)    | Clean-FID  |
|:-: | :---:           | :-:                    | :-:          | :-:         |
Outdoor Churches | stylegan2 [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://nvlabs-fi-cdn.nvidia.com/stylegan2/networks/stylegan2-church-config-f.pkl) | 3.86 | 3.87 ± 0.029 | 4.08 ± 0.028 |
Horses           | stylegan2 [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://nvlabs-fi-cdn.nvidia.com/stylegan2/networks/stylegan2-horse-config-f.pkl)  | 3.43 | 3.41 ± 0.021 | 3.62 ± 0.023 |
Cat              | stylegan2 [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://nvlabs-fi-cdn.nvidia.com/stylegan2/networks/stylegan2-cat-config-f.pkl)    | 6.93 | 7.02 ± 0.039 | 7.47 ± 0.035 |

<br>

**LSUN CAT - 30k images (Few Shot Generation)**<br>
All 1,657,264 images from `trainfull` split are used as the reference images and compared to 50k generated images.
| Model     | Legacy-FID<br>(reported) | Legacy-FID<br>(reproduced)    | Clean-FID  |
| :---:     | :-:          | :-:          | :-:         |
| stylegan2-mirror-flips [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-lsun-cat-30k.pkl)	| 10.12	| 10.15 ± 0.04	| 10.87 ± 0.04	|
| stylegan2-diff-augment [[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-lsun-cat-30k.pkl)	| 9.68	| 9.70 ± 0.07	| 10.25 ± 0.07	|

**LSUN CAT - 10k images (Few Shot Generation)**<br>
All 1,657,264 images from `trainfull` split are used as the reference images and compared to 50k generated images.
| Model     | Legacy-FID<br>(reported) | Legacy-FID<br>(reproduced)    | Clean-FID  |
| :---:     | :-:          | :-:          | :-:         |
| stylegan2-mirror-flips [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-lsun-cat-10k.pkl)	| 17.93	| 17.98 ± 0.09	| 18.71 ± 0.09	|
| stylegan2-diff-augment [[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-lsun-cat-10k.pkl)	| 12.07	| 12.04 ± 0.08	| 12.53 ± 0.08	|

**LSUN CAT - 5k images (Few Shot Generation)**<br>
All 1,657,264 images from `trainfull` split are used as the reference images and compared to 50k generated images.
| Model     | Legacy-FID<br>(reported) | Legacy-FID<br>(reproduced)    | Clean-FID  |
| :---:     | :-:          | :-:          | :-:         |
| stylegan2-mirror-flips [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-lsun-cat-5k.pkl)	| 34.69	| 34.66 ± 0.12	| 35.85 ± 0.12	|
| stylegan2-diff-augment [[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-lsun-cat-5k.pkl)	| 16.11	| 16.11 ± 0.09	| 16.79 ± 0.09	|

**LSUN CAT - 1k images (Few Shot Generation)**<br>
All 1,657,264 images from `trainfull` split are used as the reference images and compared to 50k generated images.
| Model     | Legacy-FID<br>(reported) | Legacy-FID<br>(reproduced)    | Clean-FID  |
| :---:     | :-:          | :-:          | :-:         |
| stylegan2-mirror-flips [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/stylegan2-lsun-cat-1k.pkl)	| 182.85	| 182.80 ± 0.21	| 185.86 ± 0.21	|
| stylegan2-diff-augment [[Zhao et al, 2020]](https://arxiv.org/pdf/2006.10738.pdf) [[ckpt]](https://hanlab.mit.edu/projects/data-efficient-gans/models/DiffAugment-stylegan2-lsun-cat-1k.pkl)	| 42.26	| 42.07 ± 0.16	| 43.12 ± 0.16	|

<br>

### AFHQ (Few Shot Generation)
**AFHQ Dog**<br>
All 4739 images from `train` split are used as the reference images and compared to 50k generated images.
| Model     | Legacy-FID<br>(reported) | Legacy-FID<br>(reproduced)    | Clean-FID  |
| :---:     | :-:          | :-:          | :-:         |
| stylegan2 [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://nvlabs-fi-cdn.nvidia.com/stylegan2-ada-pytorch/pretrained/paper-fig11a-small-datasets/afhqdog-mirror-stylegan2-noaug.pkl)	| 19.37	| 19.34 ± 0.08	| 20.10 ± 0.08	| 9.62	| 9.56 ± 0.12	| 10.21 ± 0.11	|
| stylegan2-ada [[Karras et al, 2020]](https://arxiv.org/abs/2006.06676) [[ckpt]](https://nvlabs-fi-cdn.nvidia.com/stylegan2-ada-pytorch/pretrained/paper-fig11a-small-datasets/afhqdog-mirror-paper512-ada.pkl)	| 7.40	| 7.41 ± 0.02	| 7.61 ± 0.02	| 1.16	| 1.17 ± 0.03	| 1.28 ± 0.03	|

**AFHQ Wild**<br>
All 4738 images from `train` split are used as the reference images and compared to 50k generated images.
| Model     | Legacy-FID<br>(reported) | Legacy-FID<br>(reproduced)    | Clean-FID  |
| :---:     | :-:          | :-:          | :-:         |
| stylegan2 [[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://nvlabs-fi-cdn.nvidia.com/stylegan2-ada-pytorch/pretrained/paper-fig11a-small-datasets/afhqwild-mirror-stylegan2-noaug.pkl)	| 3.48	| 3.55 ± 0.03	| 3.66 ± 0.02	| 0.77	| 0.78 ± 0.02	| 0.83 ± 0.01	|
| stylegan2-ada [[Karras et al, 2020]](https://arxiv.org/abs/2006.06676) [[ckpt]](https://nvlabs-fi-cdn.nvidia.com/stylegan2-ada-pytorch/pretrained/paper-fig11a-small-datasets/afhqwild-mirror-paper512-ada.pkl)	| 3.05	| 3.01 ± 0.02	| 3.03 ± 0.02	| 0.45	| 0.45 ± 0.01	| 0.45 ± 0.01	|

<br>

### BreCaHAD (Few Shot Generation)
All 1944 images from `train` split are used as the reference images and compared to 50k generated images.
| Model     | Legacy<br>FID<br>(reported) | Legacy<br>FID<br>(reproduced)    | Clean-FID  | Legacy<br>KID<br>(reported)<br>10^3 | Legacy<br>KID<br>(reproduced)<br>10^3    | Clean<br>KID<br>10^3  |
| :---:     | :-:          | :-: | :-: | :-: | :-: | :-: |
| stylegan2	[[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://nvlabs-fi-cdn.nvidia.com/stylegan2-ada-pytorch/pretrained/paper-fig11a-small-datasets/brecahad-mirror-stylegan2-noaug.pkl)	| 97.72	| 97.46 ± 0.17	| 98.35 ± 0.17	| 89.76	| 89.90 ± 0.31	| 92.51 ± 0.32	|
| stylegan2-ada	[[Karras et al, 2020]](https://arxiv.org/abs/2006.06676) [[ckpt]](https://nvlabs-fi-cdn.nvidia.com/stylegan2-ada-pytorch/pretrained/paper-fig11a-small-datasets/brecahad-mirror-paper512-ada.pkl)	| 15.71	| 15.70 ± 0.06	| 15.63 ± 0.06	| 2.88	| 2.93 ± 0.08	| 3.08 ± 0.08	|

<br>

### MetFaces (Few Shot Generation)
All 1336 images from `train` split are used as the reference images and compared to 50k generated images.
| Model     | Legacy<br>FID<br>(reported) | Legacy<br>FID<br>(reproduced)    | Clean-FID  | Legacy<br>KID<br>(reported)<br>10^3 | Legacy<br>KID<br>(reproduced)<br>10^3    | Clean<br>KID<br>10^3  |
| :---:     | :-:          | :-: | :-: | :-: | :-: | :-: |
| stylegan2	[[Karras et al, 2020]](https://arxiv.org/abs/1912.04958) [[ckpt]](https://nvlabs-fi-cdn.nvidia.com/stylegan2-ada-pytorch/pretrained/paper-fig11a-small-datasets/metfaces-mirror-stylegan2-noaug.pkl)	| 57.26	| 57.36 ± 0.10	| 65.74 ± 0.11	| 35.66	| 35.69 ± 0.16	| 40.90 ± 0.14	|
| stylegan2-ada	[[Karras et al, 2020]](https://arxiv.org/abs/2006.06676) [[ckpt]](https://nvlabs-fi-cdn.nvidia.com/stylegan2-ada-pytorch/pretrained/paper-fig11a-small-datasets/metfaces-mirror-paper1024-ada.pkl)	| 18.22	| 18.18 ± 0.03	| 19.60 ± 0.03	| 2.41	| 2.38 ± 0.05	| 2.86 ± 0.04	|


<br>

### Horse2Zebra (Image to Image Translation)
All 140 images from `test` split are used as the reference images and compared to 120 translated images.
| Model     | Legacy-FID<br>(reported) | Legacy-FID<br>(reproduced)    | Clean-FID  |
| :---:     | :-:          | :-:          | :-:         |
| CUT     [[Part et al, 2020]](https://arxiv.org/abs/2007.15651)| 45.5 | 45.51 | 43.71 |
| FastCUT [[Part et al, 2020]](https://arxiv.org/abs/2007.15651) | 73.4 | 73.38 | 72.53 |

<br>

### Cat2Dog (Image to Image Translation)
All 500 images from `test` split are used as the reference images and compared to 500 translated images.
| Model     | Legacy-FID<br>(reported) | Legacy-FID<br>(reproduced)    | Clean-FID  |
| :---:     | :-:          | :-:          | :-:         |
| CUT     [[Part et al, 2020]](https://arxiv.org/abs/2007.15651) | 76.2 | 76.21 | 77.58 |
| FastCUT [[Part et al, 2020]](https://arxiv.org/abs/2007.15651) | 94.0 | 93.95 | 95.37 |


---


## Citation

If you find this repository useful for your research, please cite the following work.
```
@article{parmar2021cleanfid,
  title={On Buggy Resizing Libraries and Surprising Subtleties in FID Calculation},
  author={Parmar, Gaurav and Zhang, Richard and Zhu, Jun-Yan},
  journal={arXiv preprint arXiv:2104.11222},
  year={2021}
}
```

---

### Related Projects
[torch-fidelity](https://github.com/toshas/torch-fidelity): High-fidelity performance metrics for generative models in PyTorch. <br>
[TTUR](https://github.com/bioinf-jku/TTUR): Two time-scale update rule for training GANs. <br>
[LPIPS](https://github.com/richzhang/PerceptualSimilarity): Perceptual Similarity Metric and Dataset. <br>


### Credits
[PyTorch-StyleGAN2](https://github.com/rosinality/stylegan2-pytorch) ([LICENSE](https://github.com/rosinality/stylegan2-pytorch/blob/master/LICENSE))

[PyTorch-FID](https://github.com/mseitzer/pytorch-fid/) ([LICENSE](https://github.com/mseitzer/pytorch-fid/blob/master/LICENSE))

[StyleGAN2](https://github.com/NVlabs/stylegan2) ([LICENSE](https://nvlabs.github.io/stylegan2/license.html))

converted FFHQ weights: [code](https://github.com/eladrich/pixel2style2pixel) |  [LICENSE](https://github.com/eladrich/pixel2style2pixel/blob/master/LICENSE)
