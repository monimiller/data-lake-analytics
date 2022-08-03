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