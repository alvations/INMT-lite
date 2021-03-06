
# INMT-lite

<p align="center">
  <img  src="https://i.ibb.co/KzDG7VR/INMT-lite-Preview.png">
</p>

Interactive Neural Machine Translation-lite (INMT-lite) is a framework to train and develop lite versions (.tflite) of models for neural machine translation (NMT) that can be run on embedded devices like mobile phones and tablets that have low computation power and space. The tflite models generated can be used to build the offline version of INMT mobile, a mobile version of [INMT web](https://microsoft.github.io/inmt/).

Table of Contents
=================
  * [Project structure](#project-structure)
  * [Requirements](#requirements)
  * [Quickstart](#quickstart)
  * [INMT-mobile](#INMT-Mobile)
  * [Integration](#Integration)
  * [Hindi-Gondi Parallel Corpus](#Hindi-Gondi-Parallel-Corpus)
  * [Acknowledgements](#acknowledgements)

## Project structure

The project has the following structure:

```
INMT-lite/
├── CODE_OF_CONDUCT.md
├── INMT-lite
│   ├── Android/
│   ├── preprocess.py
│   ├── train.py
│   ├── translate.py
│   └── utils
│       ├── Model_architectures.py
│       └── Model_types.py
├── LICENSE
├── README.md
├── SECURITY.md
└── requirements.txt
```

All the source code and the executable files of the project are present in the `INMT-lite` sub-folder.

Here is more information about all the components that are present in `INMT-lite`:
 - **Android/** - Code and necessary files for building Android app. This can be directly imported into Android Studio.
 - **preprocess.py** - Code for preprocessing the input data for the models.
 - **train.py** - Code for training the models.
 - **translate.py** - Code for performing inference/testing on the trained models.
 - **utils/Model_architectures.py** - Code for defining the architecture of the Encoder and the Decoder blocks.
 - **utils/Model_types.py** - Code for building specific models for translation and partial mode.

Apart from the above, the project creates additional intermediate folders during the course of its run to store model snapshots, optimized tflite model etc. 

Such cases are clearly mentioned at relevant places in the documentation.

## Requirements

Create a separate environment and install the necessary packages using the following command in the root path:

```
pip install -r requirements.txt
```

Please use pip to install the required packages.

## Quickstart

The following section contains the command to quickly get started with the training and generate the tflite file. For full set of features and options, please look into the Documentation section.

### Step 1: Getting your dataset ready

The dataset should be a set of four files one pair for training and one for validation. Both training dataset and validation dataset should consist of two files each, one file consisting of source language sentences separated by an new line and tokens separated by a space and another file for target language in similar format. 

Please make sure your dataset is according to the above mentioned format before heading to further steps.
 
### Step 2: Preprocessing the data

We will be using the example dataset stored in `data/` folder. While inside the root of the project, type in:

```
python preprocess.py --path_src_train ./data/src-train.txt --path_tgt_train ./data/tgt-train.txt  --path_src_val ./data/src-val.txt --path_tgt_val ./data/tgt-val.txt
```

After the command is successfully executed, a set of six files is generated inside `data/` folder:
* `src_train_processed.txt`: Text file consisting tokenized and processed text for `src-train.txt` file.
* `tgt_train_processed.txt`: Text file consisting tokenized and processed text for `tgt-train.txt` file.
* `src_val_processed.txt`: Text file consisting tokenized and processed text for `src-val.txt` file.
* `tgt_val_processed.txt`: Text file consisting tokenized and processed text for `tgt-val.txt` file.
* `src_vocab.json`:  Consists of source language vocabulary in JSON format.
* `tgt_vocab.json`:  Consists of source language vocabulary in JSON format.

### Step 3: Training the model

To train the model on the processed dataset, run the following command in the root of the project:

```
python train.py --to_path_tgt_train ./data/tgt_train_processed.txt --to_path_src_train ./data/src_train_processed.txt --to_path_src_val ./data/src_val_processed.txt --to_path_tgt_val ./data/tgt_val_processed.txt --to_path_src_vocab ./data/src_vocab.json --to_path_tgt_vocab ./data/tgt_vocab.json
```

The command just takes the processed dataset files and json files for vocabulary of both the languages. The model has a encoder decoder architecture and uses attention. The model uses GRU with 500 units on both the encoder and decoder side and would run for 10 epochs. The model configuration such as the number of hidden units can be changed. For more information, please look into detailed documentation.

This command will generate a model config file under `model/` directory as `model/model_config.json` and a tflite file under the `tflite/` directory as `tflite/tflite_model.tflite`. Also, after every 2 epochs a checkpoint file would be saved under `model/` folder. This setting also can be changed.

By default, GPU hardware acceleration would not be used. To use the GPU, please pass in the argument: `--use_gpu true`. 

### Step 4: Testing, Inference and further

Both testing and inference are handled by `translate.py` file. It supports three modes testing, inference and inline.

#### Testing

For testing the model's performance please use the following command while in root of the project:

```
python translate.py --src_path data/src-val.txt --tgt_path data/tgt-val.txt --src_vocab_path data/src_vocab.json --tgt_vocab_path data/tgt_vocab.json --model_path model/model_weight_epoch_10.h5 --model_config_path ./model/model_config.json --mode test​​​​​​
```

After the successful run, a line would be printed reporting the test score for the model on the given dataset. The test score can be interpreted as the average words predicted correctly by the model.

The command takes in the following inputs - two text files (txt files consisting of sentences for each source and target language), a model_config file consisting information about the model, trained model's path, JSON files consisting of vocabulary of the languages and mode type for the command.

#### Inference

For getting the translations of a set of sequences written in a txt file. Use the following command while inside the root of the project.

```
python translate.py --src_path data/src-val.txt --tgt_path data/ --src_vocab_path ./data/src_vocab.json --tgt_vocab_path ./data/tgt_vocab.json --model_path ./model/model_weight_epoch_10.h5 --model_config_path ./model/model_config.json --mode inference
```

After the successful run a txt file consisting of translated sequences would be generated under `data/` folder saved as `data/inference.txt`

The command takes in the following inputs - a txt file consisting of sequences in source language, a target directory path where the inference file should be saved, the vocabulary paths of both the languages, trained model's path, a model configuration path and mode type for the command.

#### Inline

The framework also supports translating a sequence in command line itself for quick inference. For inline inference, type in the following command while in root of the project:

```
python translate.py --sentence <SENTENCE_TO_TRANSLATE> --src_vocab_path ./data/src_vocab.json --tgt_vocab_path ./data/tgt_vocab.json --model_path ./model/model_weight_epoch_10.h5 --model_config_path ./model/model_config.json --mode inline
``` 

After the successful run,  the translated sentence would be printed in the terminal itself.

Please note that the inference in all modes does not consist of `<start>` token.

## INMT Mobile

The following section is about Interactive Neural Machine Translation (INMT) mobile, a mobile version of INMT-web. This app consists of both the offline and online mode. The offline mode uses the tflite file generated by INMT-lite whereas the online mode uses INMT-web REST APIs to bring in the results. The android code can be found at `./Android` folder. This codebase provides a tempalte implementation of INMT mobile app which can be modified and integrated to add interactive machine translation compabilities in your existing or new Android app.

### Online Mode

The Online mode of INMT mobile calls the APIs of INMT web for the partial translation provided by the user as she types in the content. This mode supports multiple language pair translations that are supported by INMT-web and can be easily extended for new language pairs. 

Steps for configuring the online mode:
  1. Since we shall be using the services of INMT-web, we need to deploy it to use the APIs. To know more about INMT-web and how to host it, please refer [here](https://github.com/microsoft/inmt).  
  2. Once deployed, enter the base URL of the deployed project in the android's string resource file located here `app/src/main/res/values/strings.xml`. An example can be: `https://localhost:8000/`
  3. Build the android project and ensure that the device on which the android application runs is in the same network in which INMT-web is deployed.

### Offline Mode

The Offline mode uses the tflite model as generated by tflite framework. The model when generated by the default settings, is usually of the size < 80 MB. The translation predictions may not be as accurate as INMT-web because of the portability. However, INMT mobile offline mode doesn't need internet connection and API calls for translation predictions.

Steps for configuring offline mode:
1. This step assumes that the tflite model and the vocabulary files are availble. You can either get ana trained Hindi-Gondi model and vocabulary files from [here](https://microsoftapc-my.sharepoint.com/:f:/g/personal/taganu_microsoft_com/El5qZjb_teZJqsHyE5Cyh1kB2QCTMwot8E7wGYpWCi0BQA?e=zEo7bq) or train your own custom model following steps mentioned in [Quickstart](#quickstart) to generate the files.
2. Under the Android folder, place your files in asset folder (`/app/src/main/assets`)
3. Update the Vocabulary files name, model name, language pair name and other options in the string resource file located at `app/src/main/res/values/strings.xml`. The file contains clear commented out instructions.
4. Build the project.

## Integration

The application allows seamless integration with other apps and can be connected with any other android project quickly.

Following steps are to be followed while integrating INMT-mobile:

1. Send an intent to `WelcomePage` activity to start the activity. WelcomePage Activity is the default acitvity (first page) of INMT-mobile and can be found at `app/src/main/java/com/example/inmt_offline/UI/WelcomePage.java`.
2. Write your custom code in `Preview_Button.java` file located at `app/src/main/java/com/example/inmt_offline/External/Preview_Button.java`. Specifically, the method `onClick` provides the implementation for what happens when Preview Button is clicked. The method receives translation pairs list along with other arguments and also provides a deafault implementation for reference.
3. Update `BASE_URL` string resource in case, the online mode of INMT-mobile is expected to reach out to any other API for online translation.

The `PreviewActivity` located at `app/src/main/java/com/example/inmt_offline/UI/PreviewActivity.java` can be removed in case there is a custom implementation of `onClick` method at `Preview_Button.java`, since it is a supplementary activity to default implemetation.

## Hindi-Gondi Parallel Corpus
INMT was developed to help expand the digital datasets for low resource languages and further support in developing other language tools for such low resource languages. The [sample model](https://microsoftapc-my.sharepoint.com/:f:/g/personal/taganu_microsoft_com/El5qZjb_teZJqsHyE5Cyh1kB2QCTMwot8E7wGYpWCi0BQA?e=zEo7bq) in this codebase is trained on  the first-ever Hindi-Gondi Parallel Corpus released by [CGNet Swara](http://cgnetswara.org/) which can be found [here](http://cgnetswara.org/hindi-gondi-corpus.html). 

# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Acknowledgements

INMT-lite is a collaborative project developed at Microsoft Research.
Major contributors are:

 - [Anurag Shukla](https://github.com/anuragshukla06)
 - [Mohd Sanad Zaki Rizvi](https://github.com/mohdsanadzakirizvi)
 - Tanuja Ganu
 - Kalika Bali
 - Sebastin Santy
 - Monojit Choudhury
 - Sandipan Dandapat
