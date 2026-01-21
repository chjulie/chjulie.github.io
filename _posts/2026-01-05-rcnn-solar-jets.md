---
title:  "A recurrent convolutional neural network for coronal jet identification"
date:   2026-01-05 18:44:09 -0800
categories: projects
permalink: :categories/rcnn-solar-jets.html
hero_image: assets/images/2026-01-05-cover.png
---
{% include mathjs.html %}

In 2023 and as part of EPFL’s CS-433 Machine Learning course, I had the privilege of collaborating with European Space Agency’s (ESA) Dr. Sophie Musset along with two other classmates. Her research focuses on coronal (or solar) jets, which are plasma eruption in the Sun’s atmosphere and are linked to solar particle events. A current limitation to her research was the ability to process the increasing amount of data from the Solar Dynamic Observatory (SDO), a NASA satellite that records images of the sun with a temporal resolution of 12 seconds. The overwhelming amount of images prevented her from having access to an exhaustive list of observed solar jet events. She relied greatly on a citizen science, where a team of volunteer manually annotated image sequences when a solar jet was present. Despite being extremely valuable, this approach was too slow for the amount of data to annotate.

The project lended itself perfectly for binary image classification, and more specifically image sequence classification, as jets are typically recognised by their dynamic motion. We built a dataset of sequences of 30 images taken by the SDO/AIA (Advanced Image Assembly) instrument, taken every 24 seconds, thus spanning 12 minutes each. The manual annotations from the volunteer provided labels for the training dataset, which consisted of 883 positive and 883 negative sequences. It was randomly split between training (70%), validation (15%) and testing (15%) sets.

In the field of deep learning for image sequences classification, Recurrent Convolutional Neural Networks (RCNN) have proven to be highly effective. Convolutional networks have shown their performance in single image classification tasks. The Long Short-Term Memory (LSTM) recurrent layer enables the model to extract temporal information. LSTM layers are widely used when working with time series. We took inspiration from the model design in John A. Armstrong et
al.[^1] to build our own. The output of the neural network is mapped to the domain [0,1] through a sigmoid function and can be interpreted as a probability of the image sequence containing a solar jet.

<figure id="fig-cnn" style="text-align: center; max-width: min(1200px,90%); margin: 2em auto;">
  <img src="/assets/images/2026-01-05-cnn.png" alt="Architecture of the CNN model">
  <figcaption style="margin-top: 0.5em; font-size: 0.9em;">
    <strong>Figure 1.</strong> Architecture of the recurrent convolutional neural network (RCNN)
  </figcaption>
</figure>

The model was trained to minimise the Binary Cross-Entropy loss function, a common loss for binary classification. 
<div>
    $$BCE(y,p) = -(y \cdot log(p) + (1 - y) \cdot log(1-p))$$
</div>

The training was capped at 100 epochs and best results were found using a batch size of 2. A grid search was conducted to determine the optimal optimizer, scheduler and initial learning rate. The F1 score of the validation dataset was used to select the optimal configuration, which consisted of an initial learning rate of <span>$$10^{-4}$$</span>, a cosine annealing learning rate scheduler, and the AdamW optimizer. The classification threshold was the last hyperparameter needing to be set. In line with our higher risk tolerance regarding false positive (non-jet events that could easily be identified once the classification was done), we selected a classification threshold that yielded a high F1 score while favoring the identification of positive events. The chosen threshold of 0.1 achieved those pre-requisites, maintaining a F1-score of 0.8949 on the validation set.

To evaluate the generalization performance of our model, we used the test, which comprised of 260 image sequences (both positive and negative). The final model reached an accuracy of 92.31% and a F1-score of 0.9286 on this set. The confusion matrix, presented in [Figure 2](#fig-confusion-mat), reveals only 2 false negative classification. This result aligns well with our decision to skew the output distribution towards a higher sensitivity in detecting jet events.

<figure id="fig-confusion-mat" style="text-align: center; max-width: min(500px,50%); margin: 2em auto;">
  <img src="/assets/images/2026-01-05-confusion.png" alt="Confusion matrix">
  <figcaption style="margin-top: 0.5em; font-size: 0.9em;">
    <strong>Figure 2.</strong> Confusion matrix for the test set
  </figcaption>
</figure>

Our project introduced a Recurrent Convolutional Neural Network (RCNN) for identifying solar jets using high-resolution Solar Dynamic Observatory (SDO) data. It provides a fast and reliable alternative to the current manual classification method by citizen science, and will be used to automate the processing of the ever-increasing amount of data in the Heliophysics events knowledgebase. Not only this project had a positive impact on this research field, it also allowed our young team of three student to design and train a neural network from scratch. The reflexions behind the design choices and the hyperparameter tuning was a valuable learning experience. It taught us to think about the structure of underlying data and the objective at hand, while not losing sight of the theory and rigour thaught in the course.

Download our [project report]( /assets/pdfs/CS433-report.pdf ).

[^1]: Armstrong, J.A., Fletcher, L. Fast Solar Image Classification Using Deep Learning and Its Importance for Automation in Solar Physics. Sol Phys 294, 80 (2019). https://doi.org/10.1007/s11207-019-1473-z