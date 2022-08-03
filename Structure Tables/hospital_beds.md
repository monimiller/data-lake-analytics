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