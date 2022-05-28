# Systems and services built on top of postgreSQL


It is surprised to come across several systems/services that happen to built on top of postgreSQL. Here is a quick summary of these. 

| Systems/services | Type |
| :------: | :--: |
| AWS Redshift     | data warehousing | 
| TimeScale     | time series database |
| openGauss/GaussDB     | distributed relational db, by Huawei |

* AWS Redshift: Redshift is a data warehousing service offered by AWS. It is built on postgreSQL, with a new columnar storage engine. The use of columnar storage is expected to provide better query performance and better compression. 

* TimeScale: TimeScale DB is a new time series database. It basically is a cluster of postgreSQL databases.   Data is partitioned based on data source and time intervals to facilitate faster query processing, with each query accessing local data as much as possible.   
