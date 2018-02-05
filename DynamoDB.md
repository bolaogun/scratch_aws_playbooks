# DynamoDB (DDB): A (fairly) Terse Introduction

DynamoDB is a (key/value) NoSQL database like redis.

Items (contents of Tables) may have a Time-to-live (TTL) property attached to them. DDB  offers the auto-deletion of expired Items. 
It also offers the auto-distribution of Data & Load.
It offers a predictable response time (regardless of volume of stored data)

You may read more about NoSQL databases [here](https://blog.pandorafms.org/nosql-databases-the-definitive-guide/)

## Core Components of DDB
1. Tables
1. Items
1. Attributes
1. Primary Key
1. Secondary indices
1. Streams

Some of the above are self explanatory. Some others are:

### Secondary Indices
There are 2 types of secondary indices offered by DDB:
1. Global Secondary Indices (GSI). Where both the Partition key & Sort key are different to those on the underlying table
1. Local Secondary Indices (LSI). Where the Same Partition key as the table is used along with a different sort key (to that on the table.)


### DynamoDB Streams
These capture modification events within DDB tables. Inserts/Updates and Deletes all create a Stream record when they occur. Stream records have a 24hour lifespan. A combination of Streams and Lambda could be used to create Triggers.

## DynamoDB API
There are 3 categories of APIs:

1. Control Plane APIs: Some APIs in this category are
    * CreateTable
    * DescribeTable
    * ListTables
    * UpdateTable
    * DeleteTable
1. Data Plane APIs: Further subdivided into **C**reate**R**etrieve**U**pdate**D**elete
    * PutItem (**C**)
    * BatchWriteItem (**C**)
    * GetItem (**R**)
    * BatchGetItem (**R**)
    * Query (**R**)
    * Scan (**R**)
    * UpdateItem (**U**)
    * DeleteItem (**D**)
    * BatchWrite (**D**). Limited to 25 Items at a time.
1. Streams Plane:
    * ListStrams
    * DescribeStream
    * GetSharedIterators
    * GetRecords (get streams records using a shared iterator)

## Data Types
There are 3 categories of types:
1. Scalar
    * Number
    * String
    * Binary
    * Boolean
    * Null
1. Document
    * List [..., ..., ...]
    * Map {   }
1. Set []. A List comprising of unique members. **How does one distinguish between a set and an incidental List comprising of Unique members?**

## Read Consistency
1. Eventual Consistent Reads (the default)
1. Strong Consistent Reads

are both supported.

## Throughput Capacity for Read and Writes
Capacity requirements for both Reads and Writes to a Table are specified at Index Creation Time. The measure used are called:
1. Read Capacity Unit
1. Write Capacity Unit

If Read or Write requests against an index exceed those that were set within the settings, DDB will throttle the request. HTTP 400 (Bad Request) along with ProvisionedThroughputExceededExceptions is returned in such a case.
 _(Question: How do you monitor throttled requests?)_

### The mechanisms used by DDB to manage throughput
1. DDB AutoScaling
1. Provisioned throughput
1. Reserved Capacity (which may be purchased in advance)

## Partitions and Data Distribution
A partition is an allocation of storage for a table, backed by SSD and auto-replicated across availability zones within a region. Partitions are auto-managed by DDB.

The location of data in a simple primary key (i.e. with a Partition Key only) is determined by an internal HASH function.
With Composite Primary keys (i.e. one with both a Partition and a Sort Key), hash values of items with the same primary key value are stored physically close to each other, ordered b the sort key.

## Data Access
Access to data within DDB is via POST requests. DDB is a web Service. Interactions are stateless.

## Authentication
Each request to DDB is accompanied by a Crypto Signature.

## Authorization
Within DDB, authorization is handled by the Identity and Access Management (IAM) Service. IAM policies are used to grant access to resources (tables). DDB also offers Fine grained Access (FGA) to individual items within Tables.

## Backup and Restoration Capabilities of DynamoDB (DDB)

As of Nov 2017, Amazon launched ["DynamoDB Backup and Restore"](https://aws.amazon.com/about-aws/whats-new/2017/11/aws-launches-amazon-dynamodb-backup-and-restore/).

The new backup feature is tagged _"On-Demand Backup"_.  Backups and Restorations may be initiated either via the Management Console or via the AWS CLI.

In summary, the properties of this new feature are:
1. The backups do not consume any throughput on the backed up table
1. The backups are created asynchronously (to other activities like reads & write) against the Table.
1. The level of granularity of the backups is the complete Table (during On-demand backups)
1. DDB backups do NOT guarantee casual consistency across table items within the backup
1. On-Demand Backups include the following:
   * Global Secondary Indexes(GSI),
   * Local Secondary Indexes,
   * Streams,
   * provisioned read and write capacity.
1. (As of the time of writing,) Backups & Restores may ONLY be performed in the same region as the source Table{s).
1. Restorations DO NOT overwrite existing tables.

Upon Restoration, GSIs are auto restored. Manual setup of the following are required after a restoration of a DDB Table.
1. Auto-scaling Policies
1. IAM policies used to enforce Table items' authorization
1. CloudWatch Metrics and Alarms setup to Monitor the Table
1. Tags against the Table
1. Table Stream Settings
1. Time to Live (TTL) settings

The scheduling of Backups and Restorations may be achieved with the aid of the AWS Lambda service.

As of Jan 2018, one may opt-in to a new _"DynamoDB Point-in-Time Restore (PITR)"_ capability of the DDB. This will (allegedly) allow one to restore one's data up to the minute for the past 35 days.

## Global Tables
These provide a fully managed (AWS) solution for deploying a multi-region, multi-master replication database.

### Monitoring Global Tables
CloudWatch may be used to monitor the behaviour and Performance of Global Tables. Metrics of interest are:
* ReplicationLatency
* PendingReplication Count

## Monitoring DynamoDB
The metrics to capture (which would give you a view of your baseline performance or whatever). For each table of interest, capture the following metrics:
1. Number of Read or Write Capacity Units consumed
1. Number of Requests that exceeded a table's provisioned Read/Write Capacity
1. Number of System Errors that occurred during access to the table
1. The Rate of Time to Live (TTL) deletions on the table

A few more metrics that may be observed (for monitoring purposes) are:
* ConditionalCheckFailedRequests
* ConsumedReadCapacityUints
* ConsumedWriteCapacityUints
* ReadThrottle Events
* SsytemErrors
* UserErrors
* ReadThrottleEvents
* WriteThrottleEvnts

Each of the above is collected by DDB on a 1 minute granularity.
Some others are collected on a 5-minute granularity.

The retention and archiving algorithm used by DDB is on a sliding scale also.


## DDB integrating with other AWS services
DDB (like many other AWS servcies), integrates with a number of other AWS services. Some of the common are:
* VPC: when utilizing the _create-vpc-endpoint_ call
* Cognito: via the use of IAM roles
* Redshift: the AWS RDBMS database
* Elastic Map Reduce (EMR): A managed Hardoop Framework
* CloudTrail: Which captures all API calls made to DDB
