# 2IOI0 DBL Process Mining Prediction Tool (Group 8)

This README is best viewed at the following url:
https://github.com/Rickerd1234/2IOI0-README

## Contents
<ul>
  <li><a href="#about-the-tool">About the Tool</a></li>
  <li><a href="#datasets">Datasets</a></li>
  <li><a href="#example">Example</a></li>
  <li><a href="#using-the-tool">Using the Tool</a></li>
  <li><a href="#parameters">Parameters</a></li>
  <li><a href="#predictors">Predictors</a></li>
  <li><a href="#preprocessing">Preprocessing</a></li>
  <li><a href="#on-the-fly">On-The-Fly</a></li>
  <li><a href="#error-measurements">Error Measurements</a></li>
</ul>


## About the Tool
For the course 2IOI0 DBL Process Mining, the objective was to develop a prediction tool. The software should be able to predict both the next event and the next timestamp for a given event within a process. The requirements for the final tool were an implementation of a naïve predictor and at least two different predictors. Our final prototype includes a basic Naïve Predictor, Subseq Predictor, Random Forest Predictor and Transition Matrix Predictor. Each of these predictors are implemented for both activity and time predictions, you can read more about these predictors in their respective sections. Besides these predictors, our tool uses preprocessing to prepare the data, this is described in detail in the Preprocessing section. This README does not only explain the usage of the tool, but also describes the way it works for some parts. This helps the user to make good decisions on what parameters/settings to use.


## Datasets
Throughout the course, we have used 3 different datasets to test the software. These datasets were the following:
<ul>
	<li><a href=https://data.4tu.nl/articles/dataset/Road_Traffic_Fine_Management_Process/12683249/1>Road Traffic</a></li>
	<li><a href=https://data.4tu.nl/articles/dataset/BPI_Challenge_2012/12689204/1>BPI Challenge 2012</a></li>
	<li><a href=https://data.4tu.nl/articles/dataset/BPI_Challenge_2017/12696884/1>BPI Challenge 2017</a></li>
</ul>
We converted these to two separate csv files using the provided xes2csv.jar, which splits the data into a training and test dataset (using an 80/20 split on the cases).

## Example
This example will walk you through several steps of our software. For this example, the BPI Challenge 2012 data will be used, as this was also mostly used during testing. The following walkthrough shows what happens when you run the tool over the BPI Challenge 2012 data, which was split in training and test data, with the -r, -l and -write parameters, to run both the Random Forest and Transition Matrix Predictors and to store the naïve model. 

The initial cases of the datasets are as follows:
<details> 
  <summary>Show/hide image</summary>
   <img src="/lib/train-test-initial.png" title="Training and Test data">
Green represents the used training data and the blue cases are the test data. The vertical axis separates all the distinct cases, whereas the horizontal axis displays the date of each event within a case. This image displays 10469 training cases and 2618 test cases.
</details>

After temporal separation on the training and test data:
<details> 
  <summary>Show/hide image</summary>
   <img src="/lib/train-test-temporal.png" title="Temporally Separated Training and Test data">
Green represents the used training data and the blue cases are the test data. The vertical axis separates all the distinct cases, whereas the horizontal axis displays the date of each event within a case. As you can see, we have dropped values from the training data near the test data. At first glance it might look like we have only dropped specific events. However, by looking at the number of cases, we see a decrease, so we have dropped entire cases that overlapped with the test data. This image displays 9624 training cases and 2618 test cases.
</details>

After all the <a href="#preprocessing">preprocessing</a>, the dataset will look as follows:
<details> 
  <summary>Show/hide image</summary>
   <img src="/lib/train-validation-test.png" title="Train, Validation and Test split">
Green represents the used training data, red corresponds with the validation data and the blue cases are the test data. As you can see, <a href="#temporal-separation">temporal separation</a> has filtered cases that would overlap with test and validation data. Although it looks like it only dropped the last few events of some cases, if we look at the number of cases, we clearly see that we have dropped cases. This image displays 7089 training cases, 1866 validation cases and 2618 test cases.
</details>

After all of the preprocessing, we start computing the extra columns that are required for some predictors. After these columns are added to all of the data (training, validation and test), we will start making our predictions.
<br><br>
Firstly, we compute the naïve predictions. This model uses the mode (most common value) of both events and time bins for each index within a case. This model will simply apply the mode for each event index in a case. As specified by the "-write" parameter, we want to store the model. For this we cannot just store all the most common events and bins. To make sure the model can be extended, we need to keep track of all counts per index within a case, so we can update the mode if another event is more common in the combined models.

