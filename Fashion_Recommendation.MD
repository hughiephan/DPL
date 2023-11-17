# Fashion Recommendation with Xception and Nearest Neighbors

![image](https://github.com/hughiephan/DPL/assets/16631121/41645366-fed7-4162-aea8-a92ea935c4ac)

We will use Xception Feature Exaction layer, plugging in with the `sklearn.neighbors` method to find similiar images. 

## Step 1: Import libraries
```python
import os
import cv2
import numpy as np
import matplotlib.pyplot as plt
from keras.applications.xception import Xception,preprocess_input
from sklearn.neighbors import NearestNeighbors
```

## Step 2: Prepare dataset
Download the Fasion Recommendation dataset from: https://www.kaggle.com/competitions/h-and-m-personalized-fashion-recommendations/data . A dirname contains a list of filename (many images). We will join the dirnames and filenames together to create our dataset. Even though there are total of 105100 images, for our tutorial we focus on the first 10000 images.

```python
dataset = []
for dirname, _, filenameList in os.walk('/kaggle/input/h-and-m-personalized-fashion-recommendations/images'):
    for filename in filenameList:
        dataset.append(os.path.join(dirname, filename))
dataset = dataset[:10000]
```

![image](https://github.com/hughiephan/DPL/assets/16631121/6fdf8065-63a8-441d-aa48-658bfd8c5f0f)

## Step 3: Add Xception Model

![image](https://github.com/hughiephan/DPL/assets/16631121/625bda7a-5b44-47ad-b801-1f60f6251830)

Xception stands for “extreme inception”, it takes the principles of Inception to an extreme. XCeption architecture relies on Depthwise Separable Convolution, and Shortcuts between Convolution blocks as in ResNet. You can read more about Xception here: https://maelfabien.github.io/deeplearning/xception . For this tutorial, we will use Xception model which were already trained on 14.000.000 images of Imagenet dataset. We use `include_top=False` to remove the Fully-connected Layer as this Layer is not useful for our task at-hand. We set all the layers to False in Xception layer with `layer.trainable=False` to stop it from updating the weights.

```python
model = Xception(weights='imagenet', include_top=False)
for layer in model.layers:
    layer.trainable=False
```

## Step 4: Pre-process and extract features

Image files vary in size so we need to resize them all into the same width and height using `cv2.resize`. But Xception input must be 4-dimensional so we need to expand our image dimension with `expand_dims`. Last pre-process step is using `xception.preprocess_input` to normalize the input pixels between -1 and 1. 

![image](https://github.com/hughiephan/DPL/assets/16631121/05c87d19-8a50-4f0f-b4ae-ee2a95b40725)

```python
features=[]
for img_path in dataset:
    # Preprocess Image
    image_data = cv2.imread(img_path)
    image_data = cv2.resize(image_data, (225,225), interpolation=cv2.INTER_AREA)  
    image_data = np.expand_dims(image_data, axis=0)
    image_data = preprocess_input(image_data)
    # Feature Extraction
    extracted_feature = model.predict(image_data)
    extracted_feature = np.array(extracted_feature).flatten()
    features.append(extracted_feature)
feature_vec = np.array(features)
```

## Step 5: Training with KD Tree Nearest Neighbors
```python
nNeighbors = 4
nbrs = NearestNeighbors(n_neighbors = nNeighbors, algorithm = 'kd_tree').fit(feature_vec)
```

## Step 6: Predict

![image](https://github.com/hughiephan/DPL/assets/16631121/9e275165-fe81-4bd7-957f-a1c112b8cd4a)

`distances` is the distances to the 4 closet neighbors. And `result` is the index of that 4 neighbors.

```python
testImagePath = dataset[999]
testImage = cv2.imread(testImagePath) 
testImage = cv2.resize(testImage, (225,225), interpolation=cv2.INTER_AREA)  
testImage = np.expand_dims(testImage, axis=0)
testImage = preprocess_input(testImage)
testFeature = model.predict(testImage)
testFeature = np.array(testFeature)
testFeature = testFeature.flatten()
distances, result = nbrs.kneighbors([testFeature])
```

## Step 7: Visualize predictions
```python
plt.title("Query Image")
plt.imshow(cv2.imread(testImagePath))
fig = plt.figure(figsize=(12,8))
fig.suptitle('Similar Images')
for i in range(0, nNeighbors):
    index_result=result[0][i]
    plt.subplot(3, 4, i+1)
    plt.imshow(cv2.imread(dataset[index_result]))
plt.show()
```

![image](https://github.com/hughiephan/DPL/assets/16631121/112c37dd-f849-4711-a435-59e0b77ee537)

## Todo: 

Publish on group

## Reference
- https://www.kaggle.com/code/hamditarek/similar-image-cnn-cosine-similarity
- https://maelfabien.github.io/deeplearning/xception/#what-does-it-look-like
- https://scikit-learn.org/stable/modules/metrics.html#cosine-similarity
- https://www.geeksforgeeks.org/search-and-insertion-in-k-dimensional-tree
- https://www.geeksforgeeks.org/introductory-guide-to-information-retrieval-using-knn-and-kdtree/