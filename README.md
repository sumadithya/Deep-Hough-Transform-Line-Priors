# Hold on, I am editing right now...
# Deep-Hough-Transform-Line-Priors 
Official implementation for Deep-Hough-Transform-Line-Priors (ECCV 2020) 
https://arxiv.org/abs/2007.09493

Yancong Lin, and Silvia Laura Pintea, and Jan C. van Gemert

e-mail: y.lin-1ATtudelftDOTnl

Computer Vision Lab

Delft University of Technology, the Netherlands

## Introduction

Classical work on line segment detection is knowledge-based; it uses carefully designed geometric priors using either image gradients, pixel groupings, or Hough transform variants. Instead, current deep learning methods do away with all prior knowledge and replace priors by training deep networks on large manually annotated datasets. Here, we reduce the dependency on labeled data by building on the classic knowledge-based priors while using deep networks to learn features. We add line priors through a trainable Hough transform block into a deep network. Hough transform provides the prior knowledge about global line parameterizations, while the convolutional layers can learn the local gradient-like line features. On the Wireframe (ShanghaiTech) and York Urban datasets we show that adding prior knowledge improves data efficiency as line priors no longer need to be learned from data. Keywords: Hough transform; global line prior, line segment detection.

## Main feature: added Hough line priors

 <img src="ht-lcnn/figs/exp_gt.png" width="180">   <img src="ht-lcnn/figs/exp_input.png" width="180">   <img src="ht-lcnn/figs/exp_iht.png" width="180">   <img src="ht-lcnn/figs/exp_pred.png" width="180"> 
 
 From left to right:  Ground Truth, Input features, HTIHT output and Prediction 
 
## Main contribution: HT-IHT Module
 <img src="ht-lcnn/figs/htiht.png" width="480"> 
 
 The proposed HT-IHT module in our paper
 
 
 ## Main result: imptroved data and parameter efficiency
  <img src="ht-lcnn/figs/sap10.png" width="360">   <img src="ht-lcnn/figs/sap10_pr.png" width="360"> 
  
  <img src="ht-lcnn/figs/sap10_2.png" width="360">   <img src="ht-lcnn/figs/sap10_pr2.png" width="360"> 


## Code Structure

Our implementation is based on the [LCNN](https://github.com/zhou13/lcnn) implementation [https://arxiv.org/abs/1905.03246]. We made minor changes to fit our HT-IHT module. If you are only interested in the HT-IHT module, please check "ht-lcnn/lcnn/models/HT.py". (Thanks Yichao Zhou for such a nice implemtation!)
Below is a quick overview of the function of each file.

```bash
########################### Data ###########################
figs/
data/                           # default folder for placing the data
    wireframe/                  # folder for ShanghaiTech dataset (Huang et al.)
logs/                           # default folder for storing the output during training
########################### Code ###########################
config/                         # neural network hyper-parameters and configurations
    wireframe.yaml              # default parameter for ShanghaiTech dataset
dataset/                        # all scripts related to data generation
    wireframe.py                # script for pre-processing the ShanghaiTech dataset to npz
misc/                           # misc scripts that are not important
    draw-wireframe.py           # script for generating figure grids
    lsd.py                      # script for generating npz files for LSD
    plot-sAP.py                 # script for plotting sAP10 for all algorithms
lcnn/                           # lcnn module so you can "import lcnn" in other scripts
    models/                     # neural network structure
        hourglass_pose.py       # backbone network (stacked hourglass)
        hourglass_ht.py         # backbone network (HT-IHT)
        HT.py                   # the HT-IHT Module
        line_vectorizer.py      # sampler and line verification network
        multitask_learner.py    # network for multi-task learning
    datasets.py                 # reading the training data
    metrics.py                  # functions for evaluation metrics
    trainer.py                  # trainer
    config.py                   # global variables for configuration
    utils.py                    # misc functions
demo.py                         # script for detecting wireframes for an image
eval-sAP.py                     # script for sAP evaluation
eval-APH.py                     # script for APH evaluation
eval-mAPJ.py                    # script for mAPJ evaluation
train.py                        # script for training the neural network
post.py                         # script for post-processing
process.py                      # script for processing a dataset from a checkpoint
```



## Reproducing Results

### Installation

For the ease of reproducibility, you are suggested to install [miniconda](https://docs.conda.io/en/latest/miniconda.html) (or [anaconda](https://www.anaconda.com/distribution/) if you prefer) before following executing the following commands. 

```bash
conda create -y -n ht
source activate ht
conda install -y pytorch cudatoolkit=10.1 -c pytorch
conda install -y tensorboardx -c conda-forge
conda install -y pyyaml docopt matplotlib scikit-image opencv
```

### Pre-trained Models

You can download our reference pre-trained models from [Dropbox] (to do). Use `demo.py`, `process.py`, and
`eval-*.py` to evaluate the pre-trained models.

### Detect Wireframes for Your Own Images
To test on your own images, you need download the pre-trained models and execute

```Bash
python ./demo.py -d 0 config/wireframe.yaml <path-to-pretrained-pth> <path-to-image>
```
Here, `-d 0` is specifying the GPU ID used for evaluation, and you can specify `-d ""` to force CPU inference.

#### Processing the Dataset

```bash
download the dataset into the folder "data", from https://github.com/huangkuns/wireframe 
cd data
tar xf wireframe_raw.tar.xz
rm wireframe_raw.tar.xz
cd ..
dataset/wireframe.py data/wireframe_raw data/wireframe
```

** Recommended** You can also download the pre-processed dataset directly from [LCNN](https://github.com/zhou13/lcnn).

Make sure `curl` is installed on your system and execute
```bash
cd data
../misc/gdrive-download.sh 1T4_6Nb5r4yAXre3lf-zpmp3RbmyP1t9q wireframe.tar.xz
tar xf wireframe.tar.xz
rm wireframe.tar.xz
cd ..
```

### Training
The default batch size assumes your have a graphics card with 12GB video memory, e.g., GTX 1080Ti or RTX 2080Ti. You may reduce the batch size if you have less video memory.

To train the neural network on GPU 0 (specified by `-d 0`) with the default parameters, execute
```bash
python ./train.py -d 0 --identifier baseline config/wireframe.yaml
```

## Testing Pretrained Models
To generate wireframes on the validation dataset with the pretrained model, execute

```bash
./process.py config/wireframe.yaml <path-to-checkpoint.pth> data/wireframe logs/pretrained-model/npz/000312000
```
