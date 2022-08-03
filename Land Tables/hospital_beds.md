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
   longtitude DOUBLE
)
WITH (
   format = 'json',
   EXTERNAL_LOCATION = 's3://covid19-lake/rearc-usa-hospital-beds/json/')
;
```
