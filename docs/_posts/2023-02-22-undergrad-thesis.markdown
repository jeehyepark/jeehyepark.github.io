---
layout: post
title:  "Building Detection with Machine Learning Across Contexts (Undergraduate Thesis)"
date:   2023-03-05 18:23:12 -0800
categories: jekyll update
image: /assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/1_aerial_image.png
---
<h1> Summary </h1>
In this study, we explored the impact of geographical differences on the problem of object detection in aerial imagery. These geographical differences may be different cities or different urban/suburban environments. Machine learning algorithms can be used to detect objects in aerial imagery, but detection performance is limited to what can be learned from the data the algorithm is trained on.  Thus, when tested on an image with geographical differences from the training images, such as different rooftop materials, different landscapes, and different objects that could be confused with buildings, performance may drop significantly. In my work, I found that this was generally the case. Performance was best when the training and testing data had minimal geographical differences. In cases that the performance is poor, techniques from transfer learning could help in identifying buildings in places that are geographically different from data with available ground truth.

<h1> Data </h1>
In this work, the source and target domains are different combinations of eight satellite images from four different cities in the United States: Norfolk (VA), New Haven (CT), San Francisco (CA), and New York City (NY). The choice of cities also allows for an even division of images into categories of urban environments (New York and San Francisco) and suburban environments (Norfolk and New Haven). The aerial images were high-resolution (0.3m resolution), and large in size (5000x5000 pixels). Two images were selected from each city, one of them used in the training dataset and the other in the testing dataset. The images used are a subset of a larger dataset comprised of 25 images from seven cities in the United States, and over 44,000 buildings [17]. The images used in this study are depicted in Figure 1. The entire dataset included the high-resolution ortho-rectified satellite image and building outlines for ground-truth, obtained from Open Street Maps[18], an openly licensed map which mappers collaborate in annotating. After the data was collected, it was thoroughly inspected and manually edited.

{:refdef: style="text-align: center;"}
![City Aerial Images](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/1_aerial_image.png)
{: refdef}


The diversity and complexity of the dataset (both training and testing) are integral components for this study, and were prioritized in the selection of cities to use for experiments. The presence of bodies of water, trees and other vegetation, shadows, and roads in some of the images contribute to the complexity of the images.

{:refdef: style="text-align: center;"}
![Building Statistics](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/2_building_stats.png){: width="500" }
{: refdef}


<h1> Methods </h1>
The procedure for pixel classification used in this work is commonly used for object detection. This procedure is outlined in Figure 3. The input data is the aerial images, used for training and testing. Information is extracted from each pixel in the image by finding specific color, texture, and shape features. The features and known labels associated with the training data are used to train a Random Forest Classifier, which classifies the testing data. The results are post-processed to refine the results from the Random Forest.

{:refdef: style="text-align: center;"}
![Method Flow Diagram](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/3_method_flow.png){: width="650" }
{: refdef}

<h4> Feature Extraction </h4>
Prior to applying the classification algorithm is the feature extraction step. Features are descriptive values that can be used to differentiate objects. A vector of features is extracted for each pixel, containing information about the pixel and its surrounding pixels [19]. The features that were selected were color, texture, and shape descriptors, because they are most related to an object’s identity [20]  and are what humans look for when identifying an object. Color features include the mean values of the central pixel’s direct surroundings for each channel in the RGB (Red, Green, Blue) and in the HSV (Hue, Saturation, Value) spaces. Texture features include the standard deviations for each of these channels, and the entropy (randomness) of the pixel’s surroundings. Shape descriptors include edge information, such as the gradient of pixels in a range of distances away from the central pixel. A comprehensive list of all selected features is available in Appendix A. The features are used as input to the random forest classifier, which outputs a probability that the pixel is a target.


{:refdef: style="text-align: center;"}
![Building Pixel Feature Scatterplot](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/4_feature_scatter1.png){: width="750" }
<h5>Image caption</h5>
{: refdef}



{:refdef: style="text-align: center;"}
![Building Pixel Feature Scatterplot (2)](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/5_feature_scatter2.png){: width="750" }
{: refdef}

