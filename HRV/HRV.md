## Using heart rate variability to identify stress and sleep quality
<br>
<a href="https://github.com/jpizzollo/MMASH"><img src="https://img.shields.io/badge/GitHub-View_on_GitHub-blue?logo=GitHub" alt="View on GitHub"></a>
<img src="HRV/pexels-photo-4379289.jpeg?raw=true"/>

Photo by <a href="https://www.pexels.com/@karolina-grabowska">Karolina Grabowska</a> on <a href="https://www.pexels.com">Pexels</a>


### Introduction

If you were to measure your heart rate as 60 beats per minute, you might assume that your heart is beating once every second. While 1 second between beats might be the average, the time intervals vary slightly above and below 1 second. This phenomenon is called Heart rate variability (HRV) and is a measure of slight variations in heart rate. Having a high HRV is a good thing and is associated with rest and recovery, whereas a low HRV, or more consistency between heart beats, is associated with stress and fatigue.

<a href="HRV/img_HRV.png" target="_blank"><img src="HRV/img_HRV.png"></a>

Recent popular use of HRV has been for monitoring health or athletic training, and involves wearing some type of heart rate measuring device. Wearable technology like smart watches, wristbands, or even an electronic ring, allow us to follow changes in HRV continuously. With these devices, we can see how HRV changes throughout the day and even use it for tracking sleep.

Since HRV changes with periods of stress and rest, I wanted to see if I could distinguish between stressful and restful activities, and identify disturbances during sleep, in people wearing a continuous HR monitor. I used data from the <a href="https://physionet.org/content/mmash/1.0.0/">Multilevel Monitoring of Activity and Sleep in Healthy People (MMASH)</a> study [1,2], which used continuous monitoring of HRV, heart rate, and actigraphy (movement) in 22 healthy volunteers. In my analysis I used only HRV and actigraphy data (minimal data to replicate real-world data collection) to predict stress vs rest, and used HRV and heart rate data to predict disturbances in sleep. Even this small data set, the model shows we can predict stress and identify changes in sleep with good accuracy. Therefore, monitoring that can be done in a minimally invasive way, even just by wearing a wristband or smartwatch, can help track key metrics to inform about our daily stress and rest.

### Methods

#### Data Availability and Description

Data from the MMASH study were obtained from the PhysioNet [3] database. For each of 22 study participants, actigraphy, vector magnitude, HR, RR-interval (time between heart beats), and logs of activity throughout the day were collected.

#### Data filtering

HR measuring devices sometimes produce erroneous results, showing very high or very low HR for short periods of time (a few seconds). To avoid these high or low values confounding results, data were filtered to remove periods of HR greater or less than 2 standard deviations from the mean. Note: even during periods of intense exercise HR did not fall outside of 2 SD. Similarly, RR data were filtered to remove intervals greater than 3 seconds or less than 0.3 seconds, corresponding to HR < 20 bpm and > 200 bpm, respectively.

#### Determining HRV

To measure HRV, RR intervals are typically recorded for several minutes and variability throughout the time period is measured. To determine HRV within this dataset, I divided 24-hour periods into 5-minute periods. Some data points were missing and periods were excluded if there were less than 3 minutes of RR data available. HRV was characterized by determining the root mean square of successive differences (RMSSD) in RR intervals, a standard metric for quantifying HRV.

#### Stress versus rest classification

Time periods were annotated with participant activity throughout the study. Times when participants were laying down or sitting were labelled as restful periods, whereas heavy exercise, alcohol use, and smoking were labelled as stressful activities. To build cases on which to train a model to distinguish between stressful and restful activities, time periods were divided into 3 to 5-minute time intervals. Within these intervals were calculated, mean HR, RMSSD, mean accelerometry (Nm) along X, Y, and Z axis, mean vector magnitude derived from raw accelerometry (Nm), mean number of steps taken, and inclinometry distribution among standing, sitting, or lying down. Rest and stress cases were somewhat imbalanced at approximately a 4:1 ratio, so the stress cases were up-sampled with SMOTE. Of 2732 cases after up-sampling, 80% were used for training a random forest classifier, 10% for model validation, and 10% for final testing.

#### Identifying sleep disturbances

Sleep disturbances were identified in the dataset as periods of movement, measured by actigraphy, during times labelled as sleep. Since most of these disturbances were very short, many just several seconds, HRV was quantified in 1-minute intervals during sleep to distinguish between periods with and without sleep disturbances. Intervals were labelled as having a disturbance if there was any recorded movement during the interval. A total of 6 intervals in the preceding 10 minutes were used to predict sleep disturbances by looking at average HR and 1-minute RMSSD values over the course of 10 minutes. Because the positive class represented only about 11% of total cases, the positive class was up-sampled with SMOTE. Of 8630 cases after up-sampling, 80% were used for training a random forest classifier, 10% for model validation, and 10% for final testing.