The stored naïve model (NaïveModel.json) should look similar to:<br>
```json
{
	"activity": [
		{"A_SUBMITTED": 7089},
		{"A_PARTLYSUBMITTED": 7089},
		{"W_Afhandelen leads": 2622, "A_PREACCEPTED": 2398, "A_DECLINED": 2030, "W_Beoordelen fraude": 39},
		...
		{"O_ACCEPTED": 1},
		{"W_Valideren aanvraag": 1}
	],

	"time": [
		{"0": 6859, "5": 217, "15": 13},
		{"35": 5236, "15": 943, "75": 449, "5": 415, "0": 45, "155": 1},
		...
		{"0": 1},
		{"5": 1}
	]
}
```

After computing the naïve predictions, we compute the random forest predictions and lastly the transition matrix predictions. All of the prediction models are applied to both the validation and the test data.
<br><br>
When we have computed all the columns and predictions, we compute the error measures for the used predictions. These measures show the quality of the predictions, for this particular dataset.
For the described run, the resulting error measures should be similar to:
<details> 
  <summary>Show/Hide Error Measures</summary>
<pre>
Naïve time prediction:

Scores on test data:
                            Seconds
Accuracy               2.710000e-01
Standard Deviation     1.280262e+05
MSE                    1.743268e+10
RMSE                   1.320329e+05
R^2                   -6.400000e-02
Explained Variance    -0.000000e+00
Max Error              2.621435e+06
F Score (macro)        2.400000e-02
Average RMSE per Case  6.605197e+04

Scores on validation data:
                            Seconds
Accuracy               2.720000e-01
Standard Deviation     9.350087e+04
MSE                    9.176464e+09
RMSE                   9.579386e+04
R^2                   -5.000000e-02
Explained Variance    -0.000000e+00
Max Error              1.310715e+06
F Score (macro)        2.600000e-02
Average RMSE per Case  3.315939e+04

Random Forest time prediction:

Scores on test data:
                            Seconds
Accuracy               4.650000e-01
Standard Deviation     1.269863e+05
MSE                    1.739938e+10
RMSE                   1.319067e+05
R^2                   -6.200000e-02
Explained Variance    -6.100000e-02
Max Error              2.621360e+06
F Score (macro)        2.250000e-01
Average RMSE per Case  6.294552e+04

Scores on validation data:
                            Seconds
Accuracy               4.900000e-01
Standard Deviation     9.727132e+04
MSE                    1.005742e+10
RMSE                   1.002867e+05
R^2                   -1.500000e-01
Explained Variance    -1.430000e-01
Max Error              1.310710e+06
F Score (macro)        2.570000e-01
Average RMSE per Case  3.442639e+04
    
LTS time prediction

Scores on test data:
                            Seconds
Accuracy               3.930000e-01
Standard Deviation     1.280291e+05
MSE                    1.743259e+10
RMSE                   1.320325e+05
R^2                   -6.400000e-02
Explained Variance    -0.000000e+00
Max Error              2.621435e+06
F Score (macro)        5.900000e-02
Average RMSE per Case  6.604665e+04

Scores on validation data:
                            Seconds
Accuracy               4.170000e-01
Standard Deviation     9.350373e+04
MSE                    9.176499e+09
RMSE                   9.579404e+04
R^2                   -5.000000e-02
Explained Variance    -0.000000e+00
Max Error              1.310715e+06
F Score (macro)        6.200000e-02
Average RMSE per Case  3.315443e+04

Naïve activity prediction:

Scores on test data:

Accuracy                   0.454
Standard Deviation         0.498
F Score (macro)            0.137
Average Accuracy per Case  0.498

Scores on validation data:

Accuracy                   0.442
Standard Deviation         0.497
F Score (macro)            0.134
Average Accuracy per Case  0.505
 
Random Forest activity prediction:

Scores on test data:

Accuracy                   0.406
Standard Deviation         0.491
F Score (macro)            0.052
Average Accuracy per Case  0.273

Scores on validation data:

Accuracy                   0.313
Standard Deviation         0.464
F Score (macro)            0.043
Average Accuracy per Case  0.172

LTS activity prediction:

Scores on test data:

Accuracy                   0.464
Standard Deviation         0.499
F Score (macro)            0.204
Average Accuracy per Case  0.330

Scores on validation data:

Accuracy                   0.388
Standard Deviation         0.487
F Score (macro)            0.191
Average Accuracy per Case  0.270
</pre>
</details>

