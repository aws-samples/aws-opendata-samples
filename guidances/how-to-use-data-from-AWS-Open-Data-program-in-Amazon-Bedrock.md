![image](https://github.com/user-attachments/assets/99afffa0-f552-48f7-9316-5886225ca2da)# How to use data from AWS Open Data program in Amazon Bedrock

The [AWS Open Data Sponsorship Program](https://aws.amazon.com/opendata/open-data-sponsorship-program/) eliminates data acquisition barriers by hosting high-value datasets in the cloud, enabling researchers and analysts to focus on discovery and innovation rather than data management. When data is shared on [Amazon Web Services (AWS)](https://aws.amazon.com/), anyone can analyze it and build services on top of it using a broad range of compute and data analytics products, including [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/), [Amazon Athena](https://aws.amazon.com/athena/), [AWS Lambda](https://aws.amazon.com/lambda/), and [Amazon EMR](https://aws.amazon.com/emr/). AWS provides a catalog of publicly available datasets on AWS through the [Registry of Open Data on AWS](https://registry.opendata.aws/). The registry has over 650 datasets open to the public, such as government data, scientific research, life sciences, climate, satellite imagery, geospatial, and genomic data.

Many government agencies, like the National Oceanic and Atmospheric Administration (NOAA), participate in the AWS Open Data Sponsorship Program. NOAA makes data available to the public through the [NOAA Open Data Dissemination (NODD) program](https://www.noaa.gov/information-technology/open-data-dissemination). You can [view all of the NODD datasets on AWS](https://registry.opendata.aws/collab/noaa/). We use NOAA data from the NODD program to demonstrate how customers can use data made available through AWS Open Data in Amazon Bedrock.

In this post, we discuss how to use NOAA datasets in the Registry of Open Data on AWS using [Amazon Bedrock Knowledge Bases](https://aws.amazon.com/bedrock/knowledge-bases/). With Amazon Bedrock Knowledge Bases, you can give [foundation models (FMs)](https://aws.amazon.com/what-is/foundation-models/) and agents contextual information from private and public data sources to deliver more relevant, accurate, and customized responses. With the NOAA Global Historical Climatology Network (GHCN) in particular, you can make information like precipitation and snow depth available to a set of users that might not be comfortable with SQL commands or other tools commonly used to search these types of data. Now nontechnical decision-makers can have access to highly technical data in an accessible and understandable format through a chat-based assistant.

## Solution Overview
This post is divided into two parts. You can try the feature without incurring significant cost in part one, which only uses a few files. Part two uses an entire dataset and gives a fuller picture of the feature, but it also incurs a higher cost. It’s recommended to delete the knowledge base and vector store when you’re done to stop incurring additional costs.

We use the [NOAA Global Historical Climatology Network (GHCN)](https://registry.opendata.aws/noaa-ghcn/) dataset as our knowledge base. This dataset contains temperature and snow depth readings from stations across the United States. The dataset is formatted as a set of .csv and text files available in a public [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/) bucket in the Registry of Open Data on AWS.

## Prerequisites
To perform the solution, you need to have the following prerequisites:
1.	Access to an AWS account with permissions to the relevant services
2.	Familiarity with the [AWS Management Console](https://aws.amazon.com/console/)

## Part 1: Solution walkthrough
Start with the Registry of Open Data on AWS:

1.	On the [Registry of Open Data on AWS](https://registry.opendata.aws/) website, in the Search datasets bar, enter GHCN, as shown in the following screenshot.
![image](https://github.com/user-attachments/assets/fc1b18a8-e469-4291-95b0-ae6d8e93363b)
*Figure 1: Registry of Open Data on AWS with GHCN in the search datasets box*

2.  Select the [NOAA Global Historical Climatology Network Daily (GHCN-D)](https://registry.opendata.aws/noaa-ghcn/) dataset to open the registry page.
3.  Right click on the Browse Bucket link to open in a new window. This link will open a list of all the files and folders that are currently in the NOAA GHCN public bucket.
![image](https://github.com/user-attachments/assets/5bf1eac6-8a6c-481b-a13a-9bd56a46a86c)
*Figure 2: AWS S3 Explorer view of the noaa-ghcn-pds bucket*

4.	Open the csv/ then by_year/ folders. Notice that GHCN data goes back to the year 1750! We start with a few of the oldest files to see what data we have available. Select the 1763.csv and 1764.csv files to download them.
5.	Scroll to the bottom of the page and choose Next. Keep choosing Next until you reach the last page of data. As of this writing, page 6 will have the years 2012 through 2025. Select 2023.csv and 2024.csv to download them also.
6.	Starting with the 1763.csv and 1764.csv files, open each one up in an application so you can view the data. 

### File and data format
knowledge base. You need to understand what is in the file and how it’s referenced, so that you can ask questions of the knowledge base and get appropriate answers. 

For example, the 1763.csv GHCN data has the following columns: ID, DATE, ELEMENT, and DATA_VALUE. These first four columns are the focus of the rest of this post. NOAA has provided documentation on each column at the [open-data-docs](https://github.com/awslabs/open-data-docs/tree/main/docs/noaa/noaa-ghcn) repository. The NOAA documentation shows that the first column is the station ID (ID). The date is formatted as YYYYMMDD in the DATE column, for example, 17630127 means January 27, 1763. The ELEMENT column contains maximum temperature (TMAX), minimum temperature (TMIN), snow depth (SNWD), and precipitation (PRCP). DATA VALUE is the actual value for ELEMENT.

By viewing the file locally, you can observe that Station (ID) ITE00100554 had a maximum temperature of 9 on 17630131 (January 31, 1763), as shown in the following screenshot.
![image](https://github.com/user-attachments/assets/04a22d7b-f22d-4132-88d9-4fcda6657241)
*Figure 3: 1763.csv showing 17630131 TMAX of 9 for ID ITE00100554*

### Create private S3 bucket
1.	On the [Amazon S3 console](https://console.aws.amazon.com/s3/), choose **Create bucket** and name the bucket `YOURNAME-ghcn-1763-1764`, replacing `YOURNAME` with your last name to make the bucket unique. Leave everything else on the page as the default and choose **Create bucket** at the bottom of the page.
2.	After the bucket is created, select it to view the bucket contents (it will be empty). 
3.	Drag and drop the 1763.csv and 1764.csv files into the bucket and choose **Upload**. Wait for both files to be copied into the bucket, as shown in the following screenshot.
a.	Or use the Upload button. You can drag and drop the 1763.csv and 1764.csv files or choose **Add files** to add them.
b.	Choose **Upload**, and wait for both files to be copied into the bucket, as shown in the following screenshot.
![image](https://github.com/user-attachments/assets/bfc0b066-f870-4161-a398-9b218e9a6e53)
*Figure 4: Files 1764.csv and 1763.csv in **Files and folders***



