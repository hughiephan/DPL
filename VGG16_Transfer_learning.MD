# Transfer Learning with VGG16 + our custom Fully-connected Layer
Transfer learning is a machine learning research subject that is concerned with the storage of knowledge obtained while resolving one problem and its subsequent application to another related problem. Pre-trained models are frequently utilized as the foundation for deep learning tasks in computer vision and natural language processing because they save both cost and time compared to developing neural network models from scratch and because they perform significantly better on related tasks.

![image](https://github.com/hughiephan/DPL/assets/16631121/8be69cb2-072a-40a5-8141-6b6b776cdde0)

# Objective

Identifying if a person is wearing a facemask or not.

## Step 1: Create a new Notebook from Dataset
Dataset: https://www.kaggle.com/datasets/ashishjangra27/face-mask-12k-images-dataset

![image](https://github.com/hughiephan/DPL/assets/16631121/40eeb8d3-3b12-497c-8ba3-cb2b4e1ffc8d)

## Step 2: Import Libraries
```python
import numpy as np
from tensorflow.keras.preprocessing import image
from tensorflow.keras.preprocessing.image import ImageDataGenerator
from tensorflow.keras.layers import Input, Lambda, Dense, Flatten, Dropout
from tensorflow.keras.models import Model, Sequential
from tensorflow.keras.applications.vgg16 import VGG16
from tensorflow.keras.applications.vgg16 import preprocess_input
```

## Step 3: Config variables
```python
input_size = (128,128)
input_shape = (128, 128, 3)
base_dir = '../input/face-mask-12k-images-dataset/Face Mask Dataset/'
```
![image](https://github.com/hughiephan/DPL/assets/16631121/71604595-7981-4808-890e-e5d57a224c81)

![image](https://github.com/hughiephan/DPL/assets/16631121/15f8cff9-cd23-4d5e-ad79-3e63e1e437eb)

## Step 4: Import Pre-processing & Augmentation Generator

Data augmentation is a process of artificially increasing the amount of data by generating new data points from existing data. This includes adding minor alterations to data or using machine learning models to generate new data points in the latent space of original data to amplify the dataset. Use the ImageDataGenerator function from Keras to create an image generator that can apply these augmentations to the training images. The parameters for the different augmentations are set as follows:
- `Rotation`: Up to 20 degrees
- `Height Shift`: Up to 10% of the image height
- `Width Shift`: Up to 10% of the image width
- `Fill Mode`: Nearest, replaces the empty area with the nearest pixel values when shifting
- `Zoom`: Up to 10%
- `Horizontal Flip`: Enabled

For Validation and Test Set, we only apply normalization by dividing with 255 and get the pixel range from [0 ~ 1] 

```python
train_gen = ImageDataGenerator(
    rescale=1./255, 
    zoom_range=0.1,
    rotation_range=20,
    width_shift_range=0.1,
    height_shift_range=0.1,
    fill_mode="nearest",
    horizontal_flip=True,
    brightness_range=[0.2,1.5],
)
val_gen = ImageDataGenerator(rescale=1./255)
test_gen = ImageDataGenerator(rescale=1./255)
```

## Step 5: Apply Pre-processing & Augmentation 

We can use `flow_from_directory` when the directory structure is organized in a way that each subdirectory represents a different class.

```python
train_data = train_gen.flow_from_directory(base_dir + 'Train',target_size=input_size,seed=42)
val_data = val_gen.flow_from_directory(base_dir + 'Validation',target_size=input_size,seed=42)
test_data = test_gen.flow_from_directory(base_dir + 'Test',target_size=input_size,seed=42)
```
![image](https://github.com/hughiephan/DPL/assets/16631121/3dc60840-7919-49fa-a626-45678cc4d1e9)


## Step 6: Import pre-trained model
Use a Convolutional Neural Network model (VGG16) pre-trained on the ImageNet dataset. The “16” stand for the number of weight layers in the network. In 2014, 16 networks were considered very deep. The network uses only 3×3 convolutional layers stacked on top of each other in increasing depth. Reducing volume size is handled by max pooling. Two fully-connected layers, each with 4,096 nodes are then followed by a softmax classifier

![image](https://github.com/hughiephan/DPL/assets/16631121/5f852ca0-a3fd-484c-94c1-2026617adedd)

Loop through all the VGG16 layers and freeze those layer's weights using `layer.trainable = False`. With `include_top=False`, the model can be used for feature extraction, then we can stack any other model on top of it.

```python
vgg16 = VGG16(input_shape=input_shape, weights='imagenet', include_top=False)
for layer in vgg16.layers:
    layer.trainable = False
vgg16.summary()
```

The total parameters are 14,714,688 but because we have already frozen them so they will not be updated in the training process

![image](https://github.com/hughiephan/DPL/assets/16631121/3d0043ac-22ac-497e-ae3f-3813378c79c5)

## Step 7: New classifier
Add our fully-connected layer after VGG16. These hyperparameters are chosen randomly, and by experimenting we should know what are the best hyperparameters for this classifier layer. Number `2` in the last layer is equal to the `with mask` and `without mask` label. Binary cross-entropy is used for the binary classification task (two target classes)

```python
output = Flatten()(vgg16.output)
output = Dense(500, activation='relu')(output)
output = Dense(100, activation='relu')(output)
output = Dropout(0.5)(output)
output = Dense(2, activation='softmax')(output)
model = Model(inputs=vgg16.input, outputs=output)
model.compile(
  loss='binary_crossentropy',
  optimizer='adam',
  metrics=['accuracy']
)
model.summary()
```

Total parameters change from 14,714,688 to 18,861,490. With the 4,146,802 trainable parameters come from our fully-connected layer

![image](https://github.com/hughiephan/DPL/assets/16631121/807ef1fb-0805-496f-aaef-0aba55627094)

## Step 8: Train our model
```python
model.fit(train_data, batch_size=32, epochs=10, validation_data=val_data)
```

![image](https://github.com/hughiephan/DPL/assets/16631121/926d9627-9313-4760-894d-a71d6fff416d)

## Step 9: Predict

First we prepare the image so our model can run prediction on it, then we can use `train_data.class_indices` to output the mapping of class names to integer labels `{'WithMask': 0, 'WithoutMask': 1}`. But we want the revert of this, which we called `index_to_class`, so we can map the prediction from our model `predicted_class_index` and get it's label `WithMask` or `WithoutMask`

```python
img = image.load_img('/kaggle/input/face-mask-12k-images-dataset/Face Mask Dataset/Test/WithMask/1163.png', target_size=input_size)
img_array = image.img_to_array(img)
img_array = np.expand_dims(img_array, axis=0)
img_array = preprocess_input(img_array)
prediction = model.predict(img_array)
predicted_class_index = np.argmax(prediction)
index_to_class = {v: k for k, v in train_data.class_indices.items()}
predicted_class_label = index_to_class[predicted_class_index]
print("Predicted Class:", predicted_class_label)
```

![image](https://github.com/hughiephan/DPL/assets/16631121/31ba7179-4241-46e4-be8b-f65d4643d556)
