```sql
create table
    ejhu_consume_us as
select
    province_state,
    sum(confirmed) as confirmed_cases,
    sum(active) as active_cases
from
    covid.ejhu_stg_us
where
    last_update = '2020-05-30T02:32:48'
group by
    province_state;
```