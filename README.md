# Emotion-Recognition via CNN and Explaination AI (LIME algorithm)
The main idea of this work is to apply several AI  methods  on the CNN classifier using the emotion datasets. The LIME and LRP algorithms are the objectives, but other methods will also be studied.

### In this work, we followed two steps: 
 
1- We used a convolution neural network (CNN) for the emotions recognition 

2- Using the Lime algorithm to monitor the heat maps of the input image which help our CNN decide to select the corresponding emotion.


### Requirements

- Python:-------------------->             ```              3.3+ or Python 2.7                         ```
- OS:------------------------>             ```              Windows macOS or Linux                     ```
- Keras:--------------------->             ```              pip install Keras                          ```
- Tensorflow:--------------->             ```              pip install Tensorflow                     ```
- dlib:----------------------->             ```              pip install dlib                           ```
- face-recognition:---------->             ```              pip install face-recognition               ```
- fer2013 dataset:----------->             ```              https://www.kaggle.com/deadskull7/fer2013  ```
- LIME explaination AI:----->             ```              pip install lime                           ```

Fer2013 dataset
-
   The data consists of 48x48 pixel grayscale images of faces. The faces have been automatically registered so that the face is more or less centered and occupies about the same amount of space in each image. The task is to categorize each face based on the emotion shown in the facial expression in to one of seven categories (0=Angry, 1=Disgust, 2=Fear, 3=Happy, 4=Sad, 5=Surprise, 6=Neutral).

   train.csv contains two columns, "emotion" and "pixels". The "emotion" column contains a numeric code ranging from 0 to 6, inclusive, for the emotion that is present in the image. The "pixels" column contains a string surrounded in quotes for each image. The contents of this string a space-separated pixel values in row major order. test.csv contains only the "pixels" column and your task is to predict the emotion column.

   The training set consists of 28,709 examples. The public test set used for the leaderboard consists of 3,589 examples. The final test set, which was used to determine the winner of the competition, consists of another 3,589 examples.

   This dataset was prepared by Pierre-Luc Carrier and Aaron Courville, as part of an ongoing research project. They have graciously provided the workshop organizers with a preliminary version of their dataset to use for this contest.

### Note:
In our work we converted CSV files to the images.

## Main Code
Import the needed libraries
```

from __future__ import print_function
import keras
from keras.preprocessing.image import ImageDataGenerator
from keras.models import Sequential
from keras.layers import Dense, Dropout, Activation, Flatten, BatchNormalization
from keras.layers import Conv2D, MaxPooling2D
from keras.preprocessing.image import ImageDataGenerator
import os
from keras.models import Sequential
from keras.layers.normalization import BatchNormalization
from keras.layers.convolutional import Conv2D, MaxPooling2D
from keras.layers.advanced_activations import ELU
from keras.layers.core import Activation, Flatten, Dropout, Dense
from keras.optimizers import RMSprop, SGD, Adam
from keras.callbacks import ModelCheckpoint, EarlyStopping, ReduceLROnPlateau
from keras import regularizers
from keras.regularizers import l1
```
Initialing data
- 
```
num_classes = 7                                        # We have 7 Classes: Angry, Disgust, Fear, Happy, Natural, Sad and Surprise
img_rows, img_cols = 48,48                             # The size of input image
batch_size = 512                                       # The number of input pixels for augmentation
train_data_dir = '/src/fer2013/train'                  # Train data(contain 7 subfolder for each emotion)
validation_data_dir = '/src/fer2013/validation/'       # Test data(contain 7 subfolder for each emotion)
```
Use some data augmentation
-
- Use ImageDataGenerator to create fake samples to help our network learn better and avoid overfitting. We perform a rescale, rotation_range, shear_range, zoom_range, horizontal_flip to do training, validation and test data.
        