Lastly, we output the generated predictions together with the default column labels in the specified file. This file should look similar to:
```csv
eventID ,case concept:name,case REG_DATE,case AMOUNT_REQ,event concept:name,event lifecycle:transition,event time:timestamp,predictedNextEvent,predictedDuration,RForestActivityPred,RForestTimePred,lts_event_prediction,lts_time_prediction
44964012621824,206324,2012-02-03T17:17:11.047+01:00,2500,A_SUBMITTED,COMPLETE,2012-02-03 17:17:11.047,A_PARTLYSUBMITTED,35,W_Completeren aanvraag,0.0,A_SUBMITTED,0
44964012621825,206324,2012-02-03T17:17:11.047+01:00,2500,A_PARTLYSUBMITTED,COMPLETE,2012-02-03 17:17:11.323,W_Afhandelen leads,0,W_Completeren aanvraag,35.0,A_PARTLYSUBMITTED,0
...
```

## Using the Tool
We aimed to make our tool very user friendly, for this we used multiple approaches. Firstly, we implemented a basic user interface, this allows the user to easily interact with the input and settings of the system. Secondly, our tool is frozen into a single file package, this allows us to create a simple .exe that contains the entire tool. The benefit of this is that our software does not require installing python nor any python packages. Lastly, we tried to keep our tool rather flexible, the input of the tool does not have a lot of (odd) requirements.

### Requirements
To run our tool, it is required to provide distinct training and test csv files, which both need to contain the following columns: "eventID ", "event time:timestamp", "event concept:name" and "case concept:name". If any of these exact names is not found, the tool will prompt you to provide the right column. The values of most of these columns do not have any requirements, except for the timestamp, which should be formatted as: %d-%m-%Y %H:%M:%S.%f based on <a href="https://docs.python.org/3/library/datetime.html#strftime-and-strptime-format-codes">this documentation</a>.

### Basic Usage
To use the tool, the user can simply run the .exe, this will open the command-line interface (CLI). This CLI will show the status of the tool and some resulting statistics of the predictions. The CLI is also responsible for requesting the user input, in the case this is not done directly using command-line parameters. Thus, the easiest way to try the tool is to just run the .exe, which should clearly guide users through the usage of the tool. The input of the tool is split in two different parts: inputting filenames, for both input and output; providing parameters (settings) for the execution. For more details on the parameters, see the <a href="#parameters">Parameters</a> section.

### Command Line Usage
Besides the basic CLI that comes with the tool, you can also run it directly from the command-line using a single command. This command is similar to the CLI, however instead of giving inputs one-by-one, you will provide all inputs at once. This syntax is shown below:
<ul>
  <li>Tool_Group8.exe &lt;training.csv> &lt;test.csv> &lt;output.csv> {parameters}</li>
</ul>
The training and test csv files should both meet the requirements given in the Requirements section. Further, any combination of parameters can be used from the parameters section, where they are all separated by a whitespace (" "). This command will run the tool in the current command prompt window and use the directory to look for the specified files. The results of the predictions will be printed in the command prompt window and the output.csv file is saved in the working directory.

### Source Code Usage
Lastly, you can run the source code of the tool, this works very similar to the executable, however it requires you to have the right version of python and some python packages installed. The dependencies are Python version 3.7.9 with following python libraries installed: <br>
<ul>
	<li><a href=https://numpy.org/>Numpy</a> (pip install numpy)</li>
	<li><a href=https://pandas.pydata.org/>Pandas</a> (pip install pandas)</li>
	<li><a href=https://pypi.org/project/subseq/>Subseq</a> (pip install subseq)</li>
	<li><a href=https://scikit-learn.org/stable/>Scikit-learn</a> (pip install scikit-learn)</li>
</ul>
<br>
When you have all the prerequisites, you can simply run the main.py similarly to the previous two use cases. Either run main.py without parameters, or run it from command line using the following syntax:
<ul>
  <li>py main.py &lt;training.csv> &lt;test.csv> &lt;output.csv> {parameters}</li>
</ul>
Running the source code has a slight advantage, since it does not have to deal with unpacking the python code from the .exe and running it after.


## Parameters
By default, our tool will only use one prediction, however other predictors can be enabled. Besides, the tool has some other specific functionality that may or may not be wanted, depending on the data. To allow more user freedom and flexibility, we have implemented multiple parameters. These parameters are the way to control settings and predictors within our tool. Below is a list with all available parameters. They have to be separated using a single space (" "), values between brackets ({}) are optional and variable, the brackets should not be included. For the Predictor Parameters, either of the parameters on both sides of the vertical bar "|" can be used.

#### Predictor Parameters
<ul>
	<li><pre>-s|-subseq    Enable Subseq Predictor (Note: Slow for medium and large datasets)</pre></li>
	<li><pre>-r|-rforest   Enable Random Forest Predictor</pre></li>
	<li><pre>-l|-lts       Enable Transition Matrix Predictor</pre></li>