Figure 4 and Figure 5 show feature maps for six of the features that were used for feature extraction. The original image is in RGB space, in which pixel values represent how red (R), green (G), and blue (B) they are. Pixels can be transformed onto the HSV space, in which values represent the pixel’s hue (H), saturation (S), and value (V). The hue refers to the “true color,” the saturation is the amount that the color has been diluted with white, and the value is the brightness. HSV is a common color space used for computer vision problems because it is a good representation of how humans perceive color [21]. Figure 4 shows the average values of hue, saturation, and value of building pixels and their adjacent pixels. As can be seen by the figure, each image looks distinct in the hue-saturation and hue-value spaces. In the feature spaces shown, Norfolk and New Haven look the most similar to each other, while New York also includes similar colors. San Francisco seems to be an anomaly with completely different colors, represented in the color of the scatter points and in the shape of its domain in the feature spaces.

Figure 5 shows the standard deviation of HSV of the surrounding pixels. For Norfolk and New York, a lot of variance is in Hue and for San Francisco, more variance is in Saturation and Value than the other images. Both feature maps show how each image differs in appearance of the buildings. When large differences lie in the feature spaces between the training and testing data, we can infer that classification results will be very poor. In the visualizations, San Francisco is the anomaly in its domain space, with a high density of buildings and different building appearances. This agrees with the finding, discussed below, that the San Francisco image does poorly as training and testing.

By analyzing the feature maps for each classification, a greater understanding of how discrepancies in the features lead to discrepancies in the classification results can be obtained. It can also allow for transfer learning methods which tweak specific features. In the case where X_S≠X_T, we can apply a feature mapping so that X_S→X_T. This transform must be done with preexisting knowledge of the test data set, and predictions to how the features would be related to the training data features.

<h4> Classification </h4>

The features that were extracted are input to a classifier, a learning model designed to place the observations into categories (such as building and non-building). The purpose of the classifier in this problem is to label the observations, which are image pixels, as building pixels or non-building pixels. The random forest classifier is an ensemble method which uses several weak classifiers, classification trees. A bootstrapped sample of the input data and a random sample of predictors is chosen to train each classification tree. The trees assign scores to out-of-bag data, data that was not included in training. There is a “voting” of the outputs from the classification trees to classify the observation. If the vote is that the pixel is in the object of interest (ie. building), then the classification tree will vote a “1.” The random forest is well-fit for remote sensing, because of its ability to handle large datasets and many predictors [23]. The randomness introduced during bootstrapping and random predictor sampling prevent a small number of the predictors from dominating, and lessen the possibility of overfitting. A remote sensing application of random forests has proved to work well in is the classification of land cover (types of crops) [1]. The classifier outputs a matrix of confidence values in [0,1], the probabilities that the pixels are target pixels.


<h4> Post-Processing </h4>
Because the pixel values in the confidence map output by the classifier were determined independently of each other, post-processing is necessary for using regional information for refinement. For example, a low-confidence pixel in a high-confidence region or a high-confidence pixel surrounded by low-confidence pixels are likely to have been labelled inaccurately. These pixels are called salt-and-pepper noise. As such, this confidence map from the classifier benefits from further refinement by image processing techniques in the post-processing step. A median filter removes salt-and-pepper noise from the confidence map [20]. Opening, a morphological operation, is then used to eliminate such noise and to enhance the object boundaries. Closing, another operation, can fill in the gaps or holes in blobs of majority high confidence values. After applying the median filter, and the opening and closing operations, the resulting confidence map has less noisy pixels and more refined object boundaries, as shown by Figure 7. The results of the post-processing step are ready for evaluation and/or for binarization. Specific details on each feature extracted and explanations of the post-processing techniques will be included in the Appendix.

{:refdef: style="text-align: center;"}
![Building Pixel Feature Scatterplot (2)](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/6_building_conf_map.png){: width="450" }
{: refdef}

