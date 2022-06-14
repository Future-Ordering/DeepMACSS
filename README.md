# DeepMACSS
## _Deep Modular Analyzer for Creating Semantics and generate code from Sketch_

[Link to the provided paper](http://urn.kb.se/resolve?urn=urn:nbn:se:ltu:diva-91238)

DeepMACSS is a method which utilizes a modular approach to enable the usage of AI technology to recognize components drawn within a sketch. It utilizes modularity to be able to utilize any object detection algorithm and standalone code generators. More information provided within the paper mentioned above.

## Installation

In order to install the modules required to run the DeepMACSS system you need to have [Python 3.7](https://www.python.org/downloads/) and [pip](https://pip.pypa.io/en/stable/cli/pip_install/) installed.

In order to install the dependencies of DeepMACSS you need to run the following command:
```sh
pip install -r requirement.txt
```

## Data from the paper
The models used and datasets used are located at [this drive.](https://drive.google.com/drive/folders/1WAjuYzrVk1wzmpTcLgdMDlhr9MTBgxTZ?usp=sharing)

### Datasets
There are five datasets present in the drive folder:
* combined, consists of 6250 artificially combined images of the original 13 components.
* combined2semantic, consists of 15 hand drawn images with semantics for each image.
* image, consists of 540 artificially combined images containing a new component Image and the 13 original components.
* mAP, consists of 79 hand drawn images used for the mAP test of the system.
* singular, consists of 1646 hand drawn images with all the unique components each seperatly.

These datasets should be placed in `./datasets/` in order to keep the system clean but can be placed anywhere.

### Models
The models are divided into dates and names. In order to utilize these the models present in the drive shall be placed in `./models/` and can be used in the system. The noteable models which performed best were:
* `Mar-09_13-47` with `All-Comp_v9_SGD-StepLR_pre-10`, which is the network performing best which was trained on the huge 6250 image dataset.
* `Apr-11_10-43` with `Image-Retrain_v9_SGD-StepLR-9`, which is the network containing a newly trained 14th component 'Image'.

## Usage
There are multiple python modules provided in order to utilize the DeepMACSS system's full potential. These modules are called: `dataset.py`, `ai_model.py` and `codegen.py`.
Each of these modules provides a unique part of the DeepMACSS system, ranging from auto-labelling data to generating code from a semantic.

The utilization of each of the modules follows the example below:
```sh
python3 [module.py] [arguments]
```
A help manual describing the arguments can be found for each module by using the argument `-h`.

### Dataset
These are operations made possible by utilizing the `dataset.py` module, these operations are based on operations being made to a provided dataset.

The usage of the dataset module is as depicted below with arguments varying as to which operation is to be performend:
```sh
python3 dataset.py [source_folder] [arguments]
```

#### Auto-labelling
The process of auto-labelling data is made possible by using the argument `--labelling` when utilizing the dataset module.
The arguments which can be used to auto-label data are:
* `--split_ratio [Float between 0 and 1, default 1]`
* `--combine_bbs`

`--split_ratio` is meant to be utilized in order to split the data acquired into two chunks, a validation and a training set. The number provided will determine the portion in which the dataset is to be made up of for the training set. The default value is 1 due to the often usage of wanting to create an artificial combinated dataset.

`--combine_bbs` is meant to be used if the data provided has gaps in its sketch version and describes if the auto-labelling process should combine all found bounding boxes in order to adhear to the posibility of component sketches not colliding.

Examples:
```sh
# Auto-label data normally
python3 dataset.py [source_folder] --labelling

# Auto-label data and split the result into chunks of 80% training and 20% validation
python3 dataset.py [source_folder] --labelling --split_ratio 0.8

# Auto-label data and combine resulting bounding boxes found on each image
python3 dataset.py [source_folder] --labelling --combine_bbs
```
##### Due note that the labeled data provided might have faults since it utilizes the brightness within images and therefor is prone to fault for imagery with shadows. Therefor the dataset obtained should always be post-processed by a human in order to determine its quality.
&nbsp;
#### Creating a artificial combinated dataset
In order to create a artificially combined dataset the argument required is `--combine` when utilizing the dataset module. The creation of a combined dataset is saved in a folder named `{source_folder}_combined_{nr_components}`. Ex: `./Image_combined_4/images/0.jpg`

The arguments which can be used to create artificially combined data are:
* `--nr_components [Number above 1, required]`
* `--start_index [Number equal or above 0, default 0]`
* `--component_folder [Specific folder destination]` 

`--nr_components` specifies how many components which are to be combined together for each image generated. This can range all from 2 to the max number of components the system provides. This argument is required when utilizing the combination part of the dataset module.

`--start_index` specifies which index the names of the combined images are to be named, default is 0. But in order to create larger dataset with varying number of components per image this argument can be changed to create a coherent dataset.

`--component_folder` is utilized when the source folder is to be combined with an external folder containing the components to be combined with. This specifies the location of which files are to be looked at when creating the combined imagery.

Examples:
```sh
# Create a combined dataset containing at most 5 components per image
python3 dataset.py [source_folder] --combine --nr_components 5

# Create a combined dataset starting at index 5 containing at most 3 components per image
python3 dataset.py [source_folder] --combine --nr_components 3 --start_index 5

# Create a combined dataset with an external component folder containing at most 6 components per image
python3 dataset.py  [source_folder] --combine --nr_components 6 --component_folder [component_folder]
```
### AI Engine
The AI engine is the biggest module and its purpose is to train, retrain, predict and evaluate using a AI model. The predefined model is [Faster-RCNN](https://arxiv.org/abs/1506.01497).
#### Train model
In order to train an existing or a new model the argument `--train` needs to be used. This specifies that the arguments parsed is to be utilized in order to train a model.

The arguments which can be used are:
* `--new_model_name [Name of the model to be created, required]` 
* `--epochs [Number of epochs to be trained on, required]` 
* `--dataset [source_folder, required]`
* `--model_name [Name of the model]`
* `--model_date [Date which the model was created]`
* `--num_classes [Number of the amount of classes, default 13]`
* `--combined`

`--new_model_name` describes the name of the model which is to be trained. This is utilized when saving the model for each epoch in order to load a specific version.

`--epochs` determines how many 'evolutions' the model is to be trained for. This number increases the time required for the command to be processed linearly but increases the accuarcy of the model gradually.

`--dataset` describes where the dataset to be utilized in order for the model to learn. This can be hand labeled data or combined data from the previous mentioned module.

`--model_name` and `--model_date` is used to load up a old model to retrain it further. The name of the model is the name in which it was saved as and the date is when the process of training started.

`--num_classes` is used to determine how many components are present in the model to be trained. Default is 13 due to the paper describing 13 unique components but this can be changed in order to extend the system.

`--combined` is used to describe the provided dataset, if the dataset was made from the artificially combined step from before this argument has to be used. This is to not preprocess the images twice and cause issues when predicitng later on.

Examples:
```sh
# Train a new model named Big_Dataset for 10 epochs with combined data
python3 ai_model.py --train --dataset [source_folder] --epochs 10 --new_model_name 'Big_Dataset' --combined

# Retrain a model on a different dataset for 7 epochs with the name BiggerModel
python3 ai_model.py --train --dataset [source_folder] --epochs 7 --new_model_name 'BiggerModel' --model_name 'Image-Retrain_v9_SGD-StepLR-2' --model_date 'Apr-10_16-16'
```
#### Extend model
In order to extend and train a model with a new component the argument `--train_extend` needs to be used.
##### Due note that the file containing the names of each component (./datasets/labels.txt) needs to be updated with the new name of the component in which is to be supported in order for the system to fully function.

The arguments which can be used for extending the model are:
* `--new_model_name [Name of the model to be created, required]` 
* `--epochs [Number of epochs to be trained on, required]` 
* `--dataset [source_folder, required]`
* `--model_name [Name of the model, required]`
* `--model_date [Date which the model was created, required]`
* `--combined` 

`--new_model_name` describes the name of the model which is to be trained. This is utilized when saving the model for each epoch in order to load a specific version.

`--epochs` determines how many 'evolutions' the model is to be trained for. This number increases the time required for the command to be processed linearly but increases the accuarcy of the model gradually.

`--dataset` describes where the dataset to be utilized in order for the model to learn. This can be hand labeled data or combined data from the previous mentioned module.

`--model_name` and `--model_date` describes the model to be used to extend with the new provided component. The name of the model is the name in which it was saved as and the date is when the process of training started.

`--combined` is used to describe the provided dataset, if the dataset was made from the artificially combined step from before this argument has to be used. This is to not preprocess the images twice and cause issues when predicitng later on.


Examples:
```sh
# Extend a model with a new component for 10 epochs
python3 ai_model.py --train_extend --dataset [source_folder] --epochs 10 --new_model_name 'Extended Model' --model_name 'Image-Retrain_v9_SGD-StepLR-2' --model_date 'Apr-10_16-16'
```

#### Predict
In order to utilize the AI engine to make a prediction of an image the argument `--predict` needs to be used.

The arguments which can be used for making a prediction are:
* `--image [source_image, required]`
* `--model_name [Name of the model, required]`
* `--model_date [Date which the model was created, required]`
* `--iou [Float between 0 and 1, default 0.5]` 
* `--min_score [Float between 0 and 1, default 0.8]` 
* `--min_text_score [Float between 0 and 1, default 0.4]` 
* `--destination [destination_folder]` 

`--image` is the location of the image in which the prediction is to be made of.

`--model_name` and `--model_date` describes the model to be used to make the prediction. The name of the model is the name in which it was saved as and the date is when the process of training started.

`--iou` is the percentage of high the intersection over union can be before two predictions are determined to be of the same component.

`--min_score` is the percentage of the minimum confidence the model needs to have for its prediction to be accounted for.

`--min_text_score` is the percentage of the minimum confidence the OCR needs to have for its text prediction to be accounted for.

`--destination` is the location in which the semantics is to be saved in. This is optional and not providing a destination displays the resulting semantics in the console.

Examples:
```sh
# Make a prediction on a image and displaying the semantics to the console
python3 ai_model.py --predict --image [source_image] --model_name 'Image-Retrain_v9_SGD-StepLR-2' --model_date 'Apr-10_16-16'

# Make a prediction on a image and saving the resulting semantics to a file
python3 ai_model.py --predict --image [source_image] --model_name 'Image-Retrain_v9_SGD-StepLR-2' --model_date 'Apr-10_16-16' --destination './website.json'
```
#### Evaluate
In order to determine how good or bad the model is performing objectively the argument `--evaluate` can be used. This causes the model to evaluate its performance using Mean Average Precision as described in more detail in the provided paper.

The arguments which can be used in order to evaluate a model are:
* `--dataset [source_folder, required]`
* `--model_name [Name of the model, required]`
* `--model_date [Date which the model was created, required]`
* `--combined` 
* `--eval_sub_folder [String, default '']`
* `--num_classes [Number of the amount of classes, default 13]` 
* `--map_percentage [Float between 0 and 1, default 0.5]`

`--dataset` describes where the dataset to be utilized in order for the model to be evaluated on. This can be hand labeled data or combined data from the previous mentioned module.

`--model_name` and `--model_date` describes the model which is to be used to evaluate. The name of the model is the name in which it was saved as and the date is when the process of training started.

`--combined` is used to describe the provided dataset, if the dataset was made from the artificially combined step from before this argument has to be used. This is to not preprocess the images twice and cause issues when predicitng later on.

`--eval_sub_folder` is used to describe the structure of the dataset folder provided, default is empty string due to the structure of the provided mAP dataset. This can be used to evaluate using mAP on the validation set of a specific dataset.

`--num_classes` is used to determine how many components are present in the model to be evaluated on. Default is 13 due to the paper describing 13 unique components but this can be changed in order to extend the system.

`--map_percentage` is used to determine the percentage in which the mAP is to be calculated on. The default is 0.5 but any percentage can be used.

Examples:
```sh
# Evaluating a model on mAP@60 (Which means 60% IoU)
python3 ai_model.py --evaluate --dataset [source_folder] --model_name 'Image-Retrain_v9_SGD-StepLR-2' --model_date 'Apr-10_16-16' --map_percentage 0.6

# Evaluating a model with 14 classes and artificially combined data
python3 ai_model.py --evaluate --dataset [source_folder] --model_name 'Image-Retrain_v9_SGD-StepLR-2' --model_date 'Apr-10_16-16' --combined --num_classes 14

# Evaluating a model on a validation dataset containing artificially combined data
python3 ai_model.py --evaluate --dataset [source_folder] --model_name 'Image-Retrain_v9_SGD-StepLR-2' --model_date 'Apr-10_16-16' --combined --eval_sub_folder 'val'
```

### Code-gen Engine
The code generator engine is described by the module `codegen.py`. Its usage is to generate code from a specified semantics and language generator. Each language generator can be created and put into the `./language_generators/` folder.

This module has only required arguments which are:
* `source`
* `language`
* `destination`

`source` is the location of the semantics in which to be utilized in order to generate the code provided.

`language` is the name of the language to be utilized in the generation of the code depicted from the semantics provided.

`destination` is the location in which the code generated by the generator is to be stored.

Example:
```sh
# Generating a React code from a semantics file called website.json and saving it to desktop
python3 codegen.py 'website.json' 'ReactMUI' 'C:/User/foobar/Desktop/main.tsx'
```
