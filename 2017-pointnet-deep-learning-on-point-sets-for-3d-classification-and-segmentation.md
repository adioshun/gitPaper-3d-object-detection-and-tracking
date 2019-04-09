| 논문명 | PointNet: Deep Learning on Point Sets for 3D Classification and Segmentation |
| --- | --- |
| 저자\(소속\) | Charles R. Qi \(Stanford University\)|
| 학회/년도 | Dec 2016 ~ Apr 2017, CVPR 2017,  [논문](https://arxiv.org/abs/1612.00593) |
| 키워드 |Charles2016, Classification, Segmentation    |
| 데이터셋(센서)/모델| ModelNet40,  ShapeNet part data, Stanford 3D semantic parsing data|
|관련연구|이후연구 : Pointnet++|
| 참고 | [ppt](https://www.facebook.com/thinking.factory/posts/1408857052524274), [홈페이지](http://stanford.edu/~rqi/pointnet/),[CVPR2017](https://www.youtube.com/watch?v=Cge-hot0Oc0), [Youtube](https://youtu.be/8CenT_4HWyY?t=1h16m39s), [ppt](http://3ddl.stanford.edu/CVPR17_Tutorial_PointCloud.pdf),  |
| 코드 | [TF](https://github.com/charlesq34/pointnet), [pyTorch](https://github.com/fxia22/pointnet.pytorch) |

> 같은 이름의 다른 논문 :  PointNet: A 3D Convolutional Neural Network for real-time object class recognition, A. Garcia-Garcia, [링크](http://ieeexplore.ieee.org/document/7727386/authors)


![image](https://user-images.githubusercontent.com/17797922/49631762-16f86680-f9a8-11e8-94be-fdf2fbba6d7c.png)




PointNet : End-to-end learning for scattered, unordered point data, Unified framework for various tasks

> Point Cloud : 가장 직관적이고 정보를 잘 표현할 수 있으면서도 다른 방법에서 변환하기도 쉽고 역으로 돌아가기도 쉬우며 얻기도 편하므로 이를 많이 사용한다.

Two Challenges
- Unordered point set as input : data의 order에 invariant해야한다는 점 
- Invariance under geometric transformations : point clouds에 geometric transformation을 가한다고 물체가 다른 것으로 분류되어서도 안된다는 점 

본 논문은
- 첫번쨰(permutation invariance)는 symmetric function을 도입하여 해결
- 두번째는 transformer network를 하나 모듈로 붙여서 해결하였다.

PointNet은 max pooling을 기준으로 앞부분의 local feature단과 뒷부분의 global feature단을 보는 것으로 나눌 수 있는데, 논문에서는 critical point로 불리는 global feature에 영향을 주는 point set은 매우 적고 주요 경계마다만 있고 대다수의 point들은 영향을 주지 않기 떄문에 전에 point clouds에서 50%까지 data loss가 있더라도 전혀 성능에 문제가 발생하지 않는다. \(robustness to missing data\)


```
- Qi et al. [53] propose a Multilayer Perceptron(MLP) architecture 
	-  that extracts a global feature vector from a 3D point cloud of $$1m^3$$ physical size 
	-  and processes each point using the extracted feature vector and additional **point level** transformations. 

- Their method operates at the point level and thus inherently provides a fine-grained segmentation. 

- It works well for indoor semantic scene understanding,although there is no evidence that it scales to larger input dimensions without additional training or adaptation required. 
```


# PointNet


## 1. Introduction

기존 연구 방법 Due to Point cloud's irregular format, most researchers transform such data to

* regular 3D voxel grids or 
* collections of images

기존 연구 문제점 :  This data representation transformation, however, renders the resulting data unnecessarily voluminous — while also introducing quantization artifacts that can obscure natural invariances of the data


제안 방식 
- 입력 
    - s three coordinates (x, y, z)
    - normals 
    - local or global features
- 출력 
    - classifiaction : label
    - Segmentation : 각 point별 label 
    

네트워크 특징 

- a single symmetric function, max pooling. 

    - Effectively the network learns a set of optimization functions/criteria that select interesting or informative points of the point cloud and encode the reason for their selection. 

- The final fully connected layers of the network aggregate these learnt optimal values into the global descriptor for the entire shape as mentioned above (shape classification) or are used to predict per point labels (shape segmentation).




## 2. Related Work

### 2.1 Point Cloud Features

> Point Cloud 특징들은 `수작업`으로 만든것들이다. 이전 CV 방식 처럼

Most existing features for point cloud are `handcrafted` towards specific tasks.

Point features often encode certain statistical properties of points and are designed to be invariant to certain transformations, 


분류 1
- intrinsic \[2, 24, 3\] 
- extrinsic \[20, 19, 14, 10, 5\].

분류 2
- local features
- global features

For a specific task, it is not trivial to find the optimal feature combination.

### 2.2 Deep Learning on 3D Data

3D data has multiple popular representations, leading to various approaches for learning.

#### A. Volumetric CNNs

- \[28, 17, 18\] are the pioneers applying 3D convolutional neural networks on voxelized shapes.

> ShpaeNet, VoxNet, Vol/Multi-View CNNs

제약 : sparsity problem, 계산 부하 `However, volumetric representation is constrained by its resolution due to data sparsity and computation cost of 3D convolution.`

sparsity 문제 해결 논문 
- FPNN [13]
- Vote3D [26]
- however, their operations are still on sparse volumes, it’s challenging for them to process very large point clouds.

```
[28] Z. Wu, S. Song, A. Khosla, F. Yu, L. Zhang, X. Tang, and J. Xiao. 3d shapenets: A deep representation for volumetric shapes. In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, pages 1912–1920, 2015.
[17] D. Maturana and S. Scherer. Voxnet: A 3d convolutional neural network for real-time object recognition. In IEEE/RSJ International Conference on Intelligent Robots and Systems, September 2015
[18] C. R. Qi, H. Su, M. Nießner, A. Dai, M. Yan, and L. Guibas. Volumetric and multi-view cnns for object classification on 3d data. In Proc. Computer Vision and Pattern Recognition (CVPR), IEEE, 2016. 
[13] Y. Li, S. Pirk, H. Su, C. R. Qi, and L. J. Guibas. Fpnn: Field probing neural networks for 3d data. arXiv preprint arXiv:1605.06240, 2016.
[26] D. Z. Wang and I. Posner. Voting for voting in online point cloud object detection. Proceedings of the Robotics: Science and Systems, Rome, Italy, 1317, 2015
```

#### B. Multiview CNNs

> 3D Point Cloud를 2D 이미지로 맵핑하고 2D CNN을 접목하는 방법, 성능이 잘 나옴

- \[23, 18\] have tried to render 3D point cloud or shapes into 2D images and then apply 2D conv nets to classify them.

- With well engineered image CNNs, this line of methods have achieved dominating performance on shape classification and retrieval tasks \[21\].

- However, it’s nontrivial to extend them to scene understanding or other 3D tasks such as point classification and shape completion.

```
[23] H. Su, S. Maji, E. Kalogerakis, and E. G. Learned-Miller. Multi-view convolutional neural networks for 3d shape recognition. In Proc. ICCV, to appear, 2015
[18] C. R. Qi, H. Su, M. Nießner, A. Dai, M. Yan, and L. Guibas. Volumetric and multi-view cnns for object classification on 3d data. In Proc. Computer Vision and Pattern Recognition (CVPR), IEEE, 2016
[21] M. Savva, F. Yu, H. Su, M. Aono, B. Chen, D. Cohen-Or, W. Deng, H. Su, S. Bai, X. Bai, et al. Shrec16 track largescale 3d shape retrieval from shapenet core55.
```

#### D. Spectral CNNs

- Some latest works \[4, 16\] use spectral CNNs on meshes.

- However, these methods are currently constrained on manifold meshes such as organic objects and it’s not obvious how to extend them to non-isometric shapes such as furniture.

```
[4] J. Bruna, W. Zaremba, A. Szlam, and Y. LeCun. Spectral networks and locally connected networks on graphs. arXiv preprint arXiv:1312.6203, 2013
[16] J. Masci, D. Boscaini, M. Bronstein, and P. Vandergheynst. Geodesic convolutional neural networks on riemannian manifolds. In Proceedings of the IEEE International Conference on Computer Vision Workshops, pages 37–45, 2015.
```

#### E. Feature-based DNNs

- \[6, 8\]firstly convert the 3D data into a vector, by extracting traditional shape features and then use a fully connected net to classify the shape.

- 제약 : We think they are constrained by the representation power of the features extracted.

> 특징 추출단계에서 표현력 제약 발생 가능

```
[6] Y. Fang, J. Xie, G. Dai, M. Wang, F. Zhu, T. Xu, and E. Wong. 3d deep shape descriptor. In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition, pages 2319–2328, 2015. 
[8] K. Guo, D. Zou, and X. Chen. 3d mesh labeling via deep convolutional neural networks. ACM Transactions on Graphics (TOG), 35(1):3, 2015.
```

### 2.3 Deep Learning on Unordered Sets

* 구조적 관점에서 보면 포인트 클라우드는 **Unordered**이다. `From a data structure point of view, a point cloud is an unordered set of vectors.`

* 대화, 언어, 비디오, 3D\(Volumes\)들에 대하여서는 연구 되었지만, 포인트 클라우드에 대해서는 연구 되지 않았다. `While most works in deep learning focus on regular input representations like sequences (in speech and language processing), images and volumes (video or 3D data), not much work has been done in deep learning on point sets.`

> 저자는 3D를 `volumes 과 unordered로 나누어서 보고있음 `

* 그나마 최근 연구는 \[25\]이다. `One recent work from Oriol Vinyals et al [25] looks into this problem.`

    * They use a read-process-write network with attention mechanism to consume unordered input sets and show that their network has the ability to sort numbers.

    - 그러나 이 연구 역시 NLP에 초점을 두고 있어서 Geo정보처리는 안한다. `However, since their work focuses on generic sets and NLP applications, there lacks the role of geometry in the sets.`

```
[25] O. Vinyals, S. Bengio, and M. Kudlur. Order matters: Sequence to sequence for sets. arXiv preprint arXiv:1511.06391, 2015.
```


## 3. Problem Statement


We design a deep learning framework that directly consumes unordered point sets as inputs. 

A point cloud is represented as a set of 3D points $$\{P_i \mid i = 1, ..., n \} 
- each point $$P_i$$ is a vector of its $$(x, y, z)$$ coordinate plus extra feature channels such as color, normal etc. 

For simplicity and clarity, unless otherwise noted, we only use the (x, y, z) coordinate as our point’s channels.

### 3.1 classification

For the object classification task, the **input **point cloud is 
    - either directly sampled from a shape 
    - or pre-segmented from a scene point cloud. 

Our proposed deep network outputs k scores for all the k candidate classes. 

### 3.2 semantic segmentation

For semantic segmentation, the **input** can be 
    - a single object for part region segmentation, 
    - or a sub-volume from a 3D scene for object region segmentation. 

Our model will output $$n × m$$ scores for each of the $$n$$ points and each of the $$m$$ semantic subcategories.


## 4. Deep Learning on Point Sets


The architecture of our network (Sec 4.2) is inspired by the properties of point sets in $$\Re^n$$ (Sec 4.1).



### 4.1 Properties of Point Sets in $$\Re^n$$ 

Our input is a subset of points from an **Euclidean space**.

3가지 주요 특징 `It has three main properties:`

#### A. Unordered

- Unlike pixel arrays in images or voxel arrays in volumetric grids, point cloud is a set of points without specific order. 

- In other words, a network that consumes $$N$$ 3D point sets needs to be invariant to $$N!$$ permutations of the input set in data feeding order.

#### B. Interaction among points. 

- The points are from a space with a distance metric. 

- It means that points are not isolated, and neighboring points form a meaningful subset. 

- Therefore, the model needs to be able to capture local structures from nearby points, and the combinatorial interactions among local structures.


#### C. Invariance under transformations

- As a geometric object, the learned representation of the point set should be invariant to certain transformations. 

- For example, rotating and translating points all together should 
    - not modify the global point cloud category 
    - nor the segmentation of the points.

### 4.2 PointNet Architecture

![](https://i.imgur.com/LZiDf16.png)
```
[Figure 2. PointNet Architecture.] 
- The classification network takes n points as input, applies input and feature transformations, and then aggregates point features by max pooling. 
    - The output is classification scores for k classes. 
- The segmentation network is an extension to the classification net. 
    - It concatenates global and local features and outputs per point scores. 
    - “mlp” stands for multi-layer perceptron, numbers in bracket are layer sizes. 
    - Batchnorm is used for all layers with ReLU. 
    - Dropout layers are used for the last mlp in classification net.
```

Our network has three key modules: 
1. The max pooling layer as a **symmetric function** to aggregate information from all the points, 
2. a local and global information combination structure, 
3. and two joint alignment networks that align both input points and point features.


#### A. Symmetry Function for Unordered Input

입력 순서에 영향받지 않는 모델 만드는 3가지 방법 `In order to make a model invariant to input permutation, three strategies exist:` 
1. sort input into a canonical order
2. treat the input as a sequence to train an RNN, but augment the training data by all kinds of permutations;
3. use a simple symmetric function to aggregate the information from each point. 

##### 가. symmetric function (3번)

Here, a symmetric function takes n vectors as input and outputs a new vector that is invariant to the input order. 

- For example, `+` and `∗` operators are **symmetric binary functions**.

##### 나. sorting (1번)

- 정렬이 간단한 방법 같지만 고차원에서는 정렬방법은 없다. `While sorting sounds like a simple solution, in high dimensional space there in fact does not exist an ordering that is stable w.r.t. point perturbations in the general sense. This can be easily shown by contradiction. `

- 비록 있다고 하더라도 고차원 공간과 1d real line간의 bijection map을 정의한것이다. `If such an ordering strategy exists, it defines a bijection map between a high-dimensional space and a 1d real line. `

It is not hard to see, to require an ordering to be stable w.r.t point perturbations is equivalent to requiring that this map preserves spatial proximity as the dimension reduces, a task that cannot be achieved in the general case. 

Therefore,sorting does not fully resolve the ordering issue, and it’s hard for a network to learn a consistent mapping from input to output as the ordering issue persists. 

As shown in experiments (Fig 5), we find that applying a MLP directly on the sorted point set performs poorly, though slightly better than directly processing an unsorted input.

##### 다. RNN (2번)

The idea to use RNN considers the point set as a sequential signal and hopes that by training the RNN with randomly permuted sequences, the RNN will become invariant to input order. 

However in “Order Matters” [25]the authors have shown that order does matter and cannot be totally omitted. 

While RNN has relatively good robustness to input ordering for sequences with small length (dozens), it’s hard to scale to thousands of input elements, which is the common size for point sets. 

Empirically, we have also shown that model based on RNN does not perform as well as our proposed method (Fig 5).


##### 라. 제안 방식 

Our idea is to approximate a general function defined on a point set by applying a symmetric function on transformed elements in the set:

![](https://i.imgur.com/RxCawYT.png)

- we approximate $$h$$ by a multi-layer perceptron network and $$g$$ by a composition of a single variable function and a max pooling function. 

- This is found to work well by experiments. 

- Through a collection of $$h$$, we can learn a number of $$f’s$$ to capture different properties of the set. 




![](https://i.imgur.com/aFUsYsC.png)
```
[Figure 5. Three approaches to achieve order invariance.] 
- Multilayer perceptron (MLP) applied on points consists of 5 hidden layers with neuron sizes 64,64,64,128,1024, all points share a single copy of MLP. 
- The MLP close to the output consists of two layers with sizes 512,256.
```

#### B. Local and Global Information Aggregation

#### C. Joint Alignment Network

### 4.3. Theoretical Analysis

## 5. Experiment





---

# PointNet

- 스탠포드대 : http://stanford.edu/~rqi/pointnet/
- keras : https://github.com/garyli1019/pointnet-keras
- pytorch : https://github.com/fxia22/pointnet.pytorch
- Tensorflow : https://github.com/charlesq34/pointnet
- open3d활용 pytorch : https://github.com/IntelVCL/Open3D-PointNet
- Semantic3D (semantic-8) segmentation with Open3D and PointNet++ : https://github.com/IntelVCL/Open3D-PointNet2-Semantic3D
- pointnet-autoencoder : https://github.com/charlesq34/pointnet-autoencoder


This dataset provides part segmentation to a subset of ShapeNetCore models, containing ~16K models from 16 shape categories. The number of parts for each category varies from 2 to 6 and there are a total number of 50 parts.
The dataset is based on the following work:
```
@article{yi2016scalable,
  title={A scalable active framework for region annotation in 3d shape collections},
  author={Yi, Li and Kim, Vladimir G and Ceylan, Duygu and Shen, I and Yan, Mengyan and Su, Hao and Lu, ARCewu and Huang, Qixing and Sheffer, Alla and Guibas, Leonidas and others},
  journal={ACM Transactions on Graphics (TOG)},
  volume={35},
  number={6},
  pages={210},
  year={2016},
  publisher={ACM}
}
```
You could find the initial mesh files from the released version of ShapeNetCore.
An mapping from synsetoffset to category name could be found in "synsetoffset2category.txt"
```
The folder structure is as below:
	-synsetoffset
		-points : *.pts , x,y,z(??)
			-uniformly sampled points from ShapeNetCore models
		-point_labels : *.seg (1~3)
			-per-point segmentation labels
		-seg_img : *.png
			-a visualization of labeling
	-train_test_split
		-lists of training/test/validation shapes shuffled across all categories (from the official train/test split of ShapeNet)

    Airplane	02691156
    Bag	02773838
    Cap	02954340
    Car	02958343
    Chair	03001627
    Earphone	03261776
    Guitar	03467517
    Knife	03624134
    Lamp	03636649
    Laptop	03642806
    Motorbike	03790512
    Mug	03797390
    Pistol	03948459
    Rocket	04099429
    Skateboard	04225987
    Table	04379243
```


####   pts handling : [pts_loader](https://github.com/albanie/pts_loader)

```python
def load(path):
    """takes as input the path to a .pts and returns a list of 
	tuples of floats containing the points in in the form:
	[(x_0, y_0, z_0),
	 (x_1, y_1, z_1),
	 ...
	 (x_n, y_n, z_n)]"""
    with open(path) as f:
        rows = [rows.strip() for rows in f]
    
    """Use the curly braces to find the start and end of the point data""" 
    head = rows.index('{') + 1
    tail = rows.index('}')

    """Select the point data split into coordinates"""
    raw_points = rows[head:tail]
    coords_set = [point.split() for point in raw_points]

    """Convert entries from lists of strings to tuples of floats"""
    points = [tuple([float(point) for point in coords]) for coords in coords_set]
    return points
```

#### pts Visualization : [A library for visualization and creative-coding ](https://github.com/williamngan/pts)

#### open3d지원

```python
pcd = read_point_cloud('./sample.pts')
```