Creating the model
-
-   For the first step, we used a handcrafted architecture from CNN with these layers: 
```
model = Sequential()

model.add(Conv2D(32, kernel_size=(3, 3), activation='relu',kernel_regularizer=regularizers.l2(0.0001),input_shape=(48,48,1)))
model.add(Conv2D(64, kernel_size=(3, 3), activation='relu',kernel_regularizer=regularizers.l2(0.0001)))
model.add(MaxPooling2D(pool_size=(2, 2)))

model.add(Conv2D(128, kernel_size=(3, 3), activation='relu', kernel_regularizer=regularizers.l2(0.0001)))
model.add(MaxPooling2D(pool_size=(2, 2)))

model.add(Conv2D(128, kernel_size=(3, 3), activation='relu', kernel_regularizer=regularizers.l2(0.0001)))
model.add(MaxPooling2D(pool_size=(2, 2)))

model.add(Conv2D(6, kernel_size=(1, 1), activation='relu', kernel_regularizer=regularizers.l2(0.0001)))
model.add(Conv2D(7, kernel_size=(4, 4), activation='relu', kernel_regularizer=regularizers.l2(0.0001)))

model.add(Flatten())
model.add(Activation("softmax"))
model.summary()
```
About Layers:
- 
```
- Each conv2D layer extracts the features of the image according to the kernel filter. we used 3 * 3 kernels.
- And on the Maxpool diapers; they select the entities with the highest resolutions among the 2 * 2 dimension entities.
- Flatten Layer converts a tensor to a vector to send it to fully connected layers that use in the classification.
- The softmax activation is normally applied to the very last layer in a neural net, instead of using ReLU, sigmoid, tanh, or another activation function. The reason why softmax is useful is because it converts the output of the last layer in your neural network into what is essentially a probability distribution.
```
### Train the model

In this step, using our augmented data, we start to train our model. First we need to load a model file to save the training results (model weight):
```
filepath = os.path.join('/emotion_detector_models/model.hdf5')  
```
Then, we simply monitor the true values of the validation data during training and record the best values:
```
checkpoint = keras.callbacks.ModelCheckpoint(filepath,           
                                            monitor='val_acc',      
                                            verbose=1,
                                            save_best_only=True,
                                            mode='max')
callbacks = [checkpoint]

```
### Model compilation
- At first we need to compile your model. We use Adam's optimization and cross entropy to reduce the loss value of our model.
```
model.compile(loss='categorical_crossentropy',optimizer=Adam(lr=0.0001, decay=1e-6),metrics=['accuracy'])  
```
- Adam is an optimization algorithm that can be used instead of the classical stochastic gradient descent procedure to update network weights iterative based in training data.
```
nb_train_samples = 31205           #Number of train samples
nb_validation_samples = 6085       #Number of test sample
epochs = 50                        #Number of train and test loob
```
### Train Step

The main line to train our model. We train our model to augmented training and validation data.
```
model_info = model.fit_generator(                  
                                train_generator,
                                steps_per_epoch=nb_train_samples // batch_size,
                                epochs=epochs,
                                callbacks = callbacks,
                                validation_data=validation_generator,
                                validation_steps=nb_validation_samples // batch_size)

model.save_weights('/emotion_detector_models/model.hdf5')
```
Plot model loss in train step
-
```
print(model_info.history.keys())
from matplotlib import pyplot as plt
plt.plot(model_info.history['loss'])       #Plot the loss value of training step
plt.plot(model_info.history['val_loss'])   #plot the Validation loss 
plt.title('model loss in train step')     
plt.ylabel('loss')
plt.xlabel('epoch') 
plt.legend(['train', 'test'], loc='upper left')
plt.show()
```
- The output of training step:

![](https://github.com/Mahdidrm/Emotion-Recognition/blob/master/Figures/Figure_1.png?raw=true)

Testing the model with validation data
-
Here we test our trained model using our validation data
```
model_info = model.fit_generator(
            train_generator,
            steps_per_epoch=nb_train_samples // batch_size,
            epochs=epochs,
            callbacks = callbacks,
            validation_data=validation_generator,
            validation_steps=nb_validation_samples // batch_size)
```
Plot model loss in train step
-
And then we show the train loss of model
```
from matplotlib import pyplot as plt
plt.plot(model_info.history['loss'])
plt.plot(model_info.history['val_loss'])
plt.title('Model loss in Test step')
plt.ylabel('loss')
plt.xlabel('epoch')
plt.legend(['train', 'test'], loc='upper left')
plt.show()
```    
- The output of validation step: 

![](https://github.com/Mahdidrm/Emotion-Recognition/blob/master/Figures/test.png?raw=true)

Confusion Matrix of our model in some validation images
-
- First, we need to import some libraries
```
import matplotlib.pyplot as plt
import sklearn
import PIL
from sklearn.metrics import classification_report, confusion_matrix
import numpy as np
```
- And now we give the number of training and validation images
```
nb_train_samples =  4421          
nb_validation_samples =4096     
```
- Some data augmentation 
```
validation_datagen = ImageDataGenerator(featurewise_center=True,
                                        featurewise_std_normalization=True)
```
- We need to recreate our validation generator with shuffle = false
```
validation_generator = validation_datagen.flow_from_directory(
        validation_data_dir,
        color_mode = 'grayscale',
        target_size=(img_rows, img_cols),
        batch_size=batch_size,
        class_mode='categorical',
        shuffle=False)
```
- Find the labels of Classes
```
class_labels = validation_generator.class_indices
class_labels = {v: k for k, v in class_labels.items()}
classes = list(class_labels.values())
```

- Confution Matrix and Classification Report
```
Y_pred = model.predict_generator(validation_generator, nb_validation_samples // batch_size+1)
y_pred = np.argmax(Y_pred, axis=1)

print('Confusion Matrix')
print(confusion_matrix(validation_generator.classes, y_pred))
print('Classification Report')
target_names = list(class_labels.values())
print(classification_report(validation_generator.classes, y_pred, target_names=target_names))

plt.figure(figsize=(8,8))
cnf_matrix = confusion_matrix(validation_generator.classes, y_pred)

plt.imshow(cnf_matrix, interpolation='nearest')
plt.colorbar()
tick_marks = np.arange(len(classes))
_ = plt.xticks(tick_marks, classes, rotation=90)
_ = plt.yticks(tick_marks, classes)
```
- In Number schema: 

![](https://github.com/Mahdidrm/Emotion-Recognition/blob/master/Figures/confision.jpg?raw=true) 
- And in Graohic schema:

![](https://github.com/Mahdidrm/Emotion-Recognition/blob/master/Figures/confission.png?raw=true)


- And the classification report:

![](https://github.com/Mahdidrm/Emotion-Recognition/blob/master/Figures/report.jpg?raw=true)

Test on some of validation images
- 
- In this step, we test our model in some of the validation images in our dataset.
- First, we load some liberaries:
```
from keras.models import load_model
from keras.optimizers import RMSprop, SGD, Adam
from keras.preprocessing import image
import numpy as np
import os
import cv2
import numpy as np
from os import listdir
from os.path import isfile, join
import re
import keras
from keras.applications import inception_v3 as inc_net
from keras.preprocessing import image
from keras.applications.imagenet_utils import decode_predictions
from skimage.io import imread
import matplotlib.pyplot as plt
import numpy as np
```
- This function writes true and predicted class matches from images.
```
def draw_test(name, pred, im, true_label):
    BLACK = [0,0,0]
    expanded_image = cv2.copyMakeBorder(im, 160, 0, 0, 300 ,cv2.BORDER_CONSTANT,value=BLACK)
    cv2.putText(expanded_image, "predited - "+ pred, (20, 60) , cv2.FONT_HERSHEY_SIMPLEX,1, (0,0,255), 2)
    cv2.putText(expanded_image, "true - "+ true_label, (20, 120) , cv2.FONT_HERSHEY_SIMPLEX,1, (0,255,0), 2)
    cv2.imshow(name, expanded_image)
```
This function chooses some random images from our validation folder.
```
def getRandomImage(path, img_width, img_height):
    """function loads a random images from a random folder in our test path """
    folders = list(filter(lambda x: os.path.isdir(os.path.join(path, x)), os.listdir(path)))
    random_directory = np.random.randint(0,len(folders))
    path_class = folders[random_directory]
    file_path = path + path_class
    file_names = [f for f in listdir(file_path) if isfile(join(file_path, f))]
    random_file_index = np.random.randint(0,len(file_names))
    image_name = file_names[random_file_index]
    final_path = file_path + "/" + image_name
    return image.load_img(final_path, target_size = (img_width, img_height),grayscale=True), final_path, path_class
```
- Dimensions of our images
```
img_width, img_height = 48, 48
```
- We use a very small learning rate 
```
model.compile(loss = 'categorical_crossentropy',
              optimizer = RMSprop(lr = 0.001),
              metrics = ['accuracy'])
```
- And now we give the address of the training folder and call the functions
```
files = []
predictions = []
true_labels = []

# predicting images
for i in range(0, 10):
    path = '/src/fer2013/validation/' 
    img, final_path, true_label = getRandomImage(path, img_width, img_height)
    files.append(final_path)
    true_labels.append(true_label)
    x = image.img_to_array(img)
    x = x * 1./255
    x = np.expand_dims(x, axis=0)
    images = np.vstack([x])
    classes = model.predict_classes(images, batch_size = 10)
    predictions.append(classes)
    
for i in range(0, len(files)):
    image = cv2.imread((files[i]))
    image = cv2.resize(image, None, fx=3, fy=3, interpolation = cv2.INTER_CUBIC)
    draw_test("Prediction", class_labels[predictions[i][0]], image, true_labels[i])
    cv2.waitKey(0)

cv2.destroyAllWindows()
 ```           
Test on a single image out of our dataset (RGB or GrayScale)
-
In this step, we check the capacity of our model trained on an image of our dataset
- Fist, we load the needed librairies
``` 
import keras
from keras.applications.imagenet_utils import decode_predictions
from keras.models import load_model
from keras.preprocessing import image
from keras.preprocessing.image import img_to_array
import numpy as np
import os
from os import listdir
from os.path import isfile, join
import matplotlib.pyplot as plt
import lime
from lime import lime_image
import cv2
from skimage.io import imread
import matplotlib.pyplot as plt
``` 
And now we should load a face classifier to find the face on the input image. We use of haarcascade_frontalface face classifier.
- Haar Cascade is a machine learning object detection algorithm used to identify objects in an image or video and based on the concept of features proposed by Paul Viola and Michael Jones in their paper "Rapid Object Detection using a Boosted Cascade of Simple Features" in 2001.
- More Info: https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_objdetect/py_face_detection/py_face_detection.html

``` 
face_classifier = cv2.CascadeClassifier('/Haarcascades/haarcascade_frontalface_default.xml') 
```
- Face detection function
```
def face_detector(img):
    # Convert image to grayscale
    gray = cv2.cvtColor(img.copy(),cv2.COLOR_BGR2GRAY)
    faces = face_classifier.detectMultiScale(gray, 1.3, 5)
    if faces is ():
        return (0,0,0,0), np.zeros((48,48), np.uint8), img
    
    allfaces = []   
    rects = []
    for (x,y,w,h) in faces:
        cv2.rectangle(img,(x,y),(x+w,y+h),(255,0,0),2)
        roi_gray = gray[y:y+h, x:x+w]
        roi_gray = cv2.resize(roi_gray, (48, 48), interpolation = cv2.INTER_AREA)
        allfaces.append(roi_gray)
        rects.append((x,w,y,h))
    return rects, allfaces, img
```
- Load an image out of our dataset (GrayScale or RGB) and send to face detection function
```
img20 = cv2.imread("/test_images/22.png")
rects, faces, image = face_detector(img20)
i = 0
for face in faces:
    roi = face.astype("float") / 255.0
    roi = img_to_array(roi)
    roi = np.expand_dims(roi, axis=0)
```
- Make a prediction on the ROI, then lookup the class
```
    preds = classifier.predict(roi)[0]
    label = class_labels[preds.argmax()]   
```
- Overlay our detected emotion on our pic
```
    label_position = (rects[i][0] + int((rects[i][1]/2)), abs(rects[i][2] - 10))
    i =+ 1
    cv2.putText(image, label, label_position , cv2.FONT_HERSHEY_SIMPLEX,1, (0,255,0), 2)
    cv2.imshow("Emotion Detector", image)
cv2.waitKey(0)
cv2.destroyAllWindows()

   ```           
Explainable AI  
-
Explainable AI (XAI) refers to methods and techniques in the application of artificial intelligence technology (AI) such that the results of the solution can be understood by humans. It contrasts with the concept of the "black box" in machine learning where even their designers cannot explain why the AI arrived at a specific decision.[1] XAI may be an implementation of the social right to explanation.[2] XAI is relevant even if there is no legal rights or regulatory requirements (for example, XAI can improve the user experience of a product or service by helping end users trust that the AI is making good decisions.)

The technical challenge of explaining AI decisions is sometimes known as the interpretability problem.[3] Another consideration is info-besity (overload of information), thus, full transparency may not be always possible or even required. However, simplification at the cost of misleading users in order to increase trust or to hide undesirable attributes of the system should be avoided by allowing a tradeoff between interpretability and completeness of an explanation. [4]

AI systems optimize behavior to satisfy a mathematically-specified goal system chosen by the system designers, such as the command "maximize accuracy of assessing how positive film reviews are in the test dataset". The AI may learn useful general rules from the test-set, such as "reviews containing the word 'horrible'" are likely to be negative". However, it may also learn inappropriate rules, such as "reviews containing 'Daniel Day-Lewis' are usually positive"; such rules may be undesirable if they are deemed likely to fail to generalize outside the test set, or if people consider the rule to be "cheating" or "unfair". A human can audit rules in an XAI to get an idea how likely the system is to generalize to future real-world data outside the test-set.[3]


Methods
-
- LIME

LIME is a novel explanation technique that explains the predictions of any classifier in an interpretable and faithful manner, by learning an interpretable model locally around the prediction. [5]
```
import lime
from lime import lime_image
from lime.wrappers.scikit_image import SegmentationAlgorithm

explainer = lime_image.LimeImageExplainer(verbose = False)
segmenter = SegmentationAlgorithm('slic', n_segments=100, compactness=1, sigma=1)
explanation = explainer.explain_instance(roi[0], classifier.predict_proba, top_labels=6, hide_color=0, num_samples=10000, segmentation_fn=segmenter)

```
- And We put a Mask on image to show the HeatMaps
```
from skimage.color import label2rgb
temp, mask = explanation.get_image_and_mask(explanation.top_labels[1], positive_only=True, num_features=5, hide_rest=False)
plt.imshow(mark_boundaries(temp / 2 + 0.5, mask))

 ```

Refrences
-
[1] Sample, Ian (5 November 2017). "Computer says no: why making AIs fair, accountable and transparent is crucial". the Guardian. Retrieved 30 January 2018. https://www.theguardian.com/science/2017/nov/05/computer-says-no-why-making-ais-fair-accountable-and-transparent-is-crucial
 
[2] Edwards, Lilian; Veale, Michael (2017). "Slave to the Algorithm? Why a 'Right to an Explanation' Is Probably Not the Remedy You Are Looking For". Duke Law and Technology Review. 16: 18. SSRN 2972855. https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2972855

[3] "How AI detectives are cracking open the black box of deep learning". Science. 5 July 2017. Retrieved 30 January 2018. https://www.sciencemag.org/news/2017/07/how-ai-detectives-are-cracking-open-black-box-deep-learning
 
[4] Gilpin, Leilani H.; Bau, David; Yuan, Ben Z.; Bajwa, Ayesha; Specter, Michael; Kagal, Lalana (2018-05-31). "Explaining Explanations: An Overview of Interpretability of Machine Learning". arXiv:1806.00069 [stat.AI]. https://arxiv.org/abs/1806.00069

[5] Paper: https://arxiv.org/abs/1602.04938  Code: https://github.com/marcotcr/lime 