To test the effect of the contexts of the training and testing data, different combinations of training and testing images were used in a classification algorithm. For each experiment, training data and testing data are each one 5000x5000 pixel image. The combinations are defined by four different experiments in which training images are from (1) the same city, (2) the same (urban/suburban) environment and different city, (3) different environment as the testing images, or (4) leaves only the training city out. The flowchart below summarizes the steps used in the experimental procedure.

*Training/Testing Combination Experiments*

The following list of experiments are designed to see the effects of using different images, including those from different cities, and from different urban/suburban environments as training and testing data on classification performance. The first experiment is to assess the performance when training on a different image from the same city as the testing image, the second is training on an image from a different city of the same urban/suburban environment, the third is training on an image from a different city of a different urban/suburban environment, and the last is training on all other cities except for the testing image. Table 1 below shows the specific training and testing images that were used for each experiment. In total, 20 tests were run.

1.	**Same City**: The testing data is a different image from the training data but are from the same city. They will therefore have many similar features. For example, an image from Norfolk (Norfolk1 ) will be paired with another image from Norfolk (Norfolk2).
2.	**Same Environment, Different City**: The testing data is of a different city, but of the same (urban/suburban) environment. It is urban if the training data is urban and suburban if the training data is suburban.
3.	**Different Environment**: The testing data is of a different (urban/suburban) environment as the training data. This tests for the effects of cross-context training and testing.
  - a.	Testing Suburban
  - b.	Testing Urban
4.	**Leave Same City Out**: The training data is an aggregation of training images of all of the cities, excluding the city being tested. In this case, the testing data belongs to the minority category in the training data. Because the variable of interest is the diversity, not the quantity, of the data, one-third of each training image is used to conserve the quantity of the data.

{:refdef: style="text-align: center;"}
![Building Pixel Feature Scatterplot (2)](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/7_experiments.png){: width="400" }
{: refdef}

*Performance Metrics*

The performance metrics used to evaluate each test are the standard machine learning performance metrics: the ROC (Receiver Operating Characteristic) curve, the AUC (Area under the ROC), and the PR (Precision-Recall) curve. Both the ROC curve and the PR curve are necessary to fully evaluate performance in this study due to a large class imbalance (more non-building pixels than building pixels) for any given image. In the case of such class imbalance, the ROC curve could look accurate even when all positives because TPR does not account for false positives, but precision accounts for the false positives. Images of the confidence maps with polygon outlines overlaid are used to evaluate the performance qualitatively.

<h1>Results</h1>

In experiments 1-3, every training image is paired with every testing image. Figures 8 and 9 show the ROC and PR curves for each test. The resulting AUCs after classification of each pairing is showed in Figure 10. A table listing all of the AUC values for each experiment can be found in Appendix A.

A good ROC will show a line is very close to the upper left corner, indicating that the probability of detection can be high for low false alarms. After thresholding, a greater fraction of building pixels will be classified as buildings and greater fraction of non-building pixels will be classified as non-buildings. For a good classifier, the area under the ROC curve, or the AUC, will be close to 1. A good PR curve displays a line close to upper right, indicating both high precision and high recall. After thresholding, the ratio of building pixels to the ones classified as buildings and ratio of pixels classified as buildings to real building pixels are both high.

{:refdef: style="text-align: center;"}
![Building Pixel Feature Scatterplot (2)](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/8_ROC1.png){: width="600" }
{: refdef}

{:refdef: style="text-align: center;"}
![Building Pixel Feature Scatterplot (2)](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/9_ROC2.png){: width="600" }
{: refdef}

Experiment 1: Same City

It can be seen in Figure 8 that for each testing image, the best training image is consistently the image from the same city. For experiment 1 (Table A.2 in Appendix A), the AUCs were very high, all above AUCs of 0.85. From least to greatest, the AUCs were 0.860 (Norfolk), 0.877 (New Haven), 0.889 (San Francisco), and 0.928 (New York).  The upper left plot of Figure 8 shows the ROCs for experiment 1. The ROCs appear to be the best out of all of the ROCs for all experiments. In general, precision is also high for all tests in experiment 1, as shown in the upper left plot of Figure 9. For training on testing images from the same cities, data from urban areas outperform suburban areas in both detection and precision. Out of all tests, Norfolk did the worst in both detection and precision.


