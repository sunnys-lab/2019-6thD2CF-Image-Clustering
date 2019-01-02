># 2019 6th D2 CampusFest Image Clustering

- 6th D2 CampusFest Image Clustering 대회 참가작
- tensorflow Hub를 이용한 이미지 유사도 분석 및 유사이미지 clustering
- 아래의 내용은 다음의 reference들을 참고하여 작성:  
 https://www.tensorflow.org/hub/tutorials/image_retraining  
 http://androidkt.com/retrain-image-classifier-model-using-tensorflow-hub/
- **주의사항: 네이버의 대회 참가용 이미지 공유 정책에 따라 ./item_photos 폴더, ./img/dev 폴더는 github에 공유하지 않았습니다.**
 
&nbsp;
>## 01. Installation
- Python 3.6
- Tensorflow version 1.12.0 
- Tendorflow-Hub version 0.2.0 
 
&nbsp;
>## 02. TensorFlow Hub Image Module for Retraining
- Tensorflow Hub는 지난 2018년 4월에 구글이 공개한 것으로 사진이나 텍스트 데이터들을 미리 학습하여 모듈로 제공하고 있습니다. 여기에 별도로 필요한 데이터를 추가하여 빠르게 학습할 수 있다는 장점이 있습니다.
- Tensorflow Hub는 image classification 기능을 제공하는 모듈과, 단순히 image의 feature vector를 추출하는 모듈을 포함하고 있습니다. 
- (관련원문)  The module is actually a saved model.It contains pre-trained weights and graphs.It is reusable,re-trainable.It packs up the algorithm in the form of a graph and weights.
You can find a list of all of the newly released image modules.  
Some of them include the classification layers and some of them remove them just providing a feature vector as output.  
We’ll choose one of the feature vectors modules Inception V1.  
![Modules-Contain](./Modules-Contain.png)  
- 본 프로젝트에서는 기존의 상품 이미지의 특징 벡터만 뽑아내서 학습한 학습모델을 이용하여, 임의의 이미지 유사도를 측정하는 방법입니다.
- 유사도 비교 후 유사도가 가장 높은 기존 이미지의 class를 해당 class 이름 및 index로 라벨링 합니다.
 

&nbsp;
>## 03. Creating Training Set of Images
- 기존 이미지들을 tensorflow-hub 의 retrain.py에 사용하기 위해 class별 subfoler 구성이 필요
- make_item_subfolers.py 를 실행하여 해당 작업을 자동화 

&nbsp; 
>## 04.Retrain Image Module

* Training 이미지 준비 후 최신 TensorFlow 버전(1.7*)과 TensorFlow Hub를 설치
* Training은 다음과 같이 실행

~~~
retrain.py \
--bottleneck_dir=./bottlenecks \
--how_many_training_steps 100 \
--model_dir=./inception \
--output_graph=retrained_graph.pb \
--output_labels=retrained_labels.txt \
--image_dir=./item_photos \
--summaries_dir=./log
~~~
* 명령 설명    
--bottleneck_dir = 학습할 사진을 변환해서 저장할 폴더    
--how_many_training_steps 100  = 스탭 설정   
--output_graph = 추론에 사용할 학습된 바이너리 파일(.pb) 저장 경로  
--output_labels = 추론에 사용할 레이블 파일 저장 경로  
--image_dir = 원본 이미지가 저장된 경로  
--summaries_dir = tensorboard에 사용될 로그 파일 저장 경로  

* 본 프로젝트에서는 다음과 같이 실행
~~~ 
python3 retrain.py --image_dir ./item_photos --how_many_training_steps 100 --output_graph=./output/retrained_graph.pb --output_labels=./output/retrained_labels.txt
~~~

