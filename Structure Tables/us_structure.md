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