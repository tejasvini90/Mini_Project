# Extract
Extracted orders data as orders.csv from ORACLE RDS using SQL DEVELOPER:

# Upload
Uploaded orders.csv file to S3 BUCKET using POSTMAN

# Loading/Un-Loading Snowflake

- STEPS for importing the data into Snowflake from the s3 bucket:

```sh
create or replace warehouse sf_orcl_wh with
  warehouse_size='X-SMALL'
  auto_suspend = 180
  auto_resume = true
  initially_suspended=true;

use warehouse sf_orcl_wh;

create or replace database sf_orcl;

use database sf_orcl;

CREATE or replace TABLE ORDERS
   ( ORDER_ID NUMBER,
	 CUSTOMER_ID NUMBER(6,0),
	 STATUS VARCHAR2(20 BYTE),
	SALESMAN_ID NUMBER(6,0),
	ORDER_DATE DATE
   ) ;

copy into ORDERS  
  from s3://myorcale/snowflake credentials=(aws_key='############' secret_key='###########')
  file_format = (type = csv field_optionally_enclosed_by='"' skip_header = 1 null_if = ('NULL', 'null') 
  empty_field_as_null = true 
 )
  pattern = '.*orders.csv';


select extract(year from order_date)yr,count(order_id)orders
from orders
group by extract(year from order_date)
order by yr;

CREATE or replace TABLE YR_ORDERS
as 
(select extract(year from order_date)yr,count(order_id)orders
from orders
group by extract(year from order_date)
order by yr
);
```

- Exporting QUERY OUTPUT to S3 BUCKET:

```sh
copy into s3://myorcale/snowflake/YR_ORDERS.csv
from YR_ORDERS 
file_format=(type=csv compression=None) header=True
credentials=(aws_key_id='#####################' aws_secret_key='##########################')
overwrite=true;

drop warehouse sf_orcl_wh;

drop database sf_orcl;
```


