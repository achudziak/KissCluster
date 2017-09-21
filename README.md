# Welcome to KissCluster 
The simplest cluster computing solution!
- The only no-frills HPC solution with KISS approach - one command to setup the cluster (takes 30 sec to complete)
- serverless master hosted in AWS - no expenses for hosting your master node (the cluster definition entirely fits into AWS free tier)
- cross-cloud - mix nodes from various cloud vendors: AWS, Azure (note: TODO), Google  (note: TODO)
- fully hybrid - mix'n'match on-prem and cloud nodes in your cluster (note: TODO)
- scallable - run clusters with hundreds or thousands of nodes - just edit `config.conf` file

# How to start

## Setting up your AWS cloud 

For quick start let's use AWS Ohio region.

### Set up your permissions
**Beginner?** The easiest way to configure your permissions along with the S3 bucket (the place were you store your cluster information and your computation results) is to [click this AWS Cloud Formation script](https://us-east-2.console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/create/review?templateURL=https://s3.us-east-2.amazonaws.com/szufel-public/kissRoleS3.yaml&stackName=kissc) link while being logged-in to your AWS account. Select the checkbox *I acknowledge that AWS CloudFormation might create IAM resources.*, click *Create*, wait 4 minutes and you are done. 

**Advanced?** [Here](https://raw.githubusercontent.com/pszufe/KissCluster/master/aws/kisscPolicy.json) is the JSON Policy template. You need to create the S3 bucket yourself and edit the bucket name in the policy file. Please note that the S3 bucket should be in the same region where the cluster information is stored (however, the nodes can be anywhere). Once you create the policy: if you use your own machine with `aws configure` command -- assign it to your IAM accout, if you use AWS machine create a Role of type : AWS service - EC2 and attach the role to the instance. 

### Install the software

Just joking, there is no install - just download and unzip wherever you like.

In order to start just type (we assume the current release is 0.0.4):
```bash
wget -L https://github.com/pszufe/KissCluster/archive/0.0.4.zip
uznip 0.0.4.zip
cd KissCluster-0.0.4/
```

### Create the cluster 
\[For all commands we assume that your are in KissCluster's home folder\]
```bash 
./kissc create --passwordless_ssh keyname --s3_bucket s3://kissc-data-1qlz7ow7tfqmo/ myc@us-east-2
```
This command creates a cluster named `myc` in the `us-east-2` AWS region with a passwordless SSH across the nodes (not needed for some cluster configuration, you can skip this option if you do not need it) and the S3 bucket `s3://kissc-data-1qlz7ow7tfqmo/` to store the data (update the bucket name to match your configuration). 

The software will create your cluster. However the cluster has zero nodes and the KissCluster cluster master is serverless - so there is not a single server yet. 

### Create a AWS SecurityGroup (AWS passwordless ssh nodes only)

If you need to have a passwordless SSH (required by some cluster types - e.g. [Julia parallel](https://docs.julialang.org/en/latest/manual/parallel-computing) you need to make sure that the network traffic is allowed across all your cluster nodes. On the AWS platform simply create a SecurityGroup (enable the access to it from your computer) and next edit it to enable access to this security group from within itself. See [this picture](https://github.com/pszufe/KissCluster/blob/master/manual/aws_passwordless_ssh.png) for reference.

### Add nodes to the cluster

The `kissc create` command has generated `cloud_init_node_myc.sh` file. On whichever machine (that has properly configured AWS permissions) you run it - the machine becomes a cluster node (TODO: tested only as cloud-init script on AWS Ubuntu instances). 
To see the file contents type:
```bash
cat cloud_init_node_myc.sh
```

In order to add node to the cluster on AWS:
- Select "Launch instance" and choose any Ubuntu image and hardware type that you like
- Select the IAM role created in the permissions section
- Open *Advanced details* and paste the text from the `cat cloud_init_node_myc.sh` file
- If using passwordless SSH remember to select the Security Group that you created. 

### Check if it works
To list your clusters and see how many nodes and job queues they have run
```bash
 ./kissc list us-east-2
```

To see the cluster nodes run:
```bash
 ./kissc nodes myc@us-east-2
```

### Submit a job to your cluster
This submits a "Hello world" bash job onto your cluster that will be executed 100 times. 
```bash
./kissc submit --job_command "bash program.sh" --folder sample_app_bash / --max_jobid 100 myc@us-east-2
```

To see the list of jobs submitted to your cluster and their status run:
```bash
./kissc queues myc@us-east-2
```

### Deleting a cluster
```bash
./kissc delete myc@us-east-2
```
Only the information about the cluster in DynamoDB tables is deleted. The nodes and S3 data are not affected by this command. 


### Notes:
- Each process spawned within the cluster will have the jobid number parameter (available as the last parameter appended to command) - you can use this number to adjust work for processes
- the collected results will contain standard output and standard error of your processes - all gzipped and alligned on S3 bucket - ready for further analysis. 


