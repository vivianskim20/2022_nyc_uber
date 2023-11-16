create or replace table NYC_UBER.PUBLIC.fct_trips AS 

    with week_nums as (
        select
            date,
            case when left(date,4) = '2021' then 0 else WEEKOFYEAR(date) end as week_num,
            price
        from gas_prices
    )
    
    select
        request_datetime,
        pulocationid,
        dolocationid,
        trip_miles,
        trip_time,
        base_passenger_fare,
        tolls,
        sales_tax,
        shared_request_flag,
        shared_match_flag,
        bcf + congestion_surcharge + airport_fee as surcharges,
        tips,
        driver_pay,
        price as weekly_gas_price
    from tripdata_all t1
    left join week_nums t2
        on (case when t1.request_datetime < '2022-01-03' then 0 else WEEKOFYEAR(t1.request_datetime) end) = t2.week_num;
    
select
    *
from fct_trips;

-------------------------------------

create or replace table NYC_UBER.PUBLIC.dim_request_datetime AS 

    select 
        distinct request_datetime,
        case 
            when month(request_datetime) = 1 then 'Jan.'
            when month(request_datetime) = 2 then 'Feb.'
            when month(request_datetime) = 3 then 'Mar.'
            when month(request_datetime) = 4 then 'Apr.'
            when month(request_datetime) = 5 then 'May.'
            when month(request_datetime) = 6 then 'Jun.'
            when month(request_datetime) = 7 then 'Jul.'
            when month(request_datetime) = 8 then 'Aug.'
            when month(request_datetime) = 9 then 'Sep.'
            when month(request_datetime) = 10 then 'Oct.'
            when month(request_datetime) = 11 then 'Nov.'
            else 'Dec.'
        end as request_month,
        left(request_datetime,10) as request_date,
        left(date_trunc('hour',request_datetime),19) as request_hour
    from tripdata_all;

select
    request_datetime,
    request_date
from dim_request_datetime
order by 1 
limit 100;

-------------------------------------

create or replace table NYC_UBER.PUBLIC.dim_pickup_loc AS 
    
    with taxi_zone_dedup as (
        select 
            locationid,
            zone_borough,
            -- multiple lat/lon of Corona, Queens and Governor's Island/Ellis Island/Liberty Island, Manhattan
            -- with unique LocationID's, so deduped by averaging lat/lon and keeping unique LocationID's
            avg(latitude) over (partition by zone_borough) as latitude,
            avg(longitude) over (partition by zone_borough) as longitude
        from taxi_zones
    )
    
    select
        distinct t1.pulocationid,
        zone_borough,
        latitude,
        longitude
    from tripdata_all t1
    join taxi_zone_dedup t2
        on t1.pulocationid = t2.locationid;

select *
from dim_pickup_loc
order by pulocationid;

-------------------------------------

create or replace table NYC_UBER.PUBLIC.dim_dropoff_loc AS 
    
    with taxi_zone_dedup as (
        select 
            locationid,
            zone_borough,
            -- multiple lat/lon of Corona, Queens and Governor's Island/Ellis Island/Liberty Island, Manhattan
            -- with unique LocationID's, so deduped by averaging lat/lon and keeping unique LocationID's
            avg(latitude) over (partition by zone_borough) as latitude,
            avg(longitude) over (partition by zone_borough) as longitude
        from taxi_zones
    )
    
    select
        distinct t1.dolocationid,
        zone_borough,
        latitude,
        longitude
    from tripdata_all t1
    join taxi_zone_dedup t2
        on t1.dolocationid = t2.locationid;

select *
from dim_dropoff_loc
order by dolocationid;
