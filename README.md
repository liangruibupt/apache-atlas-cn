# Atlas Demo in China region 

Apache Atlas on Amazon EMR is for the Hive metastore on Amazon EMR to provide capability for lineage, discovery, and classification. Also, you can use this solution for cataloging for AWS Regions that don’t have AWS Glue.
The original guide: [Metadata classification, lineage, and discovery using Apache Atlas on Amazon EMR](https://aws.amazon.com/blogs/big-data/metadata-classification-lineage-and-discovery-using-apache-atlas-on-amazon-emr/)
This project make the demo working in AWS China region

# Artifacts used for demo

* CloudFormation template: emr-atlas-cn.template
* Atlas installation script in China region: apache-atlas-emr-cn.sh
* End2end demo command: emr-atlas-execution-command.md


# Prepare
1. Download the artifacts used in demo

```
s3://apache-atlas-setup-on-emr/apache-atlas-1.0.0-bin.tar.gz
s3://apache-atlas-setup-on-emr/kafka_2.11-1.1.0.tgz
s3://apache-atlas-setup-on-emr/jdk-8u171-linux-x64.rpm
```

2. Create a S3 bucket in China region you plan running your demo and upload artifacts download in previous step to your step.

3. Update S3 bucket location in apache-atlas-emr-cn.sh and upload apache-atlas-emr-cn.sh to your S3 bucket
```
s3://<YOUR_BUCKET>/apache-atlas-1.0.0-bin.tar.gz
s3://<YOUR_BUCKET>/kafka_2.11-1.1.0.tgz
s3://<YOUR_BUCKET>/jdk-8u171-linux-x64.rpm
```

4. Update apache-atlas-emr-cn.sh S3 bucket location in CloudFormation template emr-atlas-cn.template
```
s3://<YOUR_BUCKET>/apache-atlas-emr-cn.sh
```

# Execution
Follow up the [guide](https://aws.amazon.com/blogs/big-data/metadata-classification-lineage-and-discovery-using-apache-atlas-on-amazon-emr/) and emr-atlas-execution-command.md

