---
title: "Create a reporting structure"
---

# {{page.title}}

One of {{site.terms.sg}}'s many uses is as an analytics engine on your data
lakehouse or data lake. Once data is landed in S3, GCP, or Azure, or your
preferred storage of choice, you can easily capitalize on the "separation of
storage and compute" principle and
use {{site.terms.sg}} as the engine to build a reporting structure.

## Data lakehouse tutorial architecture

The three layers you will create in your reporting structure are:

- **Land layer**: *stores unmodified source data at any level of granularity*
- **Structure layer**: *stores joined, enriched, cleansed data*
- **Consume layer**: *stores aggregated data that is ready to be queried*

{% include image.html
  url='../../assets/img/galaxy/data-lakehouse-reporting-structure.png'
  img-id='data-lakehouse-reporting-structure'
  alt-text='Image displaying a data lake reporting structure'
  descr-text='Image displaying a data lake reporting structure'
  screenshot='true'
%}


For this tutorial, you will take two different datasets from the Covid-19 data
lake, add a reporting structure to each, and federate the two structure level
tables together into a final report to be used by your data analyst.
- The first dataset is the [Global Coronavirus (Covid-19)
Data](https://aws.amazon.com/marketplace/pp/prodview-vtnf3vvvheqzw?sr=0-1&ref_=beagle&applicationId=AWSMPContessa#offers)
provided by Enigma. This dataset tracks Covid cases.
- The second dataset shares information on [US Hospital
Beds](https://aws.amazon.com/marketplace/pp/prodview-yivxd2owkloha?qid=1585241268884&sr=0-8&ref_=srh_res_product_title),
provided by Rearc. With this hospital information, you will be able to see the
capacity and occupancy of hospital beds for each state.

This guide will walk through:
- Configuring your AWS account
- Creating a Starburst Galaxy account
- Connecting Starburst Galaxy to AWS
- Run schema discovery
- Creating a reporting structure
- Configuring role-based access control


## Configure your AWS account

First, sign into your AWS account. If you do not have access to an AWS account,
follow [the
instructions](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all)
to create one.

1. Create a S3 bucket in the Ohio region (us-east-2), with a descriptive name
   such as ```<username>-covid-lab``` . You must specify the Ohio region because
   this is where the COVID-19 public data lake exists. Use all the defaults.

2. Navigate within the newly created bucket and create three folders within that
   bucket to store each layer of the reporting structure:
   -  land
   -  structure
   -  consume

3. Create an AWS access key that will be used as the
   [authentication method for connecting from {{site.terms.sg}} to
   S3](https://docs.starburst.io/starburst-galaxy/security/external-aws.html)
   - Go to the IAM Management Console
   - Select *Users*
   - Select *Add Users*
   - Provide a Descriptive User Name like ```<username>-aws-covid```
   - Select AWS Credential Type: *Access key - Programmatic access*
   - Set Permissions: *Attach existing policies directly*
   - Add the following policy: *AmazonS3FullAccess*

4. Finish creating the access key with the rest of the defaults, and then save
   your AWS Access Key and Secret Access Key.

## Create a Starburst Galaxy account

1. Navigate to the [Starburst Galaxy login
   page](https://galaxy.starburst.io/login).

2. If you already don't have an account, create a new one and verify the account
   with your email.

## Connect Starburst Galaxy to AWS

### Create a Catalog in Starburst Galaxy

Catalogs contain the proper configuration and connection information needed to
access a data source. To gain this access, configure a catalog and use it in a
cluster. Configure the S3 catalog to access your newly created S3 bucket.

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
   - Hit _Skip_ on the *Set Permissions* page

For the purpose of this tutorial, the data will be landed in a raw JSON file to
replicate the raw format from multiple storage systems. For the structure and
consume layer tables, you will use an open table format such as Iceberg or Delta
to improve performance.
Using the Great Lakes connectivity, {{site.terms.sg}} lets you use Iceberg,
Delta, and Hive table formats in the same catalog. Learn more about the [Great
Lakes
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


## Run schema discovery

Schema discovery is a special feature in {{site.terms.sg}} which analyzes a file
to infer the schema, and the tables and columns within that schema. Use schema
discovery on the covid-19 data lake to generate a table to be used in the
reporting structure from the [JSON file](https://s3.console.aws.amazon.com/s3/buckets/covid19-lake?region=us-east-2&prefix=enigma-jhu/json/&showversions=false).

Navigate to your newly created catalog within the cluster explorer. Select the
three dots on the right side of the catalog and select ```Run discovery```.

{% include image.html
  url='../../assets/img/galaxy/run-schema-discovery.png'
  img-id='data-lakehouse-reporting-structure'
  alt-text='Image displaying a data lake reporting structure'
  descr-text='Image displaying a data lake reporting structure'
  screenshot='true'
%}

Enter the URI to be discovered. Use the Global Coronavirus (Covid-19)
Data provided by enigma. Hit run discovery.


```
s3://covid19-lake/enigma-jhu/json
```

You now have the schema discovery results which provide the SQL to create the
first table, without spending any time investigating yourself.

## Create the land layer

The land layer holds the raw data. In this case, the land layer will hold the
two tables based on the source - the public data lake. Create a schema to hold
the land tables which points to the S3 bucket location previously created in the
configurations section.

```sql
CREATE SCHEMA land WITH (location='<s3_location>/land/');

--- example statement:
--- CREATE SCHEMA land WITH (location='s3://mm-covid-lab/land/');
```

Now, in the top right corner, select your cluster, catalog, and schema so that
you can easily run the lab queries.


### enigma_jhu land table

The enigma_jhu dataset provides the _Global Coronavirus Data_ and is sourced
from John Hopkins and provided by Enigma. This data tracks confirmed COVID-19
cases in provinces, states, and countries across the world, while also providing
a county level breakdown in the United States.

Run the SQL command to create the table from the [available JSON
file](https://s3.console.aws.amazon.com/s3/buckets/covid19-lake?region=us-east-2&prefix=enigma-jhu/json/&showversions=false):

``` sql
CREATE TABLE ejhu_land (
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

Run a select all command to view your results:

```sql
SELECT * FROM ejhu_land;
```

Take notice that the ```admin2``` column is actually the County. However, it has
been improperly named. You can also see that the case information is aggregated
for each previously updated timestamp. You will account for this in
the structure layer.

### hospital_beds land table

The second dataset we will utilize is the _USA Hospital Beds_ data sourced by
Definitive Healthcare and provided by Rearc. This data provides intelligence on
the number of licensed beds, staffed beds, and ICU beds for the hospitals in the
United States.

Run the SQL command to create the table from the [available JSON
file](https://s3.console.aws.amazon.com/s3/buckets/covid19-lake?region=us-east-2&prefix=rearc-usa-hospital-beds/json/&showversions=false):

``` sql
CREATE TABLE hb_land (
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
   longtitude DOUBLE
)
WITH (
   format = 'json',
   EXTERNAL_LOCATION = 's3://covid19-lake/rearc-usa-hospital-beds/json/')
;
```

Run a select all command to view your results:
```sql
SELECT * FROM hb_land;
```

## Create the structure layer

Now that the land layer has been validated, create the structure layer using the
ORC file format and the Iceberg table format for faster querying. Next, run the
```INSERT``` queries that can be periodically executed to pull data from the
landing tables, formatted, and then inserted into the structure tables using
{{site.terms.sg}} for batch processing in combination with your favorite
scheduler such as Airflow or Dagster.

Create a schema to hold the structure tables which points to the S3 bucket
location previously created in the configuration
section.

```sql
CREATE SCHEMA structure WITH (location='<s3_location>/structure/');

--- example statement:
--- CREATE SCHEMA structure WITH (location='s3://mm-covid-lab/structure/');
```

Now, in the top right corner, select your cluster, catalog, and schema so that
you can easily run the lab queries.

{% include image.html
  url='../../assets/img/galaxy/land-schema.png'
  img-id='data-lakehouse-reporting-structure'
  alt-text='Image displaying a data lake reporting structure'
  descr-text='Image displaying a data lake reporting structure'
  screenshot='true'
%}

### Create the enigma_jhu structure table

Rename the county column that is still identified as *admin_2* and remove the
columns that you do not need for your analysis. Since our default table format
is still Hive, you will need to either update the Catalog or add the table
format within the SQL statement demonstrated in the ```CREATE TABLE``` query.

```sql
CREATE TABLE ejhu_structure (
    fips VARCHAR,
    county VARCHAR,
    province_state VARCHAR,
    country_region VARCHAR,
    confirmed INTEGER
)
WITH
    (TYPE = 'iceberg',
     FORMAT = 'ORC');
```

Now that the table is created, run the insert query that will filter out rows
that are not within the US and do not have a value for any province or state.
Since the case information is aggregated for each previously updated timestamp,
only select the most recent timestamp on May 30, 2020.

```sql
INSERT INTO ejhu_structure
SELECT
    fips,
    admin2,
    province_state,
    country_region,
    confirmed
FROM
    ejhu_land
WHERE
    country_region = 'US'
    and province_state not like ''
    and last_update = '2020-05-30T02:32:48';
```

Keep in mind that this dataset includes US territories as well as states. Run a
select all command to view your results:

```sql
SELECT * FROM ejhu_structure;
```

### hospital_beds structure tables

In the land table, there are lots of additional columns that you do not need for
your analysis. Therefore, remove them in the structure table. Because our
default table format is still Hive, you will need to either update the Catalog
or add the table format within the SQL statement demonstrated in the ```CREATE
TABLE``` query.

```sql
CREATE TABLE hb_structure (
   hospital_name VARCHAR,
   county_name VARCHAR,
   state_name VARCHAR,
   fips VARCHAR,
   num_licensed_beds INTEGER,
   num_staffed_beds INTEGER,
   num_icu_beds INTEGER,
   potential_increase_in_bed_capac INTEGER
)
WITH
    (TYPE = 'iceberg',
     FORMAT = 'ORC');
```

Now that the table is created, run the insert query that can be periodically
scheduled as a batch process to insert new data.

```sql
INSERT INTO hb_structure
SELECT
   hospital_name,
   county_name,
   state_name,
   fips,
   num_licensed_beds,
   num_staffed_beds,
   num_icu_beds,
   potential_increase_in_bed_capac
FROM
    hb_land;
```

Run a select all command to view your results:
```sql
SELECT * FROM hb_structure;
```
## Create the consume layer

The consume layer holds tables that are ready to be queried and analyzed. In
some cases, it is wise to only allow certain personas access to these final
tables. You will follow the recommended data governance and create the requested
consume layer for your data analyst. Joining the two tables in the staging
layer, create a table to view the case count and the hospital capacity in the
United States. As with the structure tables, first create the consume layer
using the ORC file format and the Iceberg table format for faster querying.
Next, run the ```INSERT``` queries that can be periodically executed to pull
data from the landing tables, formatted, and then inserted into the structure
tables using {{site.terms.sg}} for batch processing in combination with your
favorite scheduler such as Airflow or Dagster.

Create a schema to hold the consume table which points to the S3 bucket
location previously created in the configuration
section.

```sql
CREATE SCHEMA consume WITH (location='<s3_location>/consume/');

--- example statement:
--- CREATE SCHEMA consume WITH (location='s3://mm-covid-lab/consume/');
```

Now, in the top right corner, select your cluster, catalog, and schema so that
you can easily run the lab queries.

{% include image.html
  url='../../assets/img/galaxy/land-schema.png'
  img-id='data-lakehouse-reporting-structure'
  alt-text='Image displaying a data lake reporting structure'
  descr-text='Image displaying a data lake reporting structure'
  screenshot='true'
%}

### Create the consume table

Evaluate each State's current cases versus the number of licensed
beds during May 30, 2020. You can see for each state if the state is running
close to hospital capacity, or if they are managing with the current number of
infections.
```sql
CREATE TABLE states_consume (
    province_state VARCHAR,
    total_confirmed_cases INTEGER,
    num_licensed_beds INTEGER,
    num_staffed_beds INTEGER,
    num_icu_beds INTEGER
)
WITH
    (TYPE = 'iceberg',
     FORMAT = 'ORC');
```

Now that the table is created, run an insert query to populate the consume table
with the latest data.

```sql
INSERT INTO states_consume
SELECT
    province_state,
    total_confirmed_cases,
    sum(num_licensed_beds) as num_licensed_beds,
    sum(num_staffed_beds) as num_staffed_beds,
    sum(num_icu_beds) as num_icu_beds
FROM
    hb_structure beds,
    (
        SELECT
            province_state,
            sum(confirmed) as total_confirmed_cases,
            last_update
        FROM
            ejhu_structure
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

Run a select all command to view your results:

```sql
SELECT * FROM states;
```

## Configure role-based access control

To properly implement the correct permissions, configure role-based access
control with {{site.terms.sg}}.

1. Navigate to the ```Access control - Roles and Privileges``` tab.
2. Click the ```Add role``` button to create the new role.
   - Name the role: ```consume_layer_access```
   - Add a description

3. Click on the newly created role to add additional privileges by navigating to
   the ```Privileges``` tab.
4. Select ```Add privilege```
5. Choose to modify privileges for ```table```.
6. Add the catalog containing the consume layer.
7. Specify the consume layer schema.
8. Choose ```Select from table``` access for all tables in the consume schema.




### Query the consume tables

- flip role
- run the queries (do this after running the example lab)