**Experiment 2: Same Environment, Different City**

For experiment 2, we found that performance was very good when suburban images were used to train and test. Training on New Haven and testing on Norfolk yielded an AUC of 0.743 and training on Norfolk and testing on New Haven yielded an AUC of 0.853. Although both combinations showed high performance, training on New Haven and testing on Norfolk did much better than the reverse. Confidence maps for testing on New Haven, and training on each suburban city is shown in Figures 8 and 9. The confidence map also confirms the pixels in New Haven were well classified for training on Norfolk. For urban areas, performance was very poor: the AUC of training on New York and testing on San Francisco was 0.519, which is close to the chance diagonal and the AUC for training on San Francisco and testing on New York was 0.598, slightly better but is still considered very poor classification. Testing on NY did better for ROC but testing on San Francisco did slightly better for PR. Training on Norfolk and testing on New Haven showed a markedly good PR curve, while the PR curves for the other cities were not good.

{:refdef: style="text-align: center;"}
![Building Pixel Feature Scatterplot (2)](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/10_conf_map_new_haven.png){: width="500" }
{: refdef}

**Experiment 3a: Different Environment, Testing Suburban**

For experiment 3a (lower left ROC and PR plots), urban images were used to train and suburban images were used to test. In general, the algorithm did very poorly for training urban and testing suburban. The best result in this experiment was for training on New York and testing on New Haven, which yielded an AUC of 0.697. Training on New York and testing on Norfolk did slightly worse, with an AUC of 0.572, and an ROC curve that is nearly chance diagonal. Training on San Francisco resulted in ROC curves worse than the chance diagonal and AUCs lower than 0.5 (0.4876 for training on either suburban images). These results can be confirmed by the confidence maps of training on San Francisco and testing on New York, shown in Figure 10. For training suburban images, New York does marginally better than San Francisco. For all tests in this experiment, the PR curves showed very poor performance.

{:refdef: style="text-align: center;"}
![Building Pixel Feature Scatterplot (2)](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/11_conf_map_SF.png){: width="350" }
{: refdef}

Experiment 3b: Different Environment, Testing Urban

For experiment 3b (lower right ROC and PR plots), suburban images were used to train and urban images were used to test. All trials in this experiment also did very poorly. Except for testing New Haven and training New York, testing urban areas generally did better than for testing suburban areas. The ROC curves show that they are all a distance away from the chance diagonal, and all of similar performance. Testing on New York yielded AUCs around 0.65 (0.648 for training on Norfolk and 0.656 for training on New Haven). Testing on San Francisco yielded very different AUCs depending on the training data, 0.610 for training on New Haven and 0.695 for training on Norfolk. For each testing image, the combination of training and testing that had highest performance corresponds to the worse PR curve. When testing on New York, training on New Haven had slightly better detection than training on Norfolk but a significantly worse PR curve. Similarly, for testing on San Francisco, training on Norfolk had much better detection than training on New Haven, but also a significantly worse PR curve. Thus, when detection is higher, more pixels (not necessarily the correct pixels) are given high confidence values.

A summary of all of the AUCs for each combination of training and testing data can be found in Figure 12.

{:refdef: style="text-align: center;"}
![Building Pixel Feature Scatterplot (2)](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/12_AUC_exp.png){: width="550" }
{: refdef}

{:refdef: style="text-align: center;"}
![Building Pixel Feature Scatterplot (2)](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/13_ROC_LOCO.png){: width="550" }
{: refdef}

Experiment 4: Leave One City Out

For experiment 4, “Leave One City Out,” the training data was a combination of images from the cities excluding the city used as testing data. The size of the training data was preserved by only using a fraction of each training image. As can be seen, testing New Haven performed extremely well, with an AUC of 0.907. Testing on Norfolk also did well with an AUC of 0.869. These results are alarmingly high, and show performance comparable to training on images from the same city from Experiment 1. Testing on the urban cities, however, did not do very well, just above the chance diagonal. Testing on San Francisco had an AUC of 0.587, and testing on New York had an AUC of 0.558. The PR curves also show similar results: the precision for testing suburban cities are high while the precision for testing urban cities are very low.

