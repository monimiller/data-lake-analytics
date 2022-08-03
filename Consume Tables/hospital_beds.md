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