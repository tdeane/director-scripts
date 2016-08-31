# ETL Transient on AWS with CE 5.8

These scripts can be used as an example end-to-end transient demo for a Hive query with C5.8 running on AWS.  

## Instructions

- Configure AWS settings: http://www.cloudera.com/documentation/director/latest/topics/director_aws_setup_client.html
- Launch Director Instance (this script was tested with RHEL 7.2): http://www.cloudera.com/documentation/director/latest/topics/director_deployment_start_launcher.html
- Copy all files in the transient-aws folder to the Director instance
```sh
scp -r -i ~/.ssh/my_aws_key.pem director-scripts/transient-aws ec2-user@[public-ip-address]:/home/ec2-user/
```
- Copy your AWS SSH private key to the instance's  ~/.ssh/id_rsa (we will use this in the last step)
```sh
scp -i ~/.ssh/my_aws_key.pem ~/.ssh/my_aws_key.pem ec2-user@[public-ip-address]:/home/ec2-user/.ssh/id_rsa
```
- SSH into Director instance 

#### All following steps are to be executed in the Director instance

- Add S3 keys in the REPLACE_ME sections of install_director.sh (only needed if copying log files to S3)
- Install Director and other packages: 

```sh
./install_director.sh
```

### Prepare AMIs
- Open preloaded-ami-builder/packer-json/rhel.json
- Replace all REPLACE_ME's with the correct values for your AWS setup.
  Note: this script was tested in us-west-2 with Centos 7.2 base AMI images. If you are using us-west-2 you can keep the example AMI IDs, SSH username, and root device name in all files.
- Run the AMI builder with the following command.  Replace the region, base AMI ID, and target AMI name if neccessary.
```sh
cd preloaded-ami-builder
./build-ami.sh us-west-2 ami-d2c924b2 "cdh58-centos7-ami"
```

### Update Cluster Configuration File
- Open hive-example/cluster_preloaded_amis.conf 
- Replace all REPLACE_ME's with the correct values for your AWS setup.
  Note:  make sure to use your newly created AMI ID for all the AMI IDs
- Replace the SSH_USERNAME in hive-example/dispatch.sh to match the AMI SSH username
- Replace the 'centos' username in hive-example/hive_job.sh to match the AMI SSH username
- If not using preloaded AMIs, use hive-example/cluster.conf instead and update hive/run_all.sh to point to cluster.conf.

### Prepare the Hive Query
- Open hive-example/query.sql and update the query or use the sample one
- If using the sample one, make sure to replace all s3 locations with your own S3 buckets (note that Cloudera will eventually provide a public read-only bucket with sample data)

### Set S3 output log file
- Open hive-example/dispatch.sh and update the REPLACE_ME section for the S3 location to store the log files

### Run transient job
```sh
cd hive-example/
./run_all.sh
```
