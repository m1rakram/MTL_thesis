<h1> Deep Learning in Mars Exploration/Navigation: Rover Multi-Task Learning and Drone Semantic Segmentation on Synthetic and Real Datasets with Domain Adaptation Techniques </h1>

Project has been completed in AIKO-Autonomous Space missions company by Mirakram Aghalarov in collaboration with Politecnico di Torino



<h2> Abstract </h2>

Mars exploration is one of the trending topics in all fields of technology in which Computer science, mechatronics, aerospace engineering have combined workforce to execute these tasks efficiently. Computer vision side has responsibility to keep the mission alive and protect rovers from any occasions that give damage. That is why, high amount of research, accurate datasets and efficient ways should be found and implemented. We divided our work into 2 parts: Multi-task learning on rover study with domain adaptation, Semantic Segmentation on drone study with domain adaptation. 

Navigation is the main objective of rover study in which a Deep Learning model should identify the different types of terrain to help in the definition of a safe path for rover exploration. Semantic segmentation is used for this purposes while AI4Mars dataset accuracy was not sufficient to identify the objects. That is why, synthetic datasets are used for this project and domain gap has been decreased by the customized 3 step unsupervised domain adaptation method. Special augmentations have been applied and ablation study has been performed to understand the effect of the parts in determined architecture. To increase the efficiency, we set another main objective for rover study to utilize multi-task learning method to check the feasibility of combination of semantic segmentation and multi-label image classification. As synthetic and AI4Mars datasets do not contain any classification label, additional real dataset (PanCam) have been used. We expected that some limitations of synthetic dataset will be compromised by using image classification dataset which will help to generalize better. 

The same approach for semantic segmentation of rover study have been applied to the drone study in which synthetic dataset is different as applications are slightly different. We collected real samples from flight logs of Ingenuity helicopter from NASA. We selected 50 images and labeled them semantically. After training with synthetic images as source and unlabelled real dataset as target images, efficiency of domain adaptation method have been proved by comparisons with baseline performances for both studies.



<h2> Dataset </h2>

Main objective of Rover study is to create and check efficiency of a multi-tasking deep learning model for navigation purposes and possible scientific exploration. Semantic Segmentation is the most suitable task to achieve the goal for navigation. That is why, AI4Mars is selected as segmentation dataset from real domain. AI4Mars has segmentation labels and high resolution post processed samples while it brings some problems with it. Labels are defined by crowdsourcing approach which decreases the quality of annotations. There are several standards in labelling of the AI4Mars and all annotations were done by non experts. Classes are Soil, Bed Rock, Big Rock and Sand. Soil and Bed Rock are not risky terrains as rover can easily select them as path. However other classes can create beaching problem or severely damage the rover. That is why, in order to check the efficiency we also selected synthetic dataset provided from AIKO. This dataset has great accuracy of annotations as labeller software is used with the same classes. Except from domain gap, there are some problems which creates difficulty for compliance with real environment:


![Alt text](images/ai4mars_example.png?raw=true "Example from AI4Mars")


• Rover parts does not exist in the synthetic set

• All terrain images taken horizontally while camera head moving in different angles in real domain

• Class Imbalance for the big rock as in AI4Mars

• Context shift for the classes Bed rock and soil between real and synthetic set. Standards of labellings are different.

In order to perform the scientific exploration task, we needed to use image classification with multi-labels. However no dataset exists in which labels for both tasks are available. That is why, additional dataset has been selected from PanCam after some research and comparison between datasets. Samples of this set are labelled for 25 classes in which 3 of them were underrepresented. Therefore, we eliminated those classes to keep the model robust. Results have been checked after first step of simplification. In the second step, we analyzed each class and combined or removed some of them to simplify the set. At the end we have got 12 classes in which most are different from the ones in semantic segmentation datasets. 


