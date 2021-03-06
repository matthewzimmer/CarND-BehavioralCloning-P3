# Use Deep Learning to Clone Driving Behavior
##### Project 3 for the Self Driving Car Engineer Nanodegree Program

Final submission for Udacity's Self Driving Car Engineer NanoDegree Behavioral Cloning project.

See [model.ipynb](model.ipynb) for a thorough walk-through of this project including code and visualizations.

#### Project Submission Guidelines

For this project, a reviewer will be testing [model.json](model.json) and [model.h5](model.h5) which was generated on the first test track using [model.py](model.py) (the one to the left in the track selection options).

The following naming conventions were used to make it easy for reviewers to find the right files:

* `model.py` - The script used to create and train the model.
* `drive.py` - The script to drive the car. I submitted a modified version of the original drive.py.
* `model.json` - The model architecture.
* `model.h5` - The model weights.
* `README.md` - Explains the structure of my network and training approach.

The rubric for this project may be found [here](https://review.udacity.com/#!/rubrics/432/view).

#### CLI Commands
##### Train the Network (using driving_log.csv and all images in IMG)

`$ python3 model.py --network zimnet --lr 0.001 --epochs 2 --batch_size 128 --dropout_prob 0.5 --activation elu --colorspace yuv --use_weights False`

##### Start client to send signals to the simulator in Autonomous Mode

`$ python3 drive.py model.json`


#### Network Architecture

##### Overview

I present to you in this project a hand crafted, end-to-end deep learning, convolutional neural network (CNN) which performs well on the first track. My network [generalizes well on the second track](https://www.youtube.com/watch?v=Srzk2NvhMqM) in some Graphics Quality settings such as Fastest and Fast. Additional brightness augmentation is required in order to generalize to higher graphics qualities.

Before I arrived at my final architecture, I implemented and trained against several well-know network architectures such as [CommaAI's](https://github.com/commaai/research/blob/master/train_steering_model.py) and [Nvidia's End to End Learning for Self-Driving Cars](http://images.nvidia.com/content/tegra/automotive/images/2016/solutions/pdf/end-to-end-dl-using-px.pdf). While exploring and researching each component used in both of those networks, I learned a lot about the various activation and optimization functions, dropout, etc.

More specifically, here are some resources and academic papers I thoroughly read and digested to ultimately use as inspiration for my final network architecture.

[ReLU vs PRELU](https://arxiv.org/pdf/1502.01852v1.pdf)  
[Fast and Accurate Deep Network Learning by Exponential Linear Units (ELUs)](https://arxiv.org/abs/1511.07289)  
[Dropout](http://www.cs.toronto.edu/~rsalakhu/papers/srivastava14a.pdf)  
[Dropout as explained by Geoffrey Hinton himself](https://www.youtube.com/watch?v=vAVOY8frLlQ)  
[Nvidia's End to End Learning for Self-Driving Cars](http://images.nvidia.com/content/tegra/automotive/images/2016/solutions/pdf/end-to-end-dl-using-px.pdf)  
[Identification of Driver Steering and Speed Control](http://www2.eng.cam.ac.uk/~djc13/vehicledynamics/downloads/Odhams_PhDthesis_Sep06.pdf)  
[Feature Scaling by Andrew Ng](https://youtu.be/aJmorz9gD4g)  
[Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift](https://arxiv.org/abs/1502.03167)  
[Adam: A Method for Stochastic Optimization](https://arxiv.org/abs/1412.6980v8)  
[How does the dropout method work in deep learning?](https://www.quora.com/How-does-the-dropout-method-work-in-deep-learning)  

##### An early diagram I drew on my whiteboard wall in my home office while starting off with Nvidia and CommaAI's network

![Data Collection, Train, Test Lifecycle](docs/images/data_collect_train_test_lifecycle.jpg)

_NOTE_ (2) no longer saves to train.p. My training features are instances of the `RecordingMeasurement` class which holds reference to each camera's image path. It lazy instantiates the source image arrays inside my custom batch generator when the getter method is initially invoked.

##### RecordingMeasurement class documentation

![RecordingMeasurement](docs/architecture/recording_measurement_class.png)


#### Network Summary

```
____________________________________________________________________________________________________
Layer (type)                     Output Shape          Param #     Connected to                     
====================================================================================================
lambda_1 (Lambda)                (None, 20, 40, 3)     0           lambda_input_1[0][0]             
____________________________________________________________________________________________________
convolution2d_1 (Convolution2D)  (None, 16, 36, 24)    1824        lambda_1[0][0]                   
____________________________________________________________________________________________________
maxpooling2d_1 (MaxPooling2D)    (None, 8, 18, 24)     0           convolution2d_1[0][0]            
____________________________________________________________________________________________________
convolution2d_2 (Convolution2D)  (None, 4, 14, 36)     21636       maxpooling2d_1[0][0]             
____________________________________________________________________________________________________
maxpooling2d_2 (MaxPooling2D)    (None, 2, 7, 36)      0           convolution2d_2[0][0]            
____________________________________________________________________________________________________
convolution2d_3 (Convolution2D)  (None, 2, 7, 48)      43248       maxpooling2d_2[0][0]             
____________________________________________________________________________________________________
maxpooling2d_3 (MaxPooling2D)    (None, 1, 3, 48)      0           convolution2d_3[0][0]            
____________________________________________________________________________________________________
convolution2d_4 (Convolution2D)  (None, 1, 3, 64)      27712       maxpooling2d_3[0][0]             
____________________________________________________________________________________________________
dropout_1 (Dropout)              (None, 1, 3, 64)      0           convolution2d_4[0][0]            
____________________________________________________________________________________________________
flatten_1 (Flatten)              (None, 192)           0           dropout_1[0][0]                  
____________________________________________________________________________________________________
dense_1 (Dense)                  (None, 1024)          197632      flatten_1[0][0]                  
____________________________________________________________________________________________________
dropout_2 (Dropout)              (None, 1024)          0           dense_1[0][0]                    
____________________________________________________________________________________________________
dense_2 (Dense)                  (None, 100)           102500      dropout_2[0][0]                  
____________________________________________________________________________________________________
dropout_3 (Dropout)              (None, 100)           0           dense_2[0][0]                    
____________________________________________________________________________________________________
dense_3 (Dense)                  (None, 50)            5050        dropout_3[0][0]                  
____________________________________________________________________________________________________
dropout_4 (Dropout)              (None, 50)            0           dense_3[0][0]                    
____________________________________________________________________________________________________
dense_4 (Dense)                  (None, 10)            510         dropout_4[0][0]                  
____________________________________________________________________________________________________
dense_5 (Dense)                  (None, 1)             11          dense_4[0][0]                    
====================================================================================================
Total params: 400123
____________________________________________________________________________________________________
```

#### Network Breakdown

The network architecture I ultimately chose is a simple 4-layer convolutional neural network with 4 fully connected layers along with 50% dropout after each fully connected layer. This led to a grand total of 400,123 parameters.

I leveraged [Keras' Lamda layer](https://keras.io/layers/core/#lambda) as my input layer to normalize all input features on the GPU since it could do it much faster than on a CPU.

I chose the [Adam optimizer](https://keras.io/optimizers/#adam) after first analyzing and comparing the [various Keras optimizers](https://keras.io/optimizers/) and reading thier corresponding academic papers. It is extremely robust allowing me to focus on other parts of the network.

After trying out different learning rates, I found 0.001 to be the most effective starting point for my network architecture and training data.

My entire network is essentially inspired by the original [Adam: A Method for Stochastic Optimization](https://arxiv.org/abs/1412.6980v8) achademic paper, particlarly _6.3 EXPERIMENT: CONVOLUTIONAL NEURAL NETWORKS_.

> **6.3 EXPERIMENT: CONVOLUTIONAL NEURAL NETWORKS**

> Convolutional neural networks (CNNs) with several layers of convolution, pooling and non-linear
units have shown considerable success in computer vision tasks. Unlike most fully connected neural
nets, weight sharing in CNNs results in vastly different gradients in different layers. A smaller
learning rate for the convolution layers is often used in practice when applying SGD. We show the
effectiveness of Adam in deep CNNs. Our CNN architecture has three alternating stages of 5x5
convolution filters and 3x3 max pooling with stride of 2 that are followed by a fully connected layer
of 1000 rectified linear hidden units (ReLU’s). The input image are pre-processed by whitening, and dropout noise is applied to the input layer and fully connected layer. The minibatch size is also set
to 128 similar to previous experiments.

> Interestingly, although both Adam and Adagrad make rapid progress lowering the cost in the initial
stage of the training, shown in Figure 3 (left), Adam and SGD eventually converge considerably
faster than Adagrad for CNNs shown in Figure 3 (right). We notice the second moment estimate vbt
vanishes to zeros after a few epochs and is dominated by the  in algorithm 1. The second moment
estimate is therefore a poor approximation to the geometry of the cost function in CNNs comparing
to fully connected network from Section 6.2. Whereas, reducing the minibatch variance through
the first moment is more important in CNNs and contributes to the speed-up. As a result, Adagrad
converges much slower than others in this particular experiment. Though Adam shows marginal
improvement over SGD with momentum, it adapts learning rate scale for different layers instead of
hand picking manually as in SGD.


#### Training the Network

##### Gathering Training Data

All training data was collected in the Self-Driving Car simulator on Mac OS using a Playstation 3 controller. 

I recorded myself driving around the track in the center lane approximately 4 times. At each point in autonomous mode where the car vered off in a non-safe manner, I would record recovery data by hugging the left or right shoulder then recording myself gracefully steer into the middle of the lane. This took a lot of time but the effort paid off in the end.

Once I felt I collected enough training samples (~21k), I committed driving_log.csv and all images in the IMG directory to this GitHub repository.

##### Training with Initial Training Data

I trained the network on a g2.2xlarge EC2 instance, saved the model and weights persisted as model.json and model.h5 respectively, `scp`ed model.json and model.h5 to my machine, then tested the model in autonomous mode using `drive.py`.

In addition, I revived an old PC with an Nvidia GeForce 670 with 2 GB of GPU RAM and installed Ubuntu Studio 16.04 on it. This GPU has a 3.0 CUDA GPU compute rating. It trained a single epoch in 60-70s which worked great with my memory optimized batch generator.
