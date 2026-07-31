---
title: "Create an Amazon S3 Data Lake"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

Amazon S3 will serve as the central **Data Lake**, storing both **raw** and **processed** data throughout the project.

#### Steps in the AWS Management Console

1. Sign in to the AWS Management Console, search for **Amazon S3**, and choose **Create bucket**.

2. Under **General configuration**:
   * **Bucket name:** Enter a bucket name. For example: `local-aqi-dev-s3-raw`.

   > **Note:** Amazon S3 bucket names must be globally unique. To avoid naming conflicts, add your name or a random suffix to the bucket name.

   ![Bucket name configuration](/images/5-Workshop/5.3-data/5.3.1.1.png)

3. Under **Block Public Access settings for this bucket**:
   * Make sure **Block all public access** is enabled. This is an important security setting that protects your project's internal data from unauthorized public access.

   ![Block public access](/images/5-Workshop/5.3-data/5.3.1.2.png)

4. Scroll down to **Tags (optional)**. Resource tags help organize AWS resources and simplify cost management. Choose **Add new tag** and create the following tags:

   * **Key:** `Project` | **Value:** `local-aqi-forecasting`
   * **Key:** `Environment` | **Value:** `dev`
   * **Key:** `Owner` | **Value:** `quynh-tam`
   * **Key:** `Module` | **Value:** `data`

   ![AWS resource tags](/images/5-Workshop/5.3-data/5.3.1.3.png)

5. Scroll to the bottom of the page and choose **Create bucket**.

   ![Created S3 bucket](/images/5-Workshop/5.3-data/5.3.1.4.png)

---

#### Creating Folder Prefixes

After the bucket has been created, open it and create the following root folders by selecting **Create folder** for each one:

* `ml/` – Stores Machine Learning-related data and artifacts.
* `models/` – Stores trained Machine Learning models.
* `monitoring/` – Stores logs and monitoring data.
* `processed/` – Stores cleaned and transformed datasets (you can create an additional `ml-ready/` subfolder inside this folder).
* `raw/` – Stores raw data automatically delivered by Amazon Kinesis Data Firehose.

**Expected Result:**  
Your Amazon S3 bucket should contain the following root folder structure:

![S3 folder structure](/images/5-Workshop/5.3-data/5.3.1.5.png)

> **Note:** Data stored in the `raw/` folder will be automatically partitioned by time when Amazon Kinesis Data Firehose is configured in the next section. Therefore, you do not need to manually create date-based subfolders.