The same kind of synthetic dataset with accurate labelling is utilized for Drone study but from different perspective of camera. For the real images (target set) we collected 4500 images from the flight logs of the Ingenuity drone from official website. All images were distorted by fisheye lens and we performed post-processing procedure to make it compatible with normal images. We selected 50 images and annotated them for further performance check of the method. 

![Alt text](images/ingenuity_example.png?raw=true "Example from collected dataset for Ingenuity")


<h2> Model Architecture </h2>

For Semantic Segmentation task of the Drone, we selected transformer based architecture after research on state-of-art methods. Segformer has been found as most robust and best performing model. This architecture contains encoder of mixed transformer and CNN-based decoder. Only problem for this structure is its demand to high volume of training data as in Vision Transformers. Different studies were able to decrease the hunger for data while it is still higher than CNN architectures.


For the Multi-task learning of the Rover, same architecture as drone but with additional classification head attached to Mixed Transformer backbone has been selected. We optimized this task-specific head to decrease the overfitting and increase the accuracy of the multi-label classification. 

![Alt text](images/da_studies.png?raw=true "Model architectures for both tasks")


<h2> Domain Adaptation </h2>

Synhetic datasets are very easy to annotate with the cost of the domain gap. In order to decrease this gap, unsupervised domain adaptation methods are developed for autonomous driving applications. According to characteristics of our datasets, we selected self-training based unsupervised domain adaptation method to be most suitable after failed attempt with adversarial domain adaptation.

According to the research done on state-of-art domain adaptation methods, we customized the architecture suitable to our application which consists of 3 steps as shown in figure 1:

• Supervised training source dataset: Standard supervised training is carried out to train each decoder and give the model segmentation and classification capability. Source dataset contains only labelled samples from synthetic segmentation dataset and PanCam Classification dataset. Each part of the set trains appropriate part of the architecture. Backbone gains small generalization capability because of real part of the dataset which increases the segmentation result by 5 mIoU at baseline.

• Contrastive Learning is the second step of the customized method. Target samples composed of real unlabelled images from AI4Mars and PanCam are used. The same image is augmented in different variants and fed to the network. Output of the backbone is projected another space and it is checked if the features belong to the same image or not. By this way, we are able to generalize the backbone. 

• In order to avoid overfitting from segmentation head, we used mean-teacher self training method. Classification head is already trained with real dataset which allows us to focus on segmentation task only. Teacher model contains the same architecture as student model which is used in previous steps. The only difference is update method. Pseudo labels generated from teacher network are used in augmentation of the images to be fed to the student network. Considering the pseudo label as ground truth, we calculate the loss on prediction of the student and update the student model. Teacher is updated through Exponential Moving Average of student weights. By this way, confidence of the pseudo labels is kept high. 


![Alt text](images/sum.png?raw=true "Overall Domain Adaptation Architecture")



<h2> Results </h2>


In the results, we experience the problem of differences between datasets which are in addition to the domain gap. Big rock class has been affected by absence of rover parts in training images, horizontal shooting style affects half of the images to be misclassified. Additionally, training has been done with limited resources in which batch size should be much higher for efficiency of contrastive learning. In Rover study, customized domain adaptation architecture has been test on different configurations for semantic segmentation in table 1 and multi task learning in table 2.

![Alt text](images/table1.png?raw=true)

![Alt text](images/table2.png?raw=true)

Drone study has been tested with manually annotated real dataset of Ingenuity in order to check the performance of UDA. Table 3 shows the result for the drone study.

![Alt text](images/table3.png?raw=true)


As a result, images which are in the same camera style with synthetic dataset, perform decently while other images have serious problems. Main reason is that synthetic and real datasets have completely different characteristics. Mainly synthetic datasets are intended to decrease the disadvantages of real dataset in the cost of domain gap while in our application it is not suitable as seen from the results. It can be understood from the results of drone study that, customized UDA method worked suitable and can be improved by wide resources of GPU. Scores of classification side of the rover are decent to put effort on semantic segmentation task to increase the efficiency.
