# data-lake-analytics

This is a repository to demonstrate Data Lake analytics with Starburst Galaxy on AWS
>"The main challenge with a data lake architecture is that raw data is stored
>with no oversight of the contents.”

## Welcome

Welcome to the Data Lake Analytics Reporting Structures tutorial with Starburst
Galaxy on AWS. The intent of the tutorial is to demonstrate a feasible example
of data lake reporting structures. With AWS S3 as the data lake and Starburst
Galaxy serving as the analytics engine, I hope you are able to experience
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

## Data Lake Reporting Structures

Ultimately, we want to make sure that the data lake stays as clean as possible.

- **Land layer**: *stores unmodified source data at any level of granularity*
- **Structure layer**: *stores joined, enriched, cleansed data*
- **Consume layer**: *stores aggregated data that is ready to be queried*

We will create our own reporting structure based off the Covid-19 data lake.
First, we will land the raw tables in our own S3 bucket. Next, we will cleanse
and enrich some of the data available. Finally, we will create aggregated
tables that are ready to use by a data analyst.

# Tutorial

## Set up your Environment

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
   - Provide a Descriptive User Name like ```<username>-aws-covid```
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
   - Catalog name: ``` <username>_aws_lab```
   - Add a relevant description
   - Authenticate to S3 through the AWS Access Key/Secret created earlier
   - Metastore configuration: *"I don't have a metastore"*
   - Default directory name: ```<username>_metadata```
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

   - Click *Create a new cluster*
   - Enter cluster name: ``` <username>-aws-lab```
   - Cluster size: *Free*
   - Cluster type: *Standard*
   - Catalogs: ```<username>_aws_lab``` (select the catalog previously created)
   - Cloud provider region: *US East (Ohio)* aka *us-east-2*

Our environment setup is complete. Navigate back to the Query Editor.

## Tutorial Time
We want to ultimately instill some basic data lake reporting structures for the
data we have been provided.

### Create a Schema
The first step to create the reporting structures is to create a schema.

```sql
create schema <schema_name> with (location='<s3_location>');
```

```sql
create schema covid_data with (location='s3://mm-aws-covid/');
```


Now, in the top right corner, select your cluster, catalog, and schema so that
you can easily run the lab queries.

### Create Land Tables

_Land layer_ - *stores unmodified source data at any level of granularity*
#### enigma_jhu Land Table

The first dataset we want to utilize is the _Global Coronavirus Data_ sourced
from John Hopkins and provided by Enigma. This data tracks confirmed COVID-19
cases in provinces, states, and countries across the world, while also providing
a county level breakdown in the United States.

Run the SQL command to create the table:
``` sql
create table enigma_jhu (
   fips VARCHAR,
   admin2 VARCHAR,
   province_state VARCHAR,
   country_region VARCHAR,
   last_update VARCHAR,
   latitude DOUBLE,
   longitude DOUBLE,
   confirmed INTEGER,
   deaths INTEGER,
   recovered INTEGER,
   active INTEGER,
   combined_key VARCHAR
)
WITH (
   format = 'json',
   EXTERNAL_LOCATION = 's3://covid19-lake/enigma-jhu/json/')
;
```

