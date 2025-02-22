# ENDG511 Project

This repository shows the implementation of the concepts TinyAI and FastAI using
PyTorch, Tensorflow, and YoloV5 to continue the work behind 
*Radar Object Detection [RODNet](https://github.com/yizhou-wang/RODNet).* and aswell
as provide performance comparison between radar-based object detection and 
conventional camera-based object detection in cases of low visibility.

# Overview

- [Background](#background)
- [Setup](#setup)
- [Radar-Based Object Detection](#radar-based-object-detection)
    - [Installation Instructions](#installation-instructions)
        - [Cloning RODNet and CRUW Repositories](#cloning-rodnet-and-cruw-repositories)
        - [Create a New Conda Environment](#create-a-new-conda-environment)
    - [Downloading the Dataset](#downloading-the-dataset)
    - [Configuration File](#configuration-file)
    - [Processing the Dataset](#processing-the-dataset) 
    - [Running the Jupyter Notebook](#running-the-final_projectipynb-notebook)
    - [Python File Descriptions](#trainmodelpy-and-getmodelpy)
    - [RODNet Challenges](#challenges)
- [Camera-Based Object Detection](#camera-based-object-detection)
    - [Installation Instructions](#installation-instructions-1)
    - [Running the Jupyter Notebook](#running-the-final_projectipynb-notebook-1)
    - [Camera-Based Detection Files](#camera-based-detection-files)
- [References](#references)

# Background

Please see /docs/ENDG511_project_report_radar.pdf for the problem/use-case and
more details regarding the project.

# Setup

The following steps shows setting up the project repository.

1) Run the following command to clone the project repository.

```shell
git clone https://github.com/ENELEngineering/ENDG511_Project_RODNet.git
```

2) Upgrade pip with the following command.

```shell
python.exe -m pip install --upgrade pip
```

# Radar-Based Object Detection

## Installation Instructions

### Cloning RODNet and CRUW Repositories
- Change the current working directory to "radar_detections" with this command.

```shell
cd radar_detections
```

- Clone the RODNet repository with this command 

```shell
git clone https://github.com/yizhou-wang/RODNet.git
```

- Clone the cruw-devkit repository with this command. 

```shell
git clone https://github.com/yizhou-wang/cruw-devkit.git
```

- In case it is not installed, [install Anaconda](https://docs.anaconda.com/free/anaconda/install/index.html).

### Create a New Conda Environment
- Run the following lines of code in the terminal to setup the RODNet environment with the required dependencies.

```shell
conda create -n rodnet python=3.11 -y
conda activate rodnet
```

For MACOS, install PyTorch using the following command. 

```shell
pip3 install torch torchvision torchaudio
```

For Windows, install PyTorch with CUDA [only for machine with recognized NVIDIA GPUs]
following this [guideline](https://medium.com/@harunijaz/a-step-by-step-guide-to-installing-cuda-with-pytorch-in-conda-on-windows-verifying-via-console-9ba4cd5ccbef). 

Other alternate installations include the following command.

```shell
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

The following versions were used in a Window machines during testing.

* Python: 3.11.8
* Cuda: 11.8
* torch: 2.2.1

Verify that Cuda is installed and PyTorch recognizes this installation using the command below.

```python
>> print(torch.cuda.is_available())
>> True
```

Run the rest of the installation commands below to complete the setup.

```shell
pip install -e .
cd cruw-devkit
pip install .
cd ..
pip install numpy
```
    
## Downloading the Dataset
- Use the following google drive link to download the "ROD2024" folder as a subdirectory under "radar_detections": https://drive.google.com/drive/folders/1XXXKaU6_MAtqp9imyqpOEVu2vbvuqaCn?usp=sharing
- Move the downloaded ROD2024 folder inside the "radar_detections" main folder

## Configuration File
The python file *config_rodnet_cdc_win16.py* is the file that contains the path to the 
dataset folder and other relevant variables, such as the number of epochs and 
batch size required by the custom CRUW dataset.

Ensure that the paths and configurations present in this file are correct. However, by following
the instructions provided shouldn't make it necessary to make changes in this file.

```python
dataset_cfg = dict(
    dataset_name='ROD2021',
    base_root="path to the base directory of the dataset",
    data_root="path to the sequences directory",
    anno_root="path to the annotations directory",
    anno_ext='.txt',
    train=dict(
        subdir='train',
        # seqs=[],  # can choose from the subdir folder
    ),
    valid=dict(
        subdir='valid',
        seqs=[],
    ),
    test=dict(
        subdir='test',
        # seqs=[],  # can choose from the subdir folder
    ),
    demo=dict(
        subdir='demo',
        seqs=[],
    ),
)
```

```python
train_cfg = dict(
    n_epoch=5,
    batch_size=4,
    lr=0.00001,
    lr_step=5,  # lr will decrease 10 times after lr_step epoches
    win_size=16,
    train_step=1,
    train_stride=4,
    log_step=100,
    save_step=10000,
)
```

## Processing the Dataset
- change the current working directory to "RODNet".

- Run the follow line of code in the terminal.

```shell
python tools/prepare_dataset/prepare_data.py --config ../config_rodnet_cdc_win16.py --data_root ../ROD2024 --split train --out_data_dir ../data_final_converted
```

## Running the final_project.ipynb Notebook
This is the main jupyter notebook used for the radar object detection.

- There are two ways to run this script:

    - Running the entire script which involves training and validation.
        - In the section Model Validation/testing. 
        Comment the 3 and 4 statments and either uncomment 1 or 2 depending on the avaibility of GPU. 
        For base model (rod_v0) and for the multibranch model (rod_v1).

    - Only running the validation and using the provided trained models as .pkl files
        - Download the "trained_models" folder containing .pkl files as a subdirectory 
        under "radar_detections": https://drive.google.com/drive/folders/1GW93bPf7UZ-OhiEhrsuTuxwWYB4yqbbL?usp=sharing 
        - Skip the "Model training" section by not running the 8 cells inside it.
        - In the section Model Validation/testing. Comment the 1 and 2 statments and either uncomment 3 or 4 depending on the avaibility of GPU. 
        For base model (rod_v0) and for the multibranch model (rod_v1).

## trainModel.py and getModel.py
The trainModel.py contains the function declaration for the training schedule. 
And getModel.py contains the class definition for the base model and the multibranch model.

## Challenges

As described in the project report, there were challenges that was encountered during the 
pruning and quantization of the base RODNet architecture using PyTorch. The source
code showing these efforts are under `/radar_detections/pytorch_challenges`. 

The subdirectory `/primary_efforts` shows the initial efforts or attempts at pruning 
and quantizing the RODNet model.

The subdirectory `/secondary_efforts` shows the second attempt at pruning and 
quantizing the RODNet model.

Our future work will also
involve revisiting this effort in hopes to modify to perform proper pruning and 
quantization of the model that yeilds expected results. 

# Camera-Based Object Detection

## Installation Instructions 

1) Change directory to the camera-based detection portion of the project.

```shell
cd camera_detections
```

2) If an environment is not currently activated, do so by setting up a python virtual environment.

```shell
python -m venv env
```

Activate the environment with windows using the command.

```shell
<path to the environment>\Scripts\activate
```

Activate the environment with a Linux interface using the command.

```shell
source <path to the environment>\bin\activate
```

3) Install the requirements using the command below.

```shell
pip install -r requirements.txt
```

## Running the final_project.ipynb Notebook

This is the main jupyter notebook used for the camera object detection.

The instructions or process for running the script is already provided in the script.

## Camera-Based Detection Files

The link that stores the saved models, results, and dataset used can be found
[here](https://drive.google.com/drive/folders/1pXwiYTeBuV-cPwFCOQ6quV4JmPmkjDvc?usp=sharing).

# References

* The work done in this repository [RODNet](https://github.com/yizhou-wang/RODNet/tree/master) was continued in this project to implement multibranching, pruning, quantization, and clustering methods.

* [YoloV5](https://github.com/ultralytics/yolov5) was used to train the camera-based detection models.

* [Deepview-Validator](https://support.deepviewml.com/hc/en-us/articles/11511279358221-How-to-Evaluate-Computer-Vision-Models-with-Deepview-Validator) was used to validate the camera-based detection models.