{:refdef: style="text-align: center;"}
![Building Pixel Feature Scatterplot (2)](/assets/images/2023-02-22-building-detection-with-machine-learning-across-contexts/14_hist_conf_values.png){: width="650" }
{: refdef}

The distribution of buildings labelled in the training image has a large effect on the distribution of confidence values resulting from the classifier. The histogram shown in Figure 14 above shows the histogram distributions of confidence values for each combination of training city and testing city. The ideal histogram for a perfect classifier has two peaks: one at 0 and another at 1, with the ratio of the bar heights equal to the ratio of non-building to building pixels. Although none of the classifiers portray perfect classifiers, the histograms on the horizontal are closest to the ideal histogram, with large peaks at 0 for non-building pixels and the rest of the pixels are distributed across (0,1). For an ideal classifier, the average confidence values after classification and the actual proportion of building pixels out of total pixels are exactly equal. Along the horizontal, the two values are very close to each other.

It is interesting that for each training image, the resulting distribution of classifier scores are very similar for any testing image that is used. When training on Norfolk, there are a large number of pixels with a low value, and number of pixels decreases with increasing classifier score. It shows a particularly good histogram for testing New Haven in which the there is a large peak at 0. The average confidence values for each testing image are similar to the actual proportions of building pixels. When New Haven is the training image and Norfolk is a testing image, the distribution looks very uniform across (0,1), which is not good. The average confidence value is much higher than the actual proportion of building pixels, which indicates that many non-building pixels were given high confidence values.

Training on San Francisco shows that when testing on other images, the peak of density lies in a high confidence value region. For all images, the average confidence values are extremely overestimated. This could be because San Francisco is very dense in building pixels, and the class imbalance is uneven between the training dataset and the testing dataset. It could also be a result of the specific features. Training on New York showed similar performance to training on San Francisco, but to a lesser extent. The density peaks lie at a region of much lower confidence values. The exception is for another New York image and for San Francisco.

<h1> Conclusions </h1>
This work clearly demonstrates the challenge of object detection in satellite imagery when training and testing on different geographic locations. The results show that the process of classifying buildings in aerial imagery using training and testing data from the same city generally achieves high accuracy and very good ROC performance, however, this is not necessarily true when training and testing on geographically different data. Even when the images used for training and testing are geographically different, performance will be high if the images have similarities. In this work, I explored two factors could contribute to similarity of geographically different images: distribution of building pixels, and similarity of the feature maps. It can be concluded that transfer learning should be applied to the building detection problem, but may have the greatest impact on performance if the geographic locations of the images are dissimilar in building distribution and/or feature map.

A few other findings were seen through the results of the experiments. Performance is not the same when the training data is switched with the testing data. Some of the images performed better as training data, and others as better testing data. This is due to inherent characteristics of each image. It was also shown that when a subset of training data is similar to the testing data and a majority of it is dissimilar, performance can still be maintained. Therefore, when choosing a subset for the training data, it should be selected in a way that maximizes diversity.  

<h1> Acknowledgements </h1>

I would like to acknowledge two project teams I was honored to be part of and were integral in making this work possible: the summer Data Plus team and the Bass Connections team. The ground-truth dataset would not have been possible without the summer Data Plus team, and the algorithm for classification would not have been possible without collaboration from the Bass Connections team. They were the most supportive, fun, and bright teams to work with. I would like to acknowledge Dr. Leslie Collins for all of her work as a research advisor. I am so thankful for her tips and helpful advice during our team meetings. I would like to thank Dr. Jordan Malof for all of his expert input into the project, and also being available to show me new possibilities whenever I felt I had hit a wall. Last but not least, I would like to acknowledge Dr. Kyle Bradbury for overseeing every aspect of this project, for putting in so much time to helping me learn as a budding researcher. None of this would have been possible if it was not for his guidance, his contagious passion for machine learning and energy, and his extreme patience and kindness.
