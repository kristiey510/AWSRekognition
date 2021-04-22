# Using AWS Rekognition to detect labels  
This repository provides original data(.jpeg images) that uses AWS Rekognition to detect the object using labels. Users can use this on their own data and add this to software that requires recognizing objects. Our goal is to provide image detection software that can aid visually impaired individuals. We explore the effects of image quality (brightness/sharpness/contrast) on select images to find if AWS Rekognition is equally effective in recognizing objects at different times of the day and regardless of image quality.

## Adding data to S3 buckets to use Amazon Rekognition 
Users must add their data into a newly created or existing S3 bucket in the form of .jpg or .png. 

Navigate to the S3 bucket in the services tab to upload data in the form of images: 

<img width="500" alt="Screen Shot 2021-04-21 at 4 08 40 PM" src="https://user-images.githubusercontent.com/69918614/115615599-2cdafd80-a2bd-11eb-832c-ba36fcef6e93.png">


Either create a new bucket or use an existing bucket. Upload data using the upload option: 
<img width="793" alt="Screen Shot 2021-04-21 at 4 13 49 PM" src="https://user-images.githubusercontent.com/69918614/115615296-c81fa300-a2bc-11eb-87e3-29561222650d.png">

Select upload and you should see the file in your S3 bucket. 

## Preparing SageMaker for AWS Rekognition
In order to use AWS Rekognition using SageMaker in a notebook instance, you must add permissions to the IAM user role. Click on the name of an existing notebook or create a new notebook instance. 

<img width="862" alt="Screen Shot 2021-04-21 at 4 24 01 PM" src="https://user-images.githubusercontent.com/69918614/115616645-6e1fdd00-a2be-11eb-90ec-e37fcccd9adf.png">

Navigate to the IAM User Role and select it: 
<img width="724" alt="Screen Shot 2021-04-21 at 4 24 10 PM" src="https://user-images.githubusercontent.com/69918614/115616819-a4f5f300-a2be-11eb-875b-84d0f76e26a2.png">

Under the permissions, verify that the policies "AmazonS3FullAccess" and "AmazonRekognitionFullAccess" are attached. If not, select attach policy and attach these policies. We need "AmazonS3FullAccess" to access the images from the S3 buckets and "AmazonRekognitionFullAccess" to use AWS Rekognition. 

<img width="875" alt="Screen Shot 2021-04-21 at 4 24 33 PM" src="https://user-images.githubusercontent.com/69918614/115617419-5e54c880-a2bf-11eb-8248-9eafa1deb9ad.png">

## Using Amazon Rekognition via Python SDK
Make sure the image that needs to be detected by Rekognition is in a S3 bucket: 
<img width="1101" alt="Screen Shot 2021-04-22 at 1 30 46 AM" src="https://user-images.githubusercontent.com/69918614/115661232-1e1b3780-a30b-11eb-8463-6bb05fec97ea.png">

In Amazon SageMaker, ensure that the policies listed above are active. Use an existing notebook instance or create a new notebook instance. 

We will use the AWS SDK for Python Boto3. Using boto3, we can retrieve objects from the specified bucket and use "detect_labels" feature in AWS rekognition to get a dictionary response of labels: 
<img width="899" alt="Screen Shot 2021-04-22 at 1 43 39 AM" src="https://user-images.githubusercontent.com/69918614/115661897-2cb61e80-a30c-11eb-830e-87b9dddb503e.png">

<img width="830" alt="Screen Shot 2021-04-22 at 1 44 20 AM" src="https://user-images.githubusercontent.com/69918614/115662033-60914400-a30c-11eb-8657-2ab9ed0b0f18.png">

We can select labels from the dictionary response to form a new dictionary or dataframe: 

<img width="968" alt="Screen Shot 2021-04-22 at 1 46 03 AM" src="https://user-images.githubusercontent.com/69918614/115662195-9f26fe80-a30c-11eb-803d-4e3d30bfeb04.png">


## Transforming data 
To adjust the brightness/sharpness/contrast levels for images, we create two directories in the SageMaker notebook instance, one containing original data named "orig-imgs" and one containing transformed data named "transformed-imgs".

**Note:** We use scale from factor of 0.5 to 1 (intervals of 0.1) for brightness/contrast and 0 to 2 (intervals of 0.2) for sharpness. We use PIL (Python Imaging Library) to import Image and ImageEnhance modules that allow us to change features of the image. 

<img width="478" alt="Screen Shot 2021-04-21 at 4 40 08 PM" src="https://user-images.githubusercontent.com/69918614/115618114-3d40a780-a2c0-11eb-817b-d280373fdfa2.png">

We get: 

<img width="209" alt="Screen Shot 2021-04-21 at 4 47 38 PM" src="https://user-images.githubusercontent.com/69918614/115619256-a543bd80-a2c1-11eb-8de2-ef40b0bb43de.png">

