# How to use data from AWS Open Data program in Amazon Bedrock

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

### 1760s knowledge base
Using the data in the private bucket, create your first knowledge base. To use the FMs in Amazon Bedrock, you need to request access first. Follow these steps to request access and create a knowledge base:

1.	On the [Amazon Bedrock console](https://console.aws.amazon.com/bedrock) in the navigation pane, under **Bedrock configurations** in the left navigation pane, choose Model access.
2.	On the Model access page, choose **Modify model access**. 
3.	Select the check box next to **Titan Text G1 - Premier, Titan Embeddings G1 - Text,** and **Nova Pro** and choose **Next**, then **Submit** to request access to these models.
4.	Under **Builder tools**, select **Knowledge Bases**. 
5.	Choose **Create** and select **Knowledge Base with vector store**. We use vector store for this walkthrough, which uses [Amazon OpenSearch Serverless](https://aws.amazon.com/opensearch-service/features/serverless/), but there are other options you can explore at [Retrieve data and generate AI responses with Amazon Bedrock Knowledge Bases in the Amazon Bedrock User Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html).
6.	Name the knowledge base `YOURNAME-GHCN-1763-1764`, replacing `YOURNAME` with your name. Leave the rest of the page as the default and choose **Next**.
7.	Under **Data source**, change the following:
a.	For **Data source location**, leave as **This AWS account**.
b.	For **S3 URI**, choose **Browse** and select the `YOURNAME-ghcn-1763-1764` bucket that you created earlier.
8.	Leave the rest of the page as defaults, and click **Next**.
9.	Here will need to select an Embeddings Model to use, so click **Select model** and choose **Titan Text Embeddings V2** and then **Apply**.
10.	Leave the rest of the page as the default and choose **Next**.
11.	Review your selections and choose **Create Knowledge Base** when you’re ready.

This process will take a few minutes to prepare the vector database in Amazon OpenSearch Serverless. Note that the Amazon OpenSearch Serverless collection incurs a cost even with the small amount of data that used in this walkthrough. It’s recommended to set up a [Cost Explorer budget](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html) or alarm to watch your costs.

After the vector database has been created, you’ll receive a notification that “Amazon OpenSearch Serverless vector database is ready,” and then the knowledge base will be created. You now have an empty vector store and need to fill it with data. To do this, you need to sync the data:

1.	Under **Data source**, select the data source you created, as shown in the following screenshot.

![image](https://github.com/user-attachments/assets/c26babd8-4362-4ebb-87b9-63a5f3bc1ce0)

*Figure 5: GHCN-1763-1764 knowledge base ready to sync with the selected data source*

2.	To start adding data to the vector store, choose **Sync**. It should only take a few moments because there are only two files in your private bucket. 

Because we know a few values in the two files, we can open them locally and ask questions that we already know the answer to. Doing so will verify that the knowledge base is both using the data in the files and using the data correctly. To test your knowledge base, follow these steps:

1.	To select a model to use for the test, under **Test Knowledge Base**, choose **Select model**. Choose **Nova Lite** and then choose **Apply**. There are other models that you can use that will return similar answers.
2.	To ask the knowledge base a question, paste the following into the Enter your message here box and then choose Run.

`What was the TMAX on 17630131?`

We know the answer from the file is 9, as we showed earlier. The knowledge base will provide the following answer: 

`The TMAX on 17630131 was 9 degrees`

3.	Select **Show details >** to learn how the knowledge base derived its answer. The metadata tells you the file (source-uri) where Amazon Bedrock found information to form an answer. The **Source chunk** shows the actual chunk of data used and will look like this:

`, ITE00100554,17630124,TMIN,10,,,E, ITE00100554,17630125,TMAX,24,,,E, ITE00100554,17630125,TMIN,-2,,,E, ITE00100554,17630126,TMAX,6,,,E, ITE00100554,17630126,TMIN,-22,,,E, ITE00100554,17630127,TMAX,1,,,E, ITE00100554,17630127,TMIN,-27,,,E, ITE00100554,17630128,TMAX,-5,,,E, ITE00100554,17630128,TMIN,-33,,,E, ITE00100554,17630129,TMAX,-1,,,E, ITE00100554,17630129,TMIN,-29,,,E, ITE00100554,17630130,TMAX,4,,,E, ITE00100554,17630130,TMIN,-16,,,E, **ITE00100554,17630131,TMAX,9,,,E,** ITE00100554,17630131,TMIN,-21,,,E, ITE00100554,17630201,TMAX,19,,,E, ITE00100554,17630201,TMIN,-11,,,E, ITE00100554,17630202,TMAX,28,,,E, ITE00100554,17630202,TMIN,-2,,,E,`

4.	The following chunk is our answer:

`ITE00100554, 17630131, TMAX,9,,,E,`

Try a few more questions to get an idea of how the knowledge base parses information and derives answers.

### Cleanup
To avoid incurring future charges, you need to delete the knowledge base when you are done. To delete the knowledge base:

1.	On the Amazon Bedrock console under **Builder tools**, select **Knowledge Bases** and select the `YOURNAME-GHCN-1763-1764` knowledge base.
2.	If you chose to retain the vector store in advanced settings when you created the knowledge base, you will need to find the Amazon OpenSearch index name before you delete the knowledge base. 
  a.	Scroll down to **Vector database** and make note of the **Collection ARN:**

`arn:aws:aoss:REGION:AWSACCOUNTID:collection/UUID`, where `UUID` will be a unique identifier for your collection, such as `ea50z3iuyaavwy8bymq4`

  b.	On the OpenSearch Service console in a new tab or browser window, choose **Collections** under **Serverless** and select the collection you created in this walkthrough to open it.
  c.	Verify that the **Collection ARN** matches the collection [Amazon Resource Name (ARN)](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html) that you received for the vector database: 

`arn:aws:aoss:REGION:AWSACCOUNTID:collection/UUID`

  d.	To delete the vector database, choose **Delete collection** and enter `confirm` in the dialog box.
  e.	Return to the Amazon Bedrock tab to complete the last step of deleting the knowledge base, leaving **Delete the vector store** unchecked.
3.	Choose **Delete** and enter `delete` in the window.

