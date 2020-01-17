# Partition

Range Partitioning Tables

Hash Partitioning Tables

List Partitioning Tables

Composite Partitioning Tables

Partitioning Indexes

Local Prefixed Indexes

Local Non-Prefixed Indexes

Global Prefixed Indexes

Global Non-Prefixed Indexes

Partitioning Existing Tables

# Virtual Column-Based Partitioning

These virtual columns are not physically stored in the table, but derived from data in the table.
These virtual columns can be used in the partition key in all basic partitioning schemes

REATE TABLE users (
  id           NUMBER,
  username     VARCHAR2(20),
  first_letter VARCHAR2(1)
    GENERATED ALWAYS AS
      (
        UPPER(SUBSTR(TRIM(username), 1, 1))
      ) VIRTUAL
)
PARTITION BY LIST (first_letter)
(
  PARTITION part_a_g VALUES ('A','B','C','D','E','F','G'),
  PARTITION part_h_n VALUES ('H','I','J','K','L','M','N'),
  PARTITION part_o_u VALUES ('O','P','Q','R','S','T','U'),
  PARTITION part_v_z VALUES ('V','W','X','Y','Z')
);

# Interval partitioning

Interval-Hash

Interval-List

Interval-Range

CREATE TABLE interval_tab (
  id           NUMBER,
  code         VARCHAR2(10),
  description  VARCHAR2(50),
  created_date DATE
)
PARTITION BY RANGE (created_date)
INTERVAL (NUMTOYMINTERVAL(1,'MONTH'))
(
   PARTITION part_01 values LESS THAN (TO_DATE('01-NOV-2007','DD-MON-YYYY'))
);



# Split Partition



# Cascade Functionality in partition

Using on cascade delete option it will work on hiererachy basis.

TRUNCATE PARTITION 
EXCHANGE PARTITION  





# Partition Exchange

#Syntax
alter table table name exchange partition partition name with table tablename;

If the table or partitioned table involved in the exchange operation has a primary key or unique constraint enabled, then the exchange operation will be performed as if WITH VALIDATION were specified in order to maintain the integrity of the constraints. 
The validation process can slow down the operation.

# Partition Index

Local: Every partition have separate index if lcoal key word mentioned.indexes doesn't become UNUSABLE after Exchange

Global: Better to use in 1 or 2 partition and having less data.This need everytime index rebuild.

# Ref Exchange Partition
https://www.akadia.com/services/ora_exchange_partition.html