* Training이 끝나면 다음과 같은 형태로 종료
~~~
(skip)
.
.
INFO:tensorflow:2019-01-02 07:33:43.877342: Step 99: Cross entropy = 0.121734
INFO:tensorflow:2019-01-02 07:33:43.995348: Step 99: Validation accuracy = 100.0% (N=100)
2019-01-02 07:33:46.157472: W tensorflow/core/graph/graph_constructor.cc:1265] Importing a graph with a lower producer version 26 into an existing graph with producer version 27. Shape inference will have run different parts of the graph with different producer versions.
INFO:tensorflow:Saver not created because there are no variables in the graph to restore
INFO:tensorflow:Restoring parameters from /tmp/_retrain_checkpoint
INFO:tensorflow:Final test accuracy = 100.0% (N=36)
INFO:tensorflow:Save final result to : ./output/retrained_graph.pb
2019-01-02 07:33:51.302766: W tensorflow/core/graph/graph_constructor.cc:1265] Importing a graph with a lower producer version 26 into an existing graph with producer version 27. Shape inference will have run different parts of the graph with different producer versions.
INFO:tensorflow:Saver not created because there are no variables in the graph to restore
INFO:tensorflow:Restoring parameters from /tmp/_retrain_checkpoint
INFO:tensorflow:Froze 378 variables.
INFO:tensorflow:Converted 378 variables to const ops.
WARNING:tensorflow:From E:/PycharmProjects/2019-6thD2CF-Image-Cluster/retrain.py:908: FastGFile.__init__ (from tensorflow.python.platform.gfile) is deprecated and will be removed in a future version.
Instructions for updating:
Use tf.gfile.GFile.
~~~

As retrain.py proceeds, the training images are automatically separated into batches of training, test, and validation data sets.
In the output, we're hoping for high "Train accuracy" and "Validation accuracy" and low "Cross entropy." See How to retrain Inception's final layer for new categories for a detailed explanation of these terms. Expect training to take around 30 minutes on modern hardware.
Pay attention to the last line of output in your console:

INFO:tensorflow:Final test accuracy = 100.0% (N=32)
This says we've got a model that will, nine times out of 10, correctly guess which one of five possible flower types is shown in a given image. Your accuracy will likely differ because of randomness injected into the training process.

The script loads the pre-trained module and trains a new classifier on top of the fruits photos.You can replace the image_dir argument with any folder containing subfolders of
images. The label for each image is taken from the name of the subfolder it’s in.

The top layer receives as input a 2048-dimensional vector for each image. We train a softmax layer on top of this representation. If the softmax layer contains N labels, this corresponds to learning N + 2048*N model parameters for the biases and weights

&nbsp;
>## 5.TensorBoard  
You can visualize the graph and statistics, such as how the weights or accuracy varied during training.
Run this command during or after retraining.

~~~
tensorboard --host 127.0.0.1 --logdir ./log --port=8008
~~~

After TensorBoard is running, navigate your web browser to localhost:8008 to view the TensorBoard.
![tensorboard_sample](./tensorboard_sample.PNG)

&nbsp;
>## 6.Single Image Testing
* 싱글 이미지에 대한 테스트는 다음과 같은 코드로 수행
~~~
python3 label_image.py --graph=./output/retrained_graph.pb --labels=./output/retrained_labels.txt --input_layer=Placeholder --output_layer=final_result --image=./img/dev/6660613703_10513032770_0.jpg
~~~

* 결과 확인  
각 이미지 class명별 유사도 값 확인 가능
~~~
2019-01-02 07:44:08.635076: I tensorflow/core/platform/cpu_feature_guard.cc:141] Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2
Class Name: 6660613703 (score = 0.88343)
Class Name: 6676864296 (score = 0.11092)
Class Name: 6713538177 (score = 0.00565)
~~~


&nbsp;
>## 7. make_labels_pred.py (수정중)
* 2019-01-02    
label_image.py 코드를 응용하여 make_labels_pred.py prediction 수행
make_labels_pred.py 코드는 dictionary 파일 및 test 이미지 전체에 대한 예측을 위한 코드 수정 중

