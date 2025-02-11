# HiDDeN
Pytorch implementation of paper "HiDDeN: Hiding Data With Deep Networks" by Jiren Zhu*, Russell Kaplan*, Justin Johnson, and Li Fei-Fei: https://arxiv.org/abs/1807.09937  
*: These authors contributed equally

The authors have Lua+Torch implementation here: https://github.com/jirenz/HiDDeN

I have tested this on Pytorch 1.0. Note that this is a work in progress, and I was not yet able to fully reproduce the results of the original paper.

## Requirements

You need Pytorch 1.0 with TorchVision to run this, and matplotlib.
If you want to use Tensorboard, you need to install TensorboardX and Tensorboard. This allows to use a subset of Tensorboard functionality to visualize the training. However, this is optional.
The code has been tested and runs on Ubuntu 16.04 and Windows 10. 

## Data

We use 10,000 images for training and 1,000 images for validation. Following the original paper, we chose 
those 10,000 + 1,000 images randomly from one of the coco datasets.  http://cocodataset.org/#download

The data directory has the following structure:
```
<data_root>/
  train/
    train_class/
      train_image1.jpg
      train_image2.jpg
      ...
  val/
    val_class/
      val_image1.jpg
      val_image2.jpg
      ...
```

```train_class``` and ```val_class``` folders are so that we can use the standard torchvision data loaders without change.

## Running

You will need to install the dependencies, then run 
```
python main.py --data-dir <data_root> --batch-size <b>
```
By default, tensorboard logging is enabled. To use this, you need to install Tensorboard and TensorboardX. 
However, if you don't want to use Tensorboard, you don't need to install Tensorboard/TensorboardX. Simply run with the 
```--no-tensorboard``` switch.

There are additional parameters for main.py. Use
```
python main.py --help
```
to see the description of all of the parameters.

Each run creates a folder in ./runs/<date-and-time> and stores all the information about the run in there.


### Running with Noise Layers
You can specify noise layers configuration. To do so, use the ```--noise``` switch, following by configuration of noise layer or layers.
For instance, the command 
```
python main.py --no-tensorboard --data-dir /data/ --batch-size 12 --noise  'crop((0.2, 0.3), (0.4, 0.5)) + cropout((0
.11, 0.22), (0.33, 0.44)) +dropout(0.2, 0.3) + jpeg()'
```
runs the training with the following noise layers applied to each watermarked image: crop, then cropout, then dropout, then jpeg compression. The parameters of the layers are explained below. **It is important to use the quotes around the noise configuration.** If you want to stack several noise layers, specify them using + in the noise configuration, as shown in the example. 

### Noise Layer paremeters
* _Crop((height_min, height_max), (width_min, width_max))_, where **_(height_min, height_max)_** is a range from which we draw a random number and keep that fraction of the height of the original image. **_(width_min, width_max)_** controls the same for the width of the image. 
Put it another way, given an image with dimensions **_H x W,_** the Crop() will randomly crop this into dimensions **_H' x W'_**, where **_H'/H_** is in the range **_(height_min, height_max)_**, and **_W'/W_** is in the range **_(width_min, width_max)_**. In the paper, the authors use a single parameter **_p_** which shows the ratio **_(H' * W')/ (H * W)_**, i.e., which fraction of the are to keep. In our setting, you can obtain the appropriate **_p_** by picking **_height_min_**, **_height_max_**  **_width_min_**, **_width_max_** to be all equal to **_sqrt(p)_**
*  _Cropout((height_min, height_max), (width_min, width_max))_, the parameters have the same meaning as in case of _Crop_. 
* _Dropout_(keep_min, keep_max)_ : where the ratio of the pixels to keep from the watermarked image, **_keep_ratio_**, is drawn uniformly from the range **_(keep_min, keep_max)_**.
* _Jpeg_ does not have any parameters. 


## Experiments
The data for some of the experiments are stored in 'HiDDeN/experiments/<name of the experiment> folder. This includes: figures, detailed training and validation losses, all the settings in *pickle* format, and the checkpoint file of the trained model. Here, we provide summary of the experiments.

### Setup.
We try to follow the experimental setup of the original paper as closely as possibly.
We train the network on 10,000 random images from [COCO dataset](http://cocodataset.org/#home). We use 200-400 epochs for training and validation.
The validation is on 1,000 images. During training, we take randomly positioned center crops of the images. This makes sure that there is very low chance the network will see the exact same cropped image during training. For validation, we take center crops which are not random, therefore we can exactly compare metrics from one epoch to another. 

Due to random cropping, we observed no overfitting, and our train and validation metrics (mean square error, binary cross-entropy) were extremely close. For this reason, we only show the validation metrics here. 

When measuring the decoder accuracy, we do not use error-correcting codes like in the paper. We take the decoder output, clip it to range [0, 1], then round it up. We call this "Decoder bitwise error". We also report mean squate error of of the decoder for consistency with the paper.


### Default settings, no noise layers

This experiment is with default settings from the paper, and no noise layers.

|Experiment name | Combined loss  | Encoder MSE    | Decoder bitwise error  | Decoder MSE |     Epochs     |
|----------------|----------------|----------------|------------------------|-------------|----------------|
| No noise       |0.0143          | 0.0021         | 0.0007                 | 0.0112      | 200            |


