# Analysis of Transcription Factor Binding DNA Sequences with Convolutional Neural Networks

#### Content
* [Description of the dataset](#desription-of-the-dataset)
* [CNN Architecture](#cnn-architecture)
* [Results](#results)
* [References](#references)

## Desription of the Dataset
Originaly the dataset is generated by experimental method ChIP to find genome-wide protein interactions. It also applied to transcription factors and is availabe at https://genome.ucsc.edu/ENCODE/downloads.html. It overall contains 690 datasets over different human cell types positively verified as transcription factor bound. Yet, these datasets require extensive preprocessing, since they are not annotated and need to be aligned with already experimentally discovered motifs. Besides that, occurrence of positive cases in original sequences are very rare. Such unbalanced dataset can cause biased performance. So, in practice, positive sequences are extracted, and their shuffled version are used for negative class. To preserve nucleotide count in sequneces so called "dinucleotide shuffles" method is applied Zeng et al.(2016). In addition, data augmentation techniques are employed Alipanahi et al.(2015) to render enough training data for positive sets. With all this, these steps require some knowledge of computational biology, which is not covered in my project. For this research it was used already processed dataset from Zeng et al.(2016) available at cnn.csail.mit.edu/, which consist of equal length 101 bp sequences centered on a certain motif overlapping with a one of the experimentally validated motifs.
Larger negative class is on one side helpful for avoiding overfitting, as Alipanahi et al.(2015) mention. But at the same time, training on balanced dataset leads to missclassification of majority class(negative) while testing on real dataset, causing high reported false positive cases. To prevent such deterioration of performance, in this project training and testing are conducted on balanced datasets presumably from the same distribution Zambelli et al.(2013).
Each of 101-bp input DNA sequence is one-hot encoded into a 4x101 vector and stored as 3 dimensional tensor, channels corresponding to nucleotides A, G, C and T. With respect to that, convolutional sliding means 1-dimensional convolution over 4-channel input. Following experimental specification of Alipanahi et al.(2015) before proceeding to convolution non-zero padding is implemented. The authors highlight the need to detect edges with 1 dimensional padding weight set to 0,25, which also increases output dimensionality according *((m + 2p -f)/s)-1* known formula (Goodfellow et al., 2016). 

## CNN Architecture
The CNN architecture used in benchmark studies achieved by far the best results with rather simple one convolution layer(Alipanahi et al.,2015, Zeng et al. 2016). Thus the model is much simpler than the comparable models for computer vision, for example Krizhevsky et al.(2012). Recalling the commutation risk between model complexity and overfitting, the task is to decide on most parsimony model. As known, the depth and complexity of a deep network is relational to the complexity of the features to detect and existence of sufficient training data samples. Regarding the first, larger number of kernels in a single layer as motif scanners can moderate the effect of less convolutional layers by extracting existing features. However, as experiment of Zeng et al. (2016) reveals, adding too many kernels leads to saturation in convolutional units. Comparing deeper convolutional networks employed by Quang and Xie (2016) *(1000bs, 3 layers, 960 kernels*, it possible to infer for biological sequences: The longer sequence to bigger number of layers and kernels are needed. In current dataset in a single sequence incorporates less variety of motifs, hence feature detection is reasonable  with less layers and small number of kernels.
Nevertheless, to accommodate the possible effect of more complex architecture on the performance of the model, I explored three variations of model complexity: One layer 16 kernels, 2 layers with 16 and 32 kernels, 3 layer with 16,32,64 kernels respectively. Last Convlayer were pooled using global max pooling unlike max pooling of simple model. Employing more kernels in deeper network can be justified also by reduced number of parameters. Hence such approach will moderate the risk of overfitting, if all models of different complexity are trained on the same amount of data. 
For the trained convolutional networks rectification Relu is used as non linear transformation in convolutional layers. It is most widely used activation function in DL problems currently. Following heuristic approach CNN model on image classi-fication problem Krizhevsky et al.(2012) proposed to drop the learning rate by dividing it to 10 each timethe  validation  error  rate  stops  decreasing.

## Results
Initially, for the evaluation it was selected to compare accuracy of classification over defined three architectures with 4 different combinations of learning rate and kernel length, as discussed in previous section. Choice of accuracy as a measure can be justified by the strictly balanced nature of the dataset. Because it infers baseline accuracy around 50%, it was expected that simple CNN will be able to outperform. However, as the benchmark study Zeng et al. (2016) warns complicated architecture can show poorer performance than simple one. According to preliminary intention to draw statistical significance test between performances of various architectures, training and 3-fold cross validation is executed over 10 datasets from original sources. Choice of the independent 10 datasets is reasoned with the restriction to use k-fold cross validation accuracies as independent samples for statistical test. Resampling for k-fold cross validation leads to occurrence of an observation in several training dataset. Thus, samples are dependent, and as Demsar (2006 highlight, it can lead to underestimated variance and elevated Type I error.
Although results illustrated on boxplot implicitly don't show any improvement of accuracy compared to baseline, moreover,they reveal that complex architecture doesn't perform significantly worse or better from the average performance according to Wilcoxon test (*p-value = 0,57*).

![Accuracy of different architectures ](https://github.com/sipih/genome-sequence-classification/blob/main/Accuracy_comparison_box_plot.png)

In practice, best predictive result was achieved from milestone model depicted above on the learning curve over epochs. The model tends to predict better, on small dataset around 70000 sequences with simple architecture - 16 kernels with size 24 and 1 convlayer. As loss is dropping smoothly, no stagnation of loss is visible from plot. I can conclude that is neither overfitting or underfitting to learn around 15000 parameters. As I was expecting, in this experiment using Global max pooling on last layer of convolution instead of  local pooling with window size 3, showed worse results. Following the benchmark experiment, I adopted global pooling but argued that, since it reduces parameters, it can cause possible underfitting, as it was revealed for 12 experiments.

## References
>Alipanahi, B., Delong, A., Weirauch, M. T., and Frey, B. J. (2015).  Predicting the sequence specificities ofdna-and rna-binding proteins by deep learning. Nature biotechnology, 33(8):831–838. <br/>
>Demsar, J. (2006). Statistical comparisons of classifiers over multiple data sets.Journal of Machine learningresearch, 7(Jan):1–30.;<br/>
>Goodfellow,I.,Bengio,Y.,and    Courville,A.    (2016). Deep Learning.MIT Press.http://www.deeplearningbook.org.Hinton, <br/>
>Krizhevsky, A., Sutskever, I., and Hinton, G. E. (2012). Imagenet classification  with  deep  convolutionalneural networks. InAdvances in neural information processing systems, pages 1097–1105. <br/>
>Quang, D. and Xie, X. (2016). Danq: a hybrid convolutional and recurrent deep neural network for quanti-fying the function of dna sequences.Nucleic acids research, 44(11):e107–e107<br/>
>Zeng, H., Edwards, M. D., Liu, G., and Gifford, D. K. (2016). Convolutional neural network architecturesfor predicting dna–protein binding.Bioinformatics, 32(12):i121–i127.<br/>
Zhuang, Z., Shen, X., and Pan, W. (2019). A simple convolutional neural network for prediction of enhancer–promoter interactions with dna sequence data.Bioinformatics, 35(17):2899–2906.<br/>