</ul>

#### Data Separation Parameters
<ul>
	<li><pre>-dist {75}   Apply <a href="#distribution-fixing">distribution fixing</a>, can have value (1-100), default value is 75</pre></li>
	<li><pre>-split       Disable new <a href="#splitting">test/training split creation</a></pre></li>
	<li><pre>-temp        Disable <a href="#temporal-separation">temporal separation</a></pre></li>
</ul>

#### <a href="#on-the-fly">On-The-Fly</a> Parameters
<ul>
	<li><pre>-read        Use only the stored models</pre></li>
	<li><pre>-extend      Combine the stored models with the models obtained from training data</pre></li>
	<li><pre>-write       Store the models upon completion</pre></li>
</ul>


## Predictors
In our main tool we have implemented 4 different predictors, these are the Naïve Predictor, Subseq Predictor, Random Forest Predictor and Transition Matrix Predictor. However, the LSTM Predictor was developed too. We did not implement it in our final tool due to limited time and the risk of causing issues after implementing it. In the following sections, all the predictors are presented shortly, including a link to corresponding documentation/libraries if applicable.

### Naïve Predictor
The naïve prediction algorithms works as follows: Suppose you have a sequence of events in training data and N is the length of the longest subsequence. For every ‘i’ where 0 ≤ i ≤ N, the algorithm decides what the most common activity is at ‘i’ by taking the mode of the ‘i’s in all subsequences. It can also decide what the most occuring time difference is between the activity at ‘i-1’ and at ‘i’ by taking the mode of the time differences for every i. It can then predict, in the test data, the activity at ‘j+1’ by looking at the most common activity for ‘j+1’ and it can predict the timestamp by adding the mode time delta to the timestamp of ‘j’. The predictions by this algorithm are not bad considering its runtime, but lack in terms of intelligence.

### Subseq Predictor
The SuBSeq prediction algorithm works as follows: For both activity and timestamp predictions, a SuBSeq model is fit with all the known sequences in the training data. This will then be stored compactly and efficiently as a Wavelet Tree using Burrows-Wheeler Transform. To make a prediction for the next activity or timestamp for index ‘j+1’, the sequences from index 0 up to and including ‘j’ is given to the algorithm as query. It will then try to find similar sequences in the model with this query and subqueries of this query. If it finds one, it will predict the next element based on the next element found in this sequence. The predictions made by this model are quite good, but the runtime makes the algorithm not very desirable for process mining with large datasets.

### Random Forest Predictor
The random forest prediction algorithms works as follows: It bootstraps the data to create different decision trees, and then classifies the prediction as the model of all decision trees. Some preprocessing was needed since the sklearn algorithm can only take numeric values into account. Therefore we created dummy variables the time, event, previous 5 events, and transition. But this algorithm does not seem to take the traces enough into account, and is actually a bad predictor for process mining.

### Transition Matrix Predictor
The transition matrix prediction algortihm works as follows: An n x n matrix is initialized with all zeros, where n is the amount of unique events / time bins. The matrix is then filled with frequencies of transitions between two values, using the training data. The frequencies of all starting values (of each trace) are also stored. Then for each event in the test data, if that event is the first in a trace, the most occuring first value is predicted and the frequencies are updated with the actual value in this data point. If it is not the first event in a trace, the target event with the largest frequency from the previous event of this data point (according to the transition matrix) is predicted. Afterwards the transition matrix is updated with the correct value. This algortihm only looks at the previous event or previous time bin as of right now, so it performance only marginally better than the naïve algorithm. It runs quickly however, so using more features in the transition matrix might increase its prediction capability.

### LSTM Predictor
This is a more experimental predictor, we do not have this implemented in our main tool. This predictor was the latest we worked on and was the most difficult. Since this was partially finished before the deadline, we decided not to implement this any further, to prevent last minute issues with the rest of the code. Nevertheless, we decided to include this predictor as a separate python script, since it is one of the (experimental) predictors we worked on. All the files for this predictor can be found in the LSTM Predictor folder of the submitted ZIP archive. To demonstrate this predictor, we have made separate training, validation and test data (contained in the 'data' folder), which are smaller preprocessed datasets. These datasets are used automatically in the script, since their paths are hardcoded.
<br><br>
 This predictor is currently unfinished, it is able to predict values for the next event, however these are not applied or shown anywhere. They are however scored using the <a href="https://keras.rstudio.com/reference/evaluate_generator.html">evaluate_generator</a>, thus we can give an idea of how good this prediction would work for our selected dataset. Running this prediction and scoring it will take roughly 10 minutes. After which it will print the score and accuracy returned by the evaluate_generator. This script uses <a href=https://www.tensorflow.org/learn>TensorFlow</a> (pip install tensorflow) and <a href=https://keras.io/>Keras</a> (pip install keras) for the predictions, these are two additional dependencies that should be installed before running the script. With these installed you can just run the python script (from command-line) using:
 <ul>
  <li>py LSTM_activity_model.py</li>
