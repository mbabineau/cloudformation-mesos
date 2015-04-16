CloudFormation templates for a [Mesos](http://mesos.apache.org) cluster running the [Marathon](https://github.com/mesosphere/marathon) framework.

Prerequisites:
* An Exhibitor-managed ZooKeeper cluster such as provided by [mbabineau/cloudformation-zookeeper](https://github.com/mbabineau/cloudformation-zookeeper). Specifically, you'll need:
    - An Exhibitor endpoint for ZK node discovery
    - A ZK client security group to associate with the Mesos nodes

## Overview

This project includes three templates:
* `mesos-master.json` - Launch a set of Mesos masters running Marathon in an auto scaling group
* `mesos-slave.json` - Launch a set of Mesos slaves in an auto scaling group
* `mesos.json` - Creates both a `mesos-master` and `mesos-slave` stack from the corresponding templates.

In general, you'll want to launch the Mesos cluster via `mesos.json`.

Mesos servers are launched from public AMIs running Ubuntu 14.04 LTS and pre-loaded with Docker, Runit, and Mesos. If you wish to use your own image, simply modify `RegionMap` in `mesos.json`.

Marathon is run on the masters via a Docker image specified as a Parameter. You can use the default or provide your own.

To adjust cluster capacity, simply increment or decrement the slave auto scaling group. You can do the same for the masters. Node addition/removal should be handled transparently by Mesos.

Mesos uses ZooKeeper for coordination, so the templates expect a security group (granting ZK access) and a ZK node discovery URL (exposed by Exhibitor) to be passed in.

Note that this template must be used with Amazon VPC. New AWS accounts automatically use VPC, but if you have an old account and are still using EC2-Classic, you'll need to modify this template or make the switch.

## Usage

### 1. Clone the repository
```bash
git clone git@github.com:mbabineau/cloudformation-mesos.git
```

### 2. Create an Admin security group
This is a VPC security group containing access rules for cluster administration, and should be locked down to your IP range, a bastion host, or similar. This security group will be associated with the Mesos servers.

Inbound rules are at your discretion, but you may want to include access to:
* `22 [tcp]` - SSH port
* `5050 [tcp]` - Mesos Master port
* `8080 [tcp]` - Marathon port

### 3. Set up ZooKeeper
You can use the instructions and template at [mbabineau/cloudformation-zookeeper](https://github.com/mbabineau/cloudformation-zookeeper), or you can use an existing cluster.

We'll need two things:
* `ExhibitorDiscoveryUrl` - a URL that returns a list of active ZK nodes. The expected format is that of Exhibitor's `/cluster/list` call. Example response:
```
{
    "servers": [
        "zk1.mydomain.com",
        "zk2.mydomain.com",
        "zk3.mydomain.com"
    ],
    "port": 2181
}
```
* `ZkClientSecurityGroup` - a security group that grants access to the ZooKeeper servers

If you used the aforementioned template, you can simply copy the stack's outputs.

### 4. Upload the templates to S3

Upload `mesos-master.json` and `mesos-slave.json` to an S3 bucket:
```bash
aws s3 cp mesos-master.json s3://mybucket/
aws s3 cp mesos-slave.json s3://mybucket/
```

You'll need to pass the URLs for each as stack parameters. Note the URL should be formatted as `https://s3.amazonaws.com/<bucket>/<key>`, and the files do not need to be made public.

### 5. Launch the stack
Launch the stack via the AWS console, a script, or [aws-cli](https://github.com/aws/aws-cli).

See `mesos.json` for the full list of parameters, descriptions, and default values.

Example using `aws-cli`:
```bash
aws cloudformation create-stack \
    --template-body file://mesos.json \
    --stack-name <stack> \
    --capabilities CAPABILITY_IAM \
    --parameters \
        ParameterKey=KeyName,ParameterValue=<key> \
        ParameterKey=ExhibitorDiscoveryUrl,ParameterValue=<url> \
        ParameterKey=ZkClientSecurityGroup,ParameterValue=<sg_id> \
        ParameterKey=VpcId,ParameterValue=<vpc_id> \
        ParameterKey=Subnets,ParameterValue='<subnet_id_1>\,<subnet_id_2>' \
        ParameterKey=AdminSecurityGroup,ParameterValue=<sg_id> \
        ParameterKey=MesosMasterTemplateUrl,ParameterValue=<url> \
        ParameterKey=MesosSlaveTemplateUrl,ParameterValue=<url>
```

### 4. Watch the cluster converge
Once the stack has been provisioned, visit the public-facing ELB created by the stack. You can find the DNS address by checking the stack's `Outputs`.

The ELB exposes two endpoints:
* `http://<public-elb>:5050/` for Mesos
* `http://<public-elb>:8080/` for Marathon

_Note: You will need to do this from a location granted access by the specified `AdminSecurityGroup`_
