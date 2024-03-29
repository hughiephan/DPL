# Image Segmentation

Segmentation is the process of dividing an image into different regions based on the characteristics of pixels to identify objects or boundaries to simplify an image and more efficiently analyze it. The goal of segmentation is to simplify and change the representation of an image into something more meaningful and easier to analyze. Image segmentation is assigning a label to every pixel in an image such that pixels with the same label share certain characteristics. In this assignment, students are asked to write a program that implements algorithms for image segmentation.

## Traditional methods

![image](https://github.com/hughiephan/DPL/assets/16631121/19b1b3ae-8b09-4d10-904a-07f3235f3b14)

## Deep Learning methods

![image](https://github.com/hughiephan/DPL/assets/16631121/f850fd14-8818-4aa1-bbc8-82f6e54e5238)

## Requirements
- `Requirement 1`: Choose ONE traditional method: Threshold, Region-based, Edge-based, Watershed-based, Clustering-based
- `Requirement 2`: Choose ONE deep learning model: FCN, UNET, Viion Transformer,..
- `Requirement 3`: Choose ONE Dataset of your choice: ImageNet, COCO, CIFAR-10 and CIFAR-100, PASCAL VOC, Open Images...
- `Requirement 4`: Given an input image, apply image segmentation using a chosen traditional algorithm
- `Requirement 5`: Given an input image, apply image segmentation using a chosen Deep Learning algorithm

## Scoring Criteria
You will be graded based on Originality of your implementation: Did you implement anything new in the model or just re-use it? How much work did you put into implementing your model? Result of your implementation: Accuracy, Loss, Cross-validation Q/A to test your understanding: How well do you understand your code? How well do you understand your model algorithm? Note: DO NOT add comments in your code

## What is Snakes algorithm?
Snakes algorithm try to move snake in a direction where energy is minimum. Snake model is designed to vary its shape and position while tending to search through the minimal energy state.  Snake propagates through the domain of the image to reduce the energy function, and intends to dynamically move to the local minimum. You are required to implement a Snakes algorithm for active contours.

## What is watershed algorith?
A watershed is a transformation defined on a grayscale image. The name refers metaphorically to a geological watershed, or drainage divide, which separates adjacent drainage basins.  The watershed transformation treats the image it operates upon like a topographic map, with the brightness of each point representing its height, and finds the lines that run along the tops of ridges. You are required to implement the Watershed algorithm in python to perform the image segmentation.

## What is K-Means algorithm?
K Means is a clustering algorithm. It is used to identify different classes or clusters in the given data based on how similar the data is. Data points in the same group are more similar to other data points in that same group than those in other groups. The main idea of performing the following process is to find those areas of pixels that share the same color hue parameter value. You are required to implement the K-means for Segmentation.

## What is Mean Shift algorithm?
Mean shift implicitly models this distribution using a smooth continuous non-parametric model. The key to mean shift is a technique for efficiently finding peaks in this high-dimensional data distribution without ever computing the complete function explicitly. The idea is to replace each pixel with the mean of the pixels in a range neighborhood and whose value is within a distance d. You are required to implement the Mean shift algorithm for segmentation.