The json file is available
[here](https://s3.console.aws.amazon.com/s3/buckets/covid19-lake?region=us-east-2&prefix=enigma-jhu/json/&showversions=false).

Run a select all command to view your results:
```sql
select * from enigma_jhu;
```

The goal is to ultimately land this data, clean it up, and then create a
rollup table with the relevant data for both the United States and Australia.

#### hospital_beds Land Table

The second dataset we will utilize is the _USA Hospital Beds_ data sourced by
Definitive Healthcare and provided by Rearc. This data provides intelligence on
the number of licensed beds, staffed beds, and ICU beds for the hospitals in the
United States.

Run the SQL command to create the table:
``` sql
create table hospital_beds (
   objectid INTEGER,
   hospital_name VARCHAR,
   hospital_type VARCHAR,
   hq_address VARCHAR,
   hq_address1 VARCHAR,
   hq_city VARCHAR,
   hq_state VARCHAR,
   hq_zip_code VARCHAR,
   county_name VARCHAR,
   state_name VARCHAR,
   state_fips VARCHAR,
   cnty_fips VARCHAR,
   fips VARCHAR,
   num_licensed_beds INTEGER,
   num_staffed_beds INTEGER,
   num_icu_beds INTEGER,
   adult_icu_beds INTEGER,
   pedi_icu_beds INTEGER,
   bed_utilization DOUBLE,
   avg_ventilator_usage VARCHAR,
   potential_increase_in_bed_capac INTEGER,
   latitude DOUBLE,
   longitude DOUBLE
)
WITH (
   format = 'json',
   EXTERNAL_LOCATION = 's3://covid19-lake/rearc-usa-hospital-beds/json/')
;
```

Run a select all command to view your results:
```sql
select * from hospital_beds;
```

The json file is available [here](https://s3.console.aws.amazon.com/s3/buckets/covid19-lake?region=us-east-2&prefix=rearc-usa-hospital-beds/json/&showversions=false).

The goal is to ultimately land this data, clean it up, and then create a
rollup table with the relevant data for both the United States and Australia.


### Create Structure Tables

_Structure layer_ - *stores joined, enriched, cleansed data*


### enigma_jhu Structure Tables

First we want to rename the county column that is still identified as *admin_2*.
We also want to remove the columns that we do not need for our analysis, and
filter out the records that do not apply to the relevant country. The last where
clause *province_state not like ''* is to filter out any rows that have no value
for their province or state.

#### Create the Australia Structure Table

```sql
create table
    ejhu_stg_australia as
select
    fips,
    admin2 as county,
    province_state,
    country_region,
    last_update,
    confirmed,
    recovered,
    active
from
    enigma_jhu
where
    country_region = 'Australia'
    and province_state not like '';
```
Run a select all command to view your results:
```sql
select * from ejhu_stg_australia;
```


#### Create United States Structure Table

```sql
create table
    ejhu_stg_us as
select
    fips,
    admin2 as county,
    province_state,
    country_region,
    last_update,
    confirmed,
    recovered,
    active
from
    enigma_jhu
where
    country_region = 'US'
    and province_state not like '';
```

Run a select all command to view your results:
```sql
select * from ejhu_stg_us;
```
### hospital_beds Structure Tables
We want to remove any unnecessary fields for our analysis.

```sql
create table hospital_beds_stg as
select
   hospital_name,
   county_name,
   state_name,
   fips,
   num_licensed_beds,
   num_staffed_beds,
   num_icu_beds,
   adult_icu_beds,
   pedi_icu_beds,
   potential_increase_in_bed_capac
   from hospital_beds;
```

Run a select all command to view your results:
```sql
select * from hospital_beds_stg;
```
### Create Consume Tables
_Consume layer_ - *stores aggregated data that is ready to be queried*

### enigma Consume Tables
The aggregations we want to create are the summation of all confirmed and active
cases for each province or state. Because the table shows a running total of
confirmed cases (the confirmed cases accumulate for each update), we need to
select only the latest timestamp. This case will lead to filtering out
additional records.

```sql
select max(last_update) from enigma_jhu;
```

####  Create Australia Consume Table

```sql
create table
    ejhu_consume_au as
select
    province_state,
    sum(confirmed) as confirmed_cases,
    sum(active) as active_cases
from
    ejhu_stg_australia
where
    last_update = '2020-05-30T02:32:48'
group by
    province_state;
```
Run a select all command to view your results:
```sql
select * from ejhu_consume_au;
```

####  Create US Consume Table
```sql
create table
    ejhu_consume_us as
select
    province_state,
    sum(confirmed) as confirmed_cases,
    sum(active) as active_cases
from
    ejhu_stg_us
where
    last_update = '2020-05-30T02:32:48'
group by
    province_state;
```
Keep in mind that this dataset includes US territories as well as states. Run a
select all command to view your results:
```sql
select * from ejhu_consume_us;
```

#### Create Combined Consume Table

Let's intend the final business ask is to also create a table to only monitor
the COVID cases for the provinces where the business has an office. Offices are
in Queensland, New York, California, Florida, and Australian Capital Territory.

```sql
create table ejhu_consume as
select
    a.province_state,
    sum(a.confirmed) as confirmed_cases,
    sum(a.active) as active_cases
from
    ejhu_stg_australia a
    where
    a.last_update = '2020-05-30T02:32:48'
    and a.province_state in ('Queensland', 'Australian Capital Territory')
group by
    a.province_state
UNION
select
    u.province_state,
    sum(u.confirmed) as confirmed_cases,
    sum(u.active) as active_cases
from
    ejhu_stg_us u
    where
    u.last_update = '2020-05-30T02:32:48'
    and u.province_state in ('New York', 'California', 'Florida')
group by
    u.province_state
order by province_state;
```

 Run a select all command to view your results:
```sql
select * from ejhu_consume;
```

### hospital_beds Consume Table

Create the summation of each type of bed as the consume table.

```sql
create table
    hospital_beds_consume as
SELECT
    state_name,
    sum(num_licensed_beds) as num_licensed_beds,
    sum(num_staffed_beds) as num_staffed_beds,
    sum(num_icu_beds) as num_icu_beds,
    sum(adult_icu_beds) as adult_beds,
    sum(pedi_icu_beds) as ped_beds,
    sum(potential_increase_in_bed_capac) INTEGER
FROM
    hospital_beds_stg
group by
    state_name
order by
    state_name;
```

 Run a select all command to view your results:
```sql
select * from hospital_beds_consume;
```

### Query Federation from Staging Tables
We now want to evaluate each State's current cases versus the number of licensed
beds during May 30, 2020. You can see for each state if the stat is running
close to hospital capacity, or if they are managing with the current number of
infections.

```sql
create table
    states as
SELECT
    province_state,
    total_confirmed_cases,
    sum(num_licensed_beds) as num_licensed_beds,
    sum(num_staffed_beds) as num_staffed_beds,
    sum(num_icu_beds) as num_icu_beds
FROM
    hospital_beds_stg beds,
    (
        SELECT
            province_state,
            sum(confirmed) as total_confirmed_cases,
            last_update
        FROM
            ejhu_stg_us
        WHERE
            last_update = '2020-05-30T02:32:48'
            and province_state not like ''
        group by
            province_state,
            last_update
    ) cases
WHERE
    beds.state_name = cases.province_state
GROUP BY
    province_state,
    total_confirmed_cases;
```

## Implement Role Based Access Control
To ensure data analysts only have access to the consume tables, add the
role-based access control configurations.

1. Navigate to ``` Access Control - Roles and Privileges```
2. Add role named: ```  <username>_covid_lab ```
3. Entity kind: *Table*
4. Select your catalog
5. Enter the proper schema
6. Table: `ejhu_consume`
7. Privileges: `SELECT from table`
8. Hit *Add privileges*

Now, go back to the query editor, and switch your role in the top right hand
corner. Select from the table, and try selecting from a table that has not been
granted permissions yet. To learn more about role-based access control in
Starburst Galaxy, check out [my blog](https://www.starburst.io/blog/why-granularity-impacts-role-based-access-control/) on Granularity and RBAC.


Thanks for completing the data lake analytics tutorial.
