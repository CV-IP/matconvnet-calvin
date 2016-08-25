# MatConvNet-Calvin v0.4

![Example output of our E2S2 method](http://groups.inf.ed.ac.uk/calvin/caesar16eccv/Examples/street_hexp30.png)

**MatConvNet-Calvin** is a wrapper around MatConvNet that (re-)implements
several state of-the-art papers in object detection and semantic segmentation. This includes our own work "Region-based semantic segmentation with end-to-end training" \[5\]. Calvin is a Computer Vision research group at the University of Edinburgh (http://calvin.inf.ed.ac.uk/). Copyrights by Holger Caesar and Jasper Uijlings, 2015-2016.

## Overview
- [Methods](#methods)
- [Dependencies](#dependencies)
- [Installation](#installation)
- [Instructions](#instructions)
- [References](#references)
- [Disclaimer](#disclaimer)
- [Contact](#contact)

## Methods
- **Fast R-CNN (FRCN)** \[1\]: State-of-the-art object detection method. The original code was implemented for Caffe. This reimplementation ports it to MatConvNet and includes Region of Interest pooling and a simplified version of bounding box regression.
- **Fully Convolutional Networks (FCN)** \[2\]: Very successful semantic segmentation method that builds the basis for most modern semantic segmentation methods. FCNs operate directly on image pixels, performing a series of convolutional, fully connected and deconvolutional filters. This implementation is based on MatConvNet-FCN and is modified to work with arbitrary datasets.
- **Multi-Class Multipe Instance Learning** \[3\]: Extends FCNs for weakly supervised semantic segmentation. We also implement the improved loss function of "What's the point" \[4\], which takes into account label presence and absence.
- **Region-based semantic segmentation with end-to-end training (E2S2)** \[5\]: State-of-the-art semantic segmentation method that brings together the advantages of region-based methods and end-to-end trainable FCNs. This implementation is based on our implementation of Fast R-CNN and includes the freeform Region of Interest pooling and region-to-pixel layers.

## Dependencies
- **MatConvNet:** beta20 (http://github.com/vlfeat/matconvnet)
- **MatConvNet-FCN:** (http://github.com/vlfeat/matconvnet-fcn)
- **Selective Search:** for Fast R-CNN and E2S2 (http://koen.me/research/selectivesearch/)
- **Datasets:** 
  - **SIFT Flow:** (http://www.cs.unc.edu/~jtighe/Papers/ECCV10/)
  - **PASCAL VOC 2010:** (http://host.robots.ox.ac.uk/pascal/VOC/voc2010/)

## Installation
- Install Matlab R2015a (or newer) and Git
- Clone the repository and its submodules from your shell
  - `git clone https://github.com/nightrome/matconvnet-calvin.git`
  - `cd matconvnet-calvin`
  - `git submodule update --init`
- Execute the following Matlab commands
  - Setup MatConvNet
    - `cd matconvnet/matlab;`
    - `vl_compilenn('EnableGpu', true);`
    - `cd ../..;`
  - Setup MatConvNet-Calvin
    - `cd matconvnet-calvin/matlab;`
    - `vl_compilenn_calvin();`
    - `cd ../..;`
  - Add files to Matlab path
    - `setup();`
- (Optional) Download pretrained models from your shell
    - FRCN: `wget -N -P data/ http://groups.inf.ed.ac.uk/calvin/caesar16eccv/FRCN_VOC2010_model.zip;  unzip data/FRCN_VOC2010_model.zip`
    - FCN: `wget -N -P data/ http://groups.inf.ed.ac.uk/calvin/caesar16eccv/FCN_SiftFlow_model.zip;  unzip data/FCN_SiftFlow_model.zip`
    - E2S2 (Full): `wget -N -P data/ http://groups.inf.ed.ac.uk/calvin/caesar16eccv/E2S2_SiftFlow_model-full.zip; unzip data/E2S2_SiftFlow_model-full.zip`
    - E2S2 (Fast): `wget -N -P data/ http://groups.inf.ed.ac.uk/calvin/caesar16eccv/E2S2_SiftFlow_model-fast.zip; unzip data/E2S2_SiftFlow_model-fast.zip`

## Instructions
1) **FRCN**
- **Usage:** Run `demo_frcn()`
- **What:** This trains and tests Fast R-CNN using VGG-16 for object detection on PASCAL VOC 2010. The parametrization of the regressed bounding boxes is slightly simplified, but we found this to make no difference in performance.
- **Model:** Training this model takes about 8h on a Titan X GPU. If you just want to use it you can download the pretrained model in the installation step above. Then run the demo to see the test results.
- **Results:** If the program executes correctly, it will print the per-class results in average precision and their mean (mAP) for each of the 20 classes in PASCAL VOC. The example model achieves 63.5% mAP on the validation set using no external training data.

2) **FCN**
- **Usage:** Run `demo_fcn()`
- **What:** This trains and tests an FCN-16s network based on VGG-16 for semantic segmentation on the SIFT Flow dataset. The performance varies a bit compared to the implementation of \[2\], as they first train FCN-32s and use it to finetune FCN-16s. Instead we directly train FCN-16s. For weakly \[3,4\] and semi supervised training, see the options in fcnTrainGeneric().
- **Model:** Training this model takes about 12h on a Titan X GPU. If you just want to use it you can download the pretrained model in the installation step above. Then run the demo to see the test results.
- **Results:** If the program executes correctly, it will print the semantic segmentation performance. It will also show an image from the SIFT Flow dataset, the ground-truth labels, the output labeling and an image that shows the different types of error (similar to the figure above). The example model achieves 83.8% Global Accuracy, 48.8% Class Accuracy and 36.7% Mean IOU. 

3) **E2S2**
- **Usage**: Run `demo_e2s2()`
- **What:** This trains and tests a region-based end-to-end network based on VGG-16 for semantic segmentation on the SIFT Flow dataset. The script automatically extracts Selective Search region proposals from the dataset. All networks are trained with an inverse-class frequency weighted loss (Sect. 3.4 of \[5\]).
- **Model:** Training this model takes a lot of time. If you just want to use it you can download the pretrained model in the installation step above. Then run the demo to see the test results. There are two different models available:
  - Full: Our best performing model refered to in \[5\] as "separate weights".
  - Fast: A much faster model refered to in \[5\] as "tied weights".
- **Results:** If the program executes correctly, it will print the semantic segmentation performance. It will also show an image from the SIFT Flow dataset, the ground-truth labels, the output labeling and an image that shows the different types of error (similar to the figure above). The results and timings for the two models can be seen in the table below. Both were trained on a Titan X GPU with 12GB of RAM.

 | Model        | Class accuracy | Training epochs | Training time | GPU RAM
 | ---          | ---            | ---             | ---           | ---
 | Full         | 66.1%          | 25              | 75h           | 8.5GB
 | Fast         | 62.5%          | 10              | 20h           | 6.0GB

## References
- \[1\] **Fast R-CNN (FRCN)** by Girshick et al., ICCV 2015, http://arxiv.org/abs/1504.08083
- \[2\] **Fully Convolutional Networks for Semantic Segmentation (FCN)** by Long et al., CVPR 2015, http://arxiv.org/abs/1411.4038
- \[3\] **Fully Convolutional Multi-Class Multipe Instance Learning** by Pathak et al., ICLR 2015 workshop, http://arxiv.org/abs/1412.7144
- \[4\] **What's the point: Semantic segmentation with point supervision** by Bearman et al., ECCV 2016, http://arxiv.org/abs/1506.02106
- \[5\] **Region-based semantic segmentation with end-to-end training** (E2S2) by Caesar et al., ECCV 2016, http://arxiv.org/abs/1607.07671

## Disclaimer
Except for \[5\], none of the methods implemented in MatConvNet-Calvin is authorized by the original authors. These are (possibly simplified) reimplementations of parts of the described methods and they might vary in terms of performance. This software is covered by the FreeBSD License. See LICENSE.MD for more details.

## Contact
If you run into any problems with this code, please submit a bug report on the Github site of the project. For other inquiries contact holger.caesar-at-ed.ac.uk.