### Results

#### HRV and actigraphy distinguish stressful and restful activities

HRV is highly variable throughout a 24 hour period and responds not only to stress or rest, but also to changes in hear rate, or changes in physical activity. For an individual user, we can see HRV RMSSD values fluctuate from roughly 10 – 100 throughout the day. The periods of highest HRV are during sleep at night and while laying down during the day. Conversely, some of the periods of lowest HRV are during stressful activities like exercise.

<a href="HRV/img_stress_rest.png" target="_blank"><img src="HRV/img_stress_rest.png"></a>

Using a random forest classifier trained on 3 -5 minute time intervals across all users, I was able to distinguish stressful and restful activities using metrics for HRV, HR, and actigraphy. Overall, accuracy for this model is good at 87% considering this is a noisy dataset – there tends to be a lot of fluctuation in HRV and activity throughout the day and within activities. There is some imbalance in recall, where for restful activity recall is 82% and for stressful activity it is 91%. This could be balanced by increasing the probability threshold for predicting stress, but that would also make precision of rest prediction slightly worse. For the purposes of this project, I chose to balance rest precision and stress recall to best capture true negatives and true positives from the data.

<a href="HRV/img_stress_predictions.png" target="_blank"><img src="HRV/img_stress_predictions.png"></a>

#### HRV and HR identify sleep disturbances

As a way to classify sleep quality during time intervals throughout the night, I looked at actigraphy data that identify movement from a wrist-worn accelerometer. While movement during sleep isn’t entirely sufficient to fully characterize quality of sleep, it is a minimally invasive way of identifying disturbances in sleep that could indicate overall quality. Consider the HRV and actigraphy data for two different users shown below. Which person had a better night’s sleep? It’s clear that User 1 had dozens of recorded movements throughout the night, whereas User 2 had little movement, therefore I propose that sleep disturbances provide a good proxy for sleep quality.

<a href="HRV/img_sleep_patterns.png" target="_blank"><img src="HRV/img_sleep_patterns.png"></a>

Many changes in HRV coincide with movement during sleep. From this dataset, I divided each user’s sleep into 1-minute intervals and labelled an interval as having a sleep disturbance if there was any movement at all during the interval. Using these time intervals and the 6 time points prior to each 1-minute interval, I trained a random forest classifier to identify sleep disturbances. Accuracy of the model is good at 91%, and precision and recall are well balanced for both positive and negative class predictions. Similar to the stress predictions, these data are noisy – there are a lot of time intervals that have changes in HRV with no movement, and there is a lot of variability in how HRV and sleep disturbances correlate between users. Nonetheless, even in a minimally invasive way (using a device that measures HRV and HR), there is good accuracy in identifying when sleep disturbances occur during the night.

<a href="HRV/img_sleep_predictions.png" target="_blank"><img src="HRV/img_sleep_predictions.png"></a>

### Conclusion

The ability to capture data that can inform about health and wellness is beginning to see a significant improvement with wearable devices such as wrist-bands and smart-watches. These devices can leverage machine learning models to provide users with valuable information about stress and sleep by capturing data with minimally invasive devices. In this project, I show that across 22 individuals, we can predict stress during the day and quality of sleep with ~90% accuracy using a model trained on data from all users.

As a baseline, the models here perform well with only a small set of noisy and variable data. In a real-world scenario, models could be trained on data from an individual user to decrease user-to-user variability, and could benefit from having larger datasets from users wearing devices and labelling activities across a larger training phase. This project highlights the potential of using HR and HRV as ways identify factors that contribute to health and wellness and do so in a way that is accessible and minimally invasive.

### References

1. Rossi, A., Da Pozzo, E., Menicagli, D., Tremolanti, C., Priami, C., Sirbu, A., Clifton, D., Martini, C., & Morelli, D. (2020). Multilevel Monitoring of Activity and Sleep in Healthy People (version 1.0.0). PhysioNet. https://doi.org/10.13026/cerq-fc86.

2. Rossi, A., Da Pozzo, E., Menicagli, D., Tremolanti, C., Priami, C., Sirbu, A., Clifton, D., Martini, C., & Morelli, D. (2020). A Public Dataset of 24-h Multi-Levels Psycho-Physiological Responses in Young Healthy Adults. Data, 5(4), 91. https://doi.org/10.3390/data5040091.

3. Goldberger, A., Amaral, L., Glass, L., Hausdorff, J., Ivanov, P. C., Mark, R., ... & Stanley, H. E. (2000). PhysioBank, PhysioToolkit, and PhysioNet: Components of a new research resource for complex physiologic signals. Circulation [Online]. 101 (23), pp. e215–e220.
