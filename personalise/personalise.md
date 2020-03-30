## Creating a recommendation using Amazon Personalize
In this lab you will learn the basics of how to use Amazon Personalize in order to create a recommendation system.

![image](https://d1.awsstatic.com/r2018/r/Concierge/product-page-diagram_amazon_personalize_how-it-works.3ceac8883c7d6bd67d7cf26d8a7d505520d02a40.png)

## Contents
1. Concepts & Definitions
2. Preparing your data
3. Creating your Personalize Dataset
4. Creating & Training your Personalize Solution
5. Retrieving a Recommendation
6. Personalize CLI
7. Personalize Programmatic SDK
8. Author & Feedback

----

## 1. Concepts & Definitions

- Dataset Groups
    A Dataset Group are domain specific containers for your recommendations

- Datasets

    Datasets are data used in order to create solutions which then generate recommendations

- Schema

    The datasets which you will use in Personalise needs a Schema defined before import, this is provided as a JSON string.

- Solution

    A solution is a custom model generated on your datasets to provide recommendations

- Launch Campaign

    A campaign allows an application to retrieve recommendations. Analytics on a campaign's usage is also available

----

## 2. Preparing your data

In order to use personalize, you need to have a csv dataset of each of these types:

1. Users
2. Items
3. User-Item Interactions

Preferrably, you would use all three otherwise it is not very useful.

You may use these example datasets available below:

[item.csv](https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/datasets/item.csv)

[user-interactions.csv](https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/datasets/user-interactions.csv)

[users.csv](https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/datasets/users.csv)

You may also use the following dataset schemas available below:

[items_schema.json](https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/schema/personalise-items-schema.json)

[users_schema.json](https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/schema/personalise-users-schema.json)

[events_schema.json](https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/schema/personalise_user_events_schema.json)

----

## 3. Creating your Personalize Dataset Group

### 3.1 Start by creating your dataset, by providing a memorable name.

![01](https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/lab-images/01-dataset.png)

### 3.2 Dataset Details & Schema

You will now have to input your dataset name, and also **the name of the schema**. Make sure the Schema Name is relevant to the dataset you are about to upload.

![02](https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/lab-images/02-dataset-schema.png)

Then, input the json string and hit next.

![03](https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/lab-images/03-dataset-schema-01.png)

### 3.3 Importing Data

Fillout the import job name, and if you haven't created an IAM service role, select the **"Create a new role"** option.

Then fill in the S3 location, taking note of the s3 url format.

![04](https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/lab-images/04-dataset-import.png)

Fastest way to copy the path name is to use the "copy path" option when a s3 item is selected.

<img src="https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/lab-images/05-dataset-data.png" width=400>

Once you have successfully imported all three data types, you may move onto creating a solution.

<img src="https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/lab-images/06-dataset-import.png" height=200>

----

## 4. Creating & Training your Personalize Solution

Currently, as of 26-MAR-2020, Amazon Personalize in Sydney region supports the following algorithms:

| Algorithm | Explanation |
| ---- | ---- |
| aws-hrnn | Predicts items a user will interact with. A Hierarchical Recurrent Neural Network which models the temporal order of user-item interactions. |
| aws-hrnn-coldstart | Predicts items a user will interact with. HRNN - metadata with personalized exploration of new items. |
| aws-hrnn-metadata | Predicts items a user will interact with. HRNN with additional features derived from contextual metadata (user-item interactions metadata), user metadata (user dataset) and item metadata (item dataset). |
| aws-personalized-ranking | Reranks an input list of items for a given user. Trains on user-item interactions dataset. |
| aws-popularity-count | Calculates popularity of items based on total number of events for each item in the user-item interactions dataset. |
| aws-sims | Computes items similar to a given item based on co-occurrence of items in the user-item interactions dataset. |

**Note**

In order to train your solution, ensure that *Amazon Personalize* has access to the S3 bucket. (S3 > Permissions > Bucket Policy)

```
# S3 Bucket Policy

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:ListBucket",
            "Resource": <bucket>
        },
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "<bucket>/*"
        }
    ]
}
```
-----

## 5. Retrieving a Recommendation

Once the Solution has been made available, you can pass a user id in using a SDK, and view the recommendation.

<img src="https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/lab-images/08-solution-01.png" width=400 />
<img src="https://steven-devlabs.s3-ap-southeast-2.amazonaws.com/public/personalise/lab-images/08-solution-02.png" width=400 />

----

## 6. Personalize CLI

For Running a personalize campaign using CLI, you can use the command below:

```
aws personalize-runtime get-recommendations --campaign-arn <arn> --user-id <userid>
```

----

## 7. Personalize Programmatic SDK
Please see the documentation [here](https://docs.aws.amazon.com/personalize/latest/dg/getting-started-python.html) for running Personalize using Python, or another language like [nodejs](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/Personalize.html).

----

## 8. Author & Feedback

If you have any feedback, concerns or would like to have a chat, please send me an email.

Steven Tseng (stetseng@amazon.com)

Solutions Architect - Digital Natives MEL/SYD