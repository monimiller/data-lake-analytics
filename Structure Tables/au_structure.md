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