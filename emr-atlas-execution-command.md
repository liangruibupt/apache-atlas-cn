# Background

Apache Atlas on Amazon EMR leverage the Hive metastore on Amazon EMR to provide capability for lineage, discovery, and classification. 
Apache Atlas uses Apache Solr for search functions and Apache HBase for storage. 
Supports both internal and external Hive tables. 
For the Hive metastore to persist across multiple Amazon EMR clusters, you should use an external Amazon RDS or Amazon Aurora database to contain the metastore.

# For china region resources
```
# Download artifacts
mkdir -p <MY_EMR_ALTAS>
cd <MY_EMR_ALTAS>
aws s3 cp s3://apache-atlas-setup-on-emr/apache-atlas-1.0.0-bin.tar.gz . --region us-east-1 --profile us-east-1
s3://apache-atlas-setup-on-emr/kafka_2.11-1.1.0.tgz . --region us-east-1 --profile us-east-1
s3://apache-atlas-setup-on-emr/jdk-8u171-linux-x64.rpm . --region us-east-1 --profile us-east-1

# Upload to China region S3 bucket, please replace your S3 bucket
aws s3 sync . s3://ray-datalake-lab/emr-atlas/ --region cn-north-1 --profile cn-north-1
## Or you can copy it one by one
aws s3 cp apache-atlas-1.0.0-bin.tar.gz s3://ray-datalake-lab/emr-atlas/apache-atlas-1.0.0-bin.tar.gz --region cn-north-1 --profile cn-north-1
aws s3 cp kafka_2.11-1.1.0.tgz s3://ray-datalake-lab/emr-atlas/kafka_2.11-1.1.0.tgz --region cn-north-1 --profile cn-north-1
aws s3 cp jdk-8u171-linux-x64.rpm s3://ray-datalake-lab/emr-atlas/jdk-8u171-linux-x64.rpm --region cn-north-1 --profile cn-north-1
```

# Running EMR with Atlas

## Create EMR cluster with Atlas
**Please modify the command based on your account environment**

## Option 1 Create the EMR cluster with Atlas Installation Job
```
aws emr create-cluster --name EMR-Atlas-Cluster --region cn-north-1 --profile cn-north-1 --applications Name=Hive Name=HBase Name=Hue Name=Hadoop Name=ZooKeeper \
  --tags Name="EMR-Atlas-Cluster" \
  --release-label emr-5.26.0 \
  --ec2-attributes SubnetId=subnet-04e3699653a449baa,KeyName=ruiliang-lab-key-pair-cn-north1,InstanceProfile=EMR_EC2_DefaultRole \
  --service-role EMR_DefaultRole \
  --ebs-root-volume-size 100 \
  --instance-groups InstanceGroupType=MASTER,InstanceCount=1,InstanceType=m4.2xlarge InstanceGroupType=CORE,InstanceCount=2,InstanceType=m4.2xlarge \
  --emrfs Consistent=true,RetryCount=5,RetryPeriod=30,Args=[fs.s3.consistent.metadata.read.capacity=600,fs.s3.consistent.metadata.write.capacity=300] \
  --log-uri 's3://ray-emr-lab-03242151/logs' \
  --steps Name='Run Remote Script',Jar=command-runner.jar,Args=[bash,-c,'aws s3 cp s3://ray-datalake-lab/emr-atlas/apache-atlas-emr-cn.sh /tmp/script.sh; chmod +x /tmp/script.sh; /tmp/script.sh']
{
    "ClusterId": "j-1BUEK128UYEO9"
}
aws emr describe-cluster --cluster-id j-1BUEK128UYEO9 --query 'Cluster.Status.State' --region cn-north-1 --profile cn-north-1
```

## Option 2 Create the EMR cluster without Atlas Installation Job and manually install Atlas

```
aws emr create-cluster --name EMR-Atlas --region cn-north-1 --profile cn-north-1 --applications Name=Hive Name=HBase Name=Hue Name=Hadoop Name=ZooKeeper \
  --tags Name="EMR-Atlas" \
  --release-label emr-5.26.0 \
  --ec2-attributes SubnetId=subnet-04e3699653a449baa,KeyName=ruiliang-lab-key-pair-cn-north1,InstanceProfile=EMR_EC2_DefaultRole \
  --service-role EMR_DefaultRole \
  --ebs-root-volume-size 100 \
  --instance-groups InstanceGroupType=MASTER,InstanceCount=1,InstanceType=m4.2xlarge InstanceGroupType=CORE,InstanceCount=2,InstanceType=m4.2xlarge \
  --emrfs Consistent=true,RetryCount=5,RetryPeriod=30,Args=[fs.s3.consistent.metadata.read.capacity=600,fs.s3.consistent.metadata.write.capacity=300] \
  --log-uri 's3://ray-emr-lab-03242151/logs'
{
    "ClusterId": "j-20FHMXYAH6R6X"
}
aws emr describe-cluster --cluster-id j-20FHMXYAH6R6X --query 'Cluster.Status.State' --region cn-north-1 --profile cn-north-1
Then login EMR master and running apache-atlas-emr-cn.sh
```

## Create Hive tables
Using bastion/jump server VM to login the EMR master and access the Web-UI. **Please modify the command based on your account environment**

```
$ hive
hive> create database atlas_emr;
hive> use atlas_emr;

hive> CREATE external TABLE trip_details
(
  pickup_date        string ,
  pickup_time        string ,
  location_id        int ,
  trip_time_in_secs  int ,
  trip_number        int ,
  dispatching_app    string ,
  affiliated_app     string 
)
row format delimited
fields terminated by ',' stored as textfile
LOCATION 's3://ray-datalake-lab/emr-atlas/trip_details/';

hive> CREATE external TABLE trip_zone_lookup 
(
LocationID     int ,
Borough        string ,
Zone           string ,
service_zone   string
)
row format delimited
fields terminated by ',' stored as textfile
LOCATION 's3://ray-datalake-lab/emr-atlas/zone_lookup/';

hive> create table trip_details_by_zone as select *  from trip_details  join trip_zone_lookup on LocationID = location_id;

hive> exit;

/apache/atlas/bin/import-hive.sh   admin/admin
Hive Meta Data imported successfully!!!
```

Or you can use the Hue
```
http://<master_IP>:8888/  # such as http://ip-10-0-3-176.cn-north-1.compute.internal:8888/
```

## Use the atlas
```
# Login Atlas
http://<master_IP>:21000/  # such as http://ip-10-0-3-176.cn-north-1.compute.internal:21000/
admin/admin

- View the data catalog of your Hive tables using Atlas
Search By Type: hive_table
Search By Text: trip_details

- View the data lineage of your Hive tables using Atlas
Search By Type: hive_table
Search By Text: trip_details_by_zone

- Create a classification for metadata management
create classification PII for trip_zone_lookup

- Discover metadata using the Atlas domain-specific language (DSL)
from hive_table where name = trip_details
from hive_column where name = 'location_id'
hive_table select count()
```

