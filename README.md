# **Linguistic Labyrinths and Minotaurs, Linear A Optical Character Recognition**

![](https://github.com/aibatchelor22/LinearAOCR/blob/main/image001.jpg)

Palace of Knossos, Crete

[Source](https://www.lonelyplanet.com/greece/crete/knossos/attractions/palace-of-knossos/a/poi-sig/504778/359431)

The Linear A language comes from the Minoan civilization of Crete. Most examples are from clay tablets excavated from sites such as the Palace of Knossos, the site of the labyrinth where the legendary minotaur lived. The language was in use from about 1800 to 1450 BCE. The language has never been deciphered and the known corpus is very small. As of 2002, Schoep et al. note that there were 1427 known Linear A documents with a total occurrence of 7362-7396 characters (Schoep 2002, 38). The SigLA database of Linear A documents hosted by the University of Rennes, France includes approximately 300 distinct character types.

I derived my dataset from "[LinearA Explorer](https://github.com/mwenge/LinearAExplorer/)" created by Rob Hogan. The data therein was sourced from three digitized volumes of Recueil des Inscriptions en Lineair A (Louis Godart and Jean-Pierre Olivie 1970) hosted by Collections de l'Ecole française. After cleaning, my dataset included a corpus of 1295 documents with a total of 3530 characters, including 279 distinct character types. My dataset thus included most of the known Linear A documents and annotations for around half of the known set of text.

Linguists have drawn parallels between analagous characters in the Minoan Linear B language. Unlike Linear A, Linear B has been deciphered as Mycenaean Greek. The Linear A character set has been mapped into 384 [unicode signifiers](https://en.wikipedia.org/wiki/Linear_A_(Unicode_block)). Of these, 31 are Linear B analogues. A total of 28 of these linear B analogues were present in my full dataset. It is estimated that 60-70 of the characters of Linear A are phonetic and the remainder are numbers and ideograms. Linear A primarily appears to be written from left to right. However, some texts are written from right to left, or as boustrophedon, i.e. lines of alternating direction "as the ox plows." In some cases, the text may even be written in spirals.

![](https://github.com/aibatchelor22/LinearAOCR/blob/main/image002.jpg) 
![](https://github.com/aibatchelor22/LinearAOCR/blob/main/image003.png)

[Source](https://en.wikipedia.org/wiki/Linear_A)

Researchers at the Oriental Institute of the University of Chicago conducting the [DeepScribe](https://datascience.uchicago.edu/news/ancient-language-processing-teaching-computers-to-read-cuneiform-tablets/) project have explored methods of character recognition for clay tablets of Elamite cuneiform excavated from the Persepolis site in Iran. Their dataset included 6000 annotated images of clay tablets with over 100,000 characters. Elamite cuneiform script includes 206 character types. The DeepScribe project has primarily explored techniques for recognizing cropped instances of single characters with convolutional neural networks such as ResNet50. They have created models with around 80% accuracy. In addition, they have explored methods of instance detection using detectron2. I used a similar approach with a similar dataset, including annotated images of clay tablets with a comparable number of character types. However, my dataset inlcuded significantly fewer images with significantly fewer characters and therefore presented a unique challenge.

![](https://github.com/aibatchelor22/LinearAOCR/blob/main/image004.png)

[Source](https://datascience.uchicago.edu/news/ancient-language-processing-teaching-computers-to-read-cuneiform-tablets/)

**Data Source and Description**

Nobody has ever created optical character recognition software for the Linear A language. However, unicode values have been assigned to the characters and these are the basis I used for class identifiers. Rob Hogan has constructed a very useful set of annotated data, including transcriptions. I was able to use this set of data to construct a dictionary of images of clay tablets inscribed with Linear A, along with bounding box data for each character and its unicode value. I did some cleaning to remove stop characters (e.g. punctuation and spaces) from the inscription data and matched each character with a single bounding box. I also removed annotations that did not match the number of bounding boxes. When this was complete, my dataset included 1295 images.

The character set in my corpus had a large class imbalance. There were 279 unique characters in the cleaned dataset. Of these, 9 were hapaxes, i.e. they only had one instance. The most frequent characters had 350 or more instances.

**Methods**

I explored three approaches to recognizing characters.

First, I used [detectron2](https://github.com/facebookresearch/detectron2) software from Meta (Facebook) to create a model using instance recognition. The model can identify a character and its corresponding bounding box within an image of a clay tablet inscribed with Linear A text.

![](https://github.com/aibatchelor22/LinearAOCR/blob/main/image005.jpg)

[Source](https://medium.com/@hirotoschwert/digging-into-detectron-2-47b2e794fabd)

![](https://github.com/aibatchelor22/LinearAOCR/blob/main/image006.jpg)

Training set example

Second, I used an EfficientNetB0 to classify cropped images of single characters with a convolutional neural network.

![](https://github.com/aibatchelor22/LinearAOCR/blob/main/image007.jpg)

[Source](https://www.researchgate.net/figure/The-EfficientNetB0-network-architecture_fig8_346296594)

![](https://github.com/aibatchelor22/LinearAOCR/blob/main/image008.jpg)
![](https://github.com/aibatchelor22/LinearAOCR/blob/main/image009.jpg)
![](https://github.com/aibatchelor22/LinearAOCR/blob/main/image010.jpg)
![](https://github.com/aibatchelor22/LinearAOCR/blob/main/image011.jpg)
![](https://github.com/aibatchelor22/LinearAOCR/blob/main/image012.jpg)

Training image examples

Third, I used a shared encoder neural network with an EfficientNetB0 base model to classify cropped images of single characters using a pair of convolutional neural networks to perform "few-shot" detection, a useful techique for very small training datasets. Such an implementation is described by [Mehmood et al](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7071616/). For each image, two different augmentations are applied. The two augmented images are each input to an EfficentNetB0 base model which does not include the default dense layers. The logits are input to a max pool layer to create embeddings. The two embeddings are concatenated and input to a series of two dense layers, and finally input to a softmax layer for classification. For validation and testing, I used one image without augmentation and one image with augmentation as inputs.

![](https://github.com/aibatchelor22/LinearAOCR/blob/main/image013.jpg)

I tested each model with three different values for the minimum number of instances for each class, more specifically, 20, 40, and 80 instances. A higher number for this value meant a smaller number of classes represented in the data.

**Results**

I evaluated models based on average precision. The models with a minimum of 20 or 40 instances failed to recognize most of the less frequent characters. The EfficientNet based models with a minimum of 80 characters performed fairly well, but each model only included 7 character classes. This is not enough for an optical character recognition tool that includes the 60-70 base characters of Linear A.

| Instances(min) | # Classes | Method 1 AP | Method 2 AP | Method 3 AP |
| --- | --- | --- | --- | --- |
| 10 | 76 | 0.02 | 0.02 | 0.02 |
| 20 | 45 | 0.08 | 0.03 | 0.04 |
| 40 | 23 | 0.08 | 0.09 | 0.16 |
| 80 | 7 | 0.09 | 0.49 | 0.44 |

I explored an additional approach where I used ZCA whitening as preprocessing for Method 2 and Method 3.  Keras has a very convenient tool for this preprocessing in the ImageDataGenerator class.  ZCA whitening improved the results very significantly, especially for Method 2.

| Instances(min) | # Classes | Method 2 AP | Method 3 AP |
| --- | --- | --- | --- |
| 10 | 76 | 0.14 | 0.02 |
| 20 | 45 | 0.26 | 0.07 |
| 40 | 23 | 0.43 | 0.11 |
| 80 | 7 | 0.55 | 0.44 |


**Conclusions**

It is apparent that the present corpus of Linear A text is too small to create a model for OCR of even the presumed base phonetic set using conventional techniques. Recognition only performed well for characters with 80 or more instances. ZCA whitening improved the results very significantly for a conventional EfficientNet classifier, but not enough to create a particularly useful software tool.  Either a larger data set or different techniques are warranted. Collection of additional data is limited by archaeological excavation and annotation on the part of researchers and is therefore not a practical option. One tool that may be promising is semi-supervised learning.  Other tools that may be worth exploring include "few shot" or "one shot" detection with shared encoders that use either [contrastive loss](https://www.cs.toronto.edu/~hinton/absps/simclr.pdf) or [triplet loss](https://arxiv.org/pdf/1412.6622.pdf). Such techniques have been successful for classification with very few or even single instances of each class in domains including signature verification and facial recognition.

**References**

Schoep, Ilse. 2002 The Administration of Neopalatial Crete: A Critical Assessment of the Linear A Tablets and Their Role in the Administrative Process (Salamanca: Ediciones Universidad Salamanca)
