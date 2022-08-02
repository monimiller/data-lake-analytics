# data-lake-analytics

This is a repository to demonstrate Data Lake analytics with Starburst Galaxy on AWS
>"The main challenge with a data lake architecture is that raw data is stored
>with no oversight of the contents.”

### Welcome

Welcome to the Data Lake Analytics Reporting Structures tutorial with Starburst
Galaxy on AWS. The intent of the tutorial was to demonstrate a feasible example
of data lake reporting structures. With AWS S3 as the data lake and Starburst
Galaxy serving as the analytics engine, I hope you were able to experience
firsthand the benefits of implementing comprehensive data lake analytics
solutions.

I chose to use a public dataset because transparency is extremely important to
me, and I wanted the lab to be reproducible by anyone at any time, without any
barriers. I consider myself at least partially a kinesthetic learner, and I
personally have only been able to buy into the value of something once I could
explore it and then adopt it on my own. Since we are utilizing the [AWS Covid 19
Data Lake](https://aws.amazon.com/covid-19-data-lake/), all you need to try this
tutorial out for yourself is a set of AWS credentials (you can create a [free
account](https://www.google.com/search?q=aws+free+account&oq=AWS+free+account&aqs=chrome.0.69i59j0i512l5j69i60l2.3577j0j4&sourceid=chrome&ie=UTF-8) as well) and a [Starburst Galaxy](https://www.starburst.io/platform/starburst-galaxy/) account.

To learn more about these concepts, visit our blog post which dives deeper into
the reason behind this tutorial.

### Data Lake Reporting Structures

Ultimately, we want to make sure that the data lake stays as clean as possible.

_Land layer_ - *stores unmodified source data at any level of granularity*
_Structure layer_ - *stores joined, enriched, cleansed data*
_Consume layer_ - *stores aggregated data that is ready to be queried*

We will create our own reporting structure based off the Covid-19 data lake.
First, we will land the raw tables in our own S3 bucket. Next, we will cleanse
and enrich some of the data available. Finally, we will create an aggregated
tables that are ready to use by a data analyst.

### Covid - 19 Data Lake Prompt





## Tutorial

### Set up your Environment

1. Sign up for an [AWS
   account](https://www.google.com/search?q=aws+free+account&oq=AWS+free+account&aqs=chrome.0.69i59j0i512l5j69i60l2.3577j0j4&sourceid=chrome&ie=UTF-8)
   and for a [Starburst
   Galaxy](https://www.starburst.io/platform/starburst-galaxy/) free trial.
2. Create a S3 bucket in the Ohio region (us-east-2). You must specify this
   region because this is where the COVID-19
   public data lake exists. Use all the defaults.
3. Create an AWS access key that will be used as the Starburst Galaxy
   [Authentication method for connecting to S3](https://docs.starburst.io/starburst-galaxy/security/external-aws.html)
   - Go to the IAM Management Console
   - Select *Users*
   - Select *Add Users*
   - Provide a Descriptive User Name like *<initials>-aws-covid*
   - Select AWS Credential Type: *Access key - Programmatic access*
   - Set Permissions: *Attach existing policies directly*
   - Add the following policy: *AmazonS3FullAccess*
4. Finish creating the access key with the rest of the defaults, and then save
   your AWS Access Key and Secret Access Key.

### Create a Catalog in Starburst Galaxy

Catalogs contain the proper configuration and connection information needed to
access a data source. To gain this access, configure a catalog and use it in a
cluster. We're going to configure the S3 catalog to access our S3 bucket.

1. Navigate to the *Catalogs* tab. Click *Configure a Catalog*.
2. Create an S3 Catalog.
   - Catalog name: *<initals>_aws_lab*
   - Add a relevant description
   - Authenticate to S3 through the AWS Access Key/Secret created earlier
   - Metastore configuration: *"I don't have a metastore"*
   - Default directory name: *<initals>_metadata*
   - Enable *Allow creating external tables*
   - Enable *Allow writing to external tables*
   - Select default table format: *Hive*
   - Hit _Skip_ the *Set Permissions* page

We picked the default table format hive because of its familiarity in the big
data space. Any read or write access to existing tables works transparently for
all table formats. Starburst Galaxy recognizes the format of your tables by
reading the metastore associated with your object storage. Learn more about the
[Great Lakes
connectivity](https://docs.starburst.io/starburst-galaxy/sql/great-lakes.html)
and table formats.

### Create a Cluster in Starburst Galaxy

A cluster in Starburst Galaxy provides the resources necessary to run queries
against your catalogs. You can access the catalog data exposed by running
clusters in the Query Editor.

1. Click *Create a new cluster*
2. Enter cluster name: *<intials>-aws-lab*
3. Cluster size: *Free*
4. Cluster type: *Standard*
5. Catalogs: *<initals>_aws_lab* (select the catalog previously created)
6. Cloud provider region: *US East (Ohio)* aka *us-east-2*

Our environment setup is complete. Navigate back to the Query Editor.

## Tutorial Time
rephrase goal

### Create a Schema










