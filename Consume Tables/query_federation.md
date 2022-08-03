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