</ul>

LUUK's explanation of LSTM

## Preprocessing
Before we run our predictors over the training and test data, we need to do some processing. This is required for the predictors, error measures, but also to remove possible biases of the dataset. This preprocessing consists of multiple parts, the most important ones are discussed in the following sections.

### Splitting
Firstly, the software tries to detect when a bad training/test split is provided. This is determined using the following formula: <i>train_test_diff < train_diff * 0.10</i>, where train_test_diff is the difference between the first (chronological) timestamp in both training and test data and train_diff is the difference between the first and last timestamp within the training data. The reason for this detection is to prevent a small dataset after applying temporal separation, since it would remove all cases that overlap with the test data. If this formula yields true, an attempt is made to split the data 80/20%, where the trainingdata will exist of the first 80% of the cases and the test data of the last 20%. The manual splitting is enabled by default, but can be disabled using the "-split" parameter.

### Temporal Separation
This process removes all the training cases that overlap with any of the test cases, since this could give an unfair advantage to the prediction model. It works by simply detecting the first (chronological) timestamp in the test data. Next, it drops all the data from the cases that include any event after this specific timestamp. This process is also enabled by default, but can be disabled using the "-temp" parameter.

### Distribution Fixing
Lastly, an attempt can be made to fix the distribution of case lengths within the data. This can be usefull when the training data includes a lot of short cases, whilst the test data includes relatively more long cases. This function will try to (randomly) drop cases from the training data to get a similar distribution of case lengths, compared to the test data. This function will drop 25% of the training data by default, but using the parameter you can modify this value. To visualize the current and generated distribution two images are created, "before.png" and "after.png", these will show the density of the case lengths for both the test and training data. Which you can use to tweak the variable of the distribution parameter. The variable is the percentage of cases to keep from the training data. To enable distribution fixing with the default 75%, use the "-dist" parameter. To enable distribution fixing with a specific percentage, use "-dist 75" where you replace 75 with any value between 1 and 100.

### Time Binning
The current timestamps are very specific, this does not favor most (used) prediction models. Therefore, we used binning for the time predictions, this reduces the amount of distinct values, thus making it easier to match patterns for predictors. The bins are calculated using the following formula: B(i) = B(i-1) + 5 \* 2^(i-1), B(0) = c Where B(i) is the lowest value of bin i (in seconds), and c is the lowest time difference of two consecutive events in the dataset. All further time predictions in the tool are made using these bins. Also, the error measure uses these bins instead of the exact timestamps to measure the score of the predictors.

## On-The-Fly
One of the extra features we implemented is the On-The-Fly technique for the <a href="#naïve-predictor">Naïve Predictor</a>. The On-The-Fly technique that we applied, allows us to reuse a trained model for the predictions. The advantage of this is that you do not have to obtain the prediction model from the training data, since we read it from a file. Another use case is to extend an existing model with another dataset, this way you can combine multiple sets of training data to obtain a single prediction model. The model works by using the 3 given <a href="#on-the-fly-parameters">On-The-Fly parameters</a>.

### On-The-Fly Example
Below is the most basic example use of the On-The-Fly technique. It first runs the tool over the entire datasets in fast mode, storing the naïve models. After executing the first command, you can run the second command, which will load the stored models. This way you do not have to compute the same naïve models every time you run the tool.
<ul>
	<li>Tool_Group_8.exe BPI_Challenge_2-training.csv BPI_Challenge_2-test.csv output.csv -write</li>
	<li>Tool_Group_8.exe BPI_Challenge_2-training.csv BPI_Challenge_2-test.csv output.csv -read</li>
</ul>


## Error Measurements
Our predictions are scored based on multiple error measurements. These error measurements give a basic indication of which predictions are good and which are bad predictors (for the tested dataset). The calculations for these values are mostly done using the <a href=https://scikit-learn.org/stable/>Scikit-learn</a> library, with exception for the standard deviation which was done using <a href=https://numpy.org/>Numpy</a>. The error measure is first computed for the test data and after that for the validation data. This is to detect possible bias within the training and test data, when the values differ a lot.
