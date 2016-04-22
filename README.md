# *Databricks Cloud* vs. *ec2 spot instance + zeppelin*

##Summary

HIPPA compliance requieres dedicated EC2 instances, and it seems impossible to have them with Spot pricing. Though it's still possible to use Spot instances for extremely cheap spark clusters, but only slightly cheaper Spark cluster (compared to Databrick Cloud) if HIPPA compliance is needed.

Also - last time I've tried Databricks Cloud serioudly was about 7 months ago, some things might've changed.

## Pricing of Databrick cloud
<https://databricks.com/product/pricing>

### expenses:

- Cost of EC2 machine full price
- $99 monthly fee (negligible)
- $0.40 per node

### example: c3.8xlarge per node per hour

- $0.40 (databricks) + $1.68 (AWS) = $2.08

## Pricing EC2 spot instance

<http://ec2price.com/?product=Linux/UNIX&type=c3.8xlarge&region=us-east-1&window=3>
<http://www.ec2instances.info/?filter=c3.8xlarge>

### expenses:

- spot instance price
- elastic IP (negligible)
- hosted NAT and IGW (negligible)

### example: c3.8xlarge per node per hour

- ~$0.44 (AWS)
- price is changing hourly

## Pricing EC2 full price instance

<http://www.ec2instances.info/?filter=c3.8xlarge>

### expenses:

- full instance price
- elastic IP (negligeble)
- hosted NAT and IGW (negligeble)

### Price per hour per node per hour

- $1.68 (AWS)



## Feature Comparison


| Feature  | Databricks Cloud  | Zeppelin |
|:------------- |:---------------:| -------------:|
| Fully featured web notebooks      | yes |         yes |
| inline scala, python, md, etc..     | yes        |           yes |
| pre-creation of SparkContext     | yes        |           yes |
| you own the data (use your own s3) | no        |           yes |
| dashboard creator | yes        |            no |
| 'drag-n-drop' cluster creation | yes        |            no |
| team colloboration tools    | yes        |           no |
| HIPPA compliance possible    | maybe        |           yes |

## HIPPA compliance

Both *EC2* and *S3* services which are needed to run suggested installation can be made *HIPPA* compliant according to <https://aws.amazon.com/compliance/hipaa-compliance/>. *EC2* servers need to be run on dedicated machines behind *VPN*. The steps presented in the following paragraph actually don't allow to have dedicated machines. *S3* needs to be encrypted to be compliant wiht HIPPA, there are several options to do this.

## zeppelin + ec2 spot installation instructions

this is not a proper "how to" - just some of my personal notes.

### AWS VPN (needed for HIPPA)

- create vpn on aws, created private and public subnet, route public though IGW and private though NAT server
- edit **~/.aws/config** to look like this

```
[default]
output = json
region = eu-west-1
aws_access_key_id = AK***************
aws_secret_access_key = iEX***************
```
### Starting Spark Cluster (without HIPPA)

- download spark (1.6.2 with hadoop 2.4) locally
- run following command to initiate spark cluster on ec2 with spot instance enabled

```
$SPARK_HOME/spark/ec2/spark-ec2 \
--key-pair=kaggle \
--identity-file=$LOCASTION_OF_YOUR_KEYS/ec2.pem \
--region=us-east-1 --zone=us-east-1c \
--vpc-id=YOUR_VPC_ID \
--subnet-id=YOUR_SUBNET_ID \
--instance-type=c3.8xlarge \
--master-instance-type=c3.8xlarge \
--slaves 20 \
--spot-price=0.6 \
--spark-version 1.5.2 \
--private-ips \
--hadoop-major-version=yarn\
--copy-aws-credentials \
launch test-spark-cluster
```

### Starting Spark Cluster with HIPPA

- need to configure your instances to run on a dedicated machine, and perhaps hardware VPN to your server. Using Spot instance is NOT possible

### Starting Zeppelin

- *ssh* into your master server (ip will be displayed during the last step)
- download Zeppelin source and build it locally

```
mvn clean package -DskipTests -Pspark-1.6 -Dspark.version=1.6.2 -Phadoop-2.4
```

- start your zeppelin instance and go to cluster settings, point it to your master (local-ip:7077)

Some tutorial I've found on YourTube on Zeppelin build process and feature overview

<https://www.youtube.com/watch?v=CfhYFqNyjGc>

### Persisting Notebooks

- It is possible to connect Zeppelin to S3 so your notebook changes are persisted
<https://blogs.aws.amazon.com/bigdata/post/Tx2HJD3Z74J2U8U/Running-an-External-Zeppelin-Instance-using-S3-Backed-Notebooks-with-Spark-on-Am>