Navigate to the "orig-imgs" directory and add images to that directory. Make sure the original images are in the form "imgname-orig.jpg" for this code to work. Run the code below (found in transform-data.ipynb), which will produce photos with different brightness/sharpness/contrast levels in the "transformed-imgs" directory. 

<img width="873" alt="Screen Shot 2021-04-21 at 4 48 53 PM" src="https://user-images.githubusercontent.com/69918614/115619306-b5f43380-a2c1-11eb-9df6-dd0fdc2e6381.png">
<img width="847" alt="Screen Shot 2021-04-21 at 4 49 00 PM" src="https://user-images.githubusercontent.com/69918614/115619313-b8ef2400-a2c1-11eb-8a45-7ffea06b1be3.png">
<img width="624" alt="Screen Shot 2021-04-21 at 4 49 04 PM" src="https://user-images.githubusercontent.com/69918614/115619428-d8864c80-a2c1-11eb-8800-04cacb522799.png">

You will see the images in the transformed folder labeled with "imagename-featurechanged:level.jpg". We must import these folders into a S3 bucket to use AWS Rekognition on these photos.

## Moving transformed data into S3 Bucket
To move the transformed images and original images into an S3 bucket, we must upload them. To do this, use the code 

<img width="695" alt="Screen Shot 2021-04-21 at 5 03 58 PM" src="https://user-images.githubusercontent.com/69918614/115620779-9c53eb80-a2c3-11eb-86d0-2a5992781dbb.png">

We have created compressed files that we can download from SageMaker and upload to S3 bucket. 

## Creating CSV files from the Transformed Data
Using the transformed data located in the S3 bucket named "transformed-imgs-parmigiana", we can organize the data for data analysis by creating data frames to test on the data. We do this by getting the objects of all the images in the S3 bucket and creating dictionaries for our selected categories and images. 

<img width="636" alt="Screen Shot 2021-04-22 at 12 32 45 PM" src="https://user-images.githubusercontent.com/69918614/115751284-0a9dba00-a367-11eb-83a3-7e82252399f6.png">

Our dictionary will have the keys "Image", Category(Brightness, Sharpness, Contrast), "Label", and Confidence Level:

<img width="1086" alt="Screen Shot 2021-04-22 at 12 23 34 PM" src="https://user-images.githubusercontent.com/69918614/115750302-06bd6800-a366-11eb-9bd8-47756a687de8.png">

We create data frames and export as csv files for two types of images: "crosswalk" that identifies "Road" label and "food" that identifies "Food" label for each metric(Brightness/Sharpness/Contrast).

<img width="929" alt="Screen Shot 2021-04-22 at 12 23 41 PM" src="https://user-images.githubusercontent.com/69918614/115750318-09b85880-a366-11eb-847c-e767ced01789.png">

<img width="904" alt="Screen Shot 2021-04-22 at 12 23 45 PM" src="https://user-images.githubusercontent.com/69918614/115750279-ff965a00-a365-11eb-9975-33c068067b69.png">

## Hypothesis Testing on Transformed Data
We test for the significance in changing the metrics by conducting two-sample t-tests on the each category(food, crosswalk) using target labels (Food, Road) for each metric(Brightness/Sharpness/Contrast). 
First import the created csv files using pandas: 
<img width="615" alt="Screen Shot 2021-04-22 at 1 01 11 PM" src="https://user-images.githubusercontent.com/69918614/115754991-d2987600-a36a-11eb-9a24-0f660e5b4f58.png">

We use a t-test for each metric using our "crosswalk" image with label "Road" and conclude: 
<img width="896" alt="1792x590 png" src="https://user-images.githubusercontent.com/69918614/115755796-b943f980-a36b-11eb-9c51-707edd0cd455.png">

<img width="978" alt="1956x592 png" src="https://user-images.githubusercontent.com/69918614/115755833-c4972500-a36b-11eb-9015-ae03dc4a9e35.png">

<img width="869" alt="1738x578 png" src="https://user-images.githubusercontent.com/69918614/115755868-cbbe3300-a36b-11eb-89ad-d40fdcdce094.png">

## Architecture Diagram 

<img width="690" alt="Screen Shot 2021-04-21 at 5 03 58 PM" src="https://github.com/kristiey510/AWSRekognition/blob/main/Architectural%20Diagram.jpg">

We initially collected 10 different pictures for each of three categories (i.e. transportation, food, crosswalk). Afterwards, we transformed each picture into 20 pictures, with 10 of them having different levels of contrast and 10 different brightness. Utilizing Amazon Simple Storage Service (S3), we uploaded all the orginal and processed pictures into the  S3 bucket for our project. Then, we used the Amazon Rekognition Service to label the pictures and got a confidence score for each one. At last, we conducted hypothesis testing on the average of confidence scores and made conclusion based on p-value of the test.















