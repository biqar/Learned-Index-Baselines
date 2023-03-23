# Learned Indexes

The code of this project is a simple implementation of the article `"The Case For Learned Index Structures"`, which implements the BTree part of the article. Currently, it supports `integer` test sets, and `random distribution` or `exponential distribution` can be selected for testing.

T. Kraska, A. Beutel, E. H. Chi, J. Dean, and N. Polyzotis. The Case for Learned Index Structures. https://arxiv.org/abs/1712.01208, 2017

Additionally, the project explored scenarios with new data insertions.

>Language: Python 3.x (tested on Python 3.8.10)
Support content: Integer values, Random and Exponential distribution

## Required Python Libraries
The code has been tested on:
> Pandas (1.5.3), TensorFlow (2.11.0), NumPy (1.23.5), enum

Please note that, the code used TensorFlow 1.x and our system have TensorFlow installed v2.11.0. So, we import TensorFlow as `import tensorflow.compat.v1 as tf` and disables TensorFlow 2.x behaviors by `tf.disable_v2_behavior()`.
Please let me know if you find TensorFlow related issues while running the code in your system.

## Files Structures
> data/: test data  
model/: learned NN model  
perfromance/：NN and BTree performance  
Learned_BTree.py: main file  
Trained_NN.py: NN structures

## HOW TO RUN
Use the following command to run the Learned_BTree.py,

```> python Learned_BTree.py -t <Type> -d <Distribution> [-p|-n] [Percent]|[Number] [-c] [New data] [-h]```.  
  
>Parameters:  
'type': 'Type: sample, full',  
'distribution': 'Distribution: random, binomial, poisson, exponential, normal, lognormal',  
'percent': 'Percent: 0.1-1.0, default value = 0.5; sample train data size = 300,000',  
'number': 'Number: 10,000-10,000,000, default value = 300,000',  
'new data' 'New Data: INTEGER, 0 for no creating new data file, others for creating'  
  
Example:  
```> python Learned_BTree.py -t full -d random -n 100000 -c 1```  

## Data Index

![Stage Models](https://github.com/biqar/Learned-Index-Baselines/blob/master/about/models.PNG)

1. Based on the ideas in the paper, build a hybrid multi-level neural network architecture
``` 
 Input: int threshold, int stages[]
 Data: record data[]
 Result: trained index
1 M = stages.size;
2 tmp_records[][];
3 tmp_records[1][1] = all_data;
4 for i ← 1 to M do
5   for j ← 1 to stages[i] do
6     index[i][j] = new NN trained on tmp.records[i][j];
7     if i < M then
8       for r ∈ tmp.records[i][j] do
9         𝑞 = f(r.key) / stages[i + 1];
10        tmp.records[i + 1][ 𝑞].add(r);
11 for j ← 1 to index[M].size do
12   index[M][j].calc_err(tmp.records[M][j]);
13   if index[M][j].max_abs_err > threshold then
14     index[M][j] = new B+Tree trained on tmp_records[M][j];
15 return index;
```

> In the above program, starting from the entire dataset (line 3), the level 1 model is first trained. Based on the prediction of the first-level model, the model is selected from the next level, and the corresponding keywords are added to the model training set (lines 9 and 10), and then the model is trained. Finally, the model at the last level is checked, and if the average error is above a predefined threshold, the neural network model is replaced with a B-tree (Lines 11-14). The models used are all fully connected networks, the random distribution uses a fully connected network without hidden; the exponential distribution uses a fully connected network with a hidden layer with 8 cores

2. Use the data to test the neural network index and the B+Tree index, and compare the performance of the two

## Sample Training
The training of the neural network model takes a long time, and the speed of training can be accelerated by extracting a part of data for training.
Sample training is also included in this project, you can use parameter 'sample' for '-t' to test sample training, while '-p' is used for change the sample training percent. Also, the sample training does not include B+Tree as part of the benchmark test. Feel free to update the code if you also want to include B+Tree benchmark as part of the sample training.

Example:  
```> python Learned_BTree.py -t sample -d random -p 0.3 -c 0```

## Storage Optimization
Based on the idea that the distribution of subsequent inserted data is close to the existing distribution.

### The main steps
1. Estimate the data distribution based on the established data index, and move the location of the data to reserve space. For example, the original 0-100 data occupies 100 BLOCKs, and it is expected that the final stored data will be twice the current size, so 100 BLOCKs will be reserved.
2. Inserting data, compared to no optimization.

### Advantage
1. Newly inserted data has fewer conflicts and speeds up insertion.
2. There is no need to readjust the index, which reduces the index maintenance cost and supports new data insertion scenarios.
