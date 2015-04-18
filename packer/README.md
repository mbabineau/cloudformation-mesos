## Packer

Packer (http://packer.io) is an open source tool for creating machine images.

We use Packer to create Amazon Machine Images (AMIs) from which our EC2 instances will be launched.

### Installation

Instructions here: http://www.packer.io/docs/installation.html

### Usage

To build an AMI, make sure your keys are set:
```
$ export AWS_ACCESS_KEY_ID="<your_access_key>"
$ export AWS_SECRET_ACCESS_KEY="<your_secret_key>"
```

Then, run `packer build <template>`:
```
$ packer build -var ami_prefix=<mycompany> ubuntu-14.04-mesos.json
```

Build times are typically 5-15 minutes plus another 10-20 minutes to replicate to other regions. You should see streamed output like this:
```
$ packer build -var ami_prefix=mbabineau ubuntu-14.04-mesos.json
amazon-ebs output will be in this color.

==> amazon-ebs: Inspecting the source AMI...
==> amazon-ebs: Creating temporary keypair: packer 55304cd7-343f-cbe0-7d08-3875e6dcf1d6
==> amazon-ebs: Creating temporary security group for this instance...
==> amazon-ebs: Authorizing SSH access on the temporary security group...
==> amazon-ebs: Launching a source AWS instance...
    amazon-ebs: Instance ID: i-0ad312c1
==> amazon-ebs: Waiting for instance (i-0ad312c1) to become ready...
==> amazon-ebs: Waiting for SSH to become available...
==> amazon-ebs: Connected to SSH!
==> amazon-ebs: Uploading include/ => /tmp/
==> amazon-ebs: Provisioning with shell script: /var/folders/01/8gq4dvp57bs9hlyh0dxpkr140000gn/T/packer-shell373313770
    amazon-ebs: Ign http://security.ubuntu.com trusty-security InRelease
    amazon-ebs: Get:1 http://security.ubuntu.com trusty-security Release.gpg [933 B]
[... REMOVED FOR BREVITY ...]
==> amazon-ebs: Adding tags to AMI (ami-f26ca9f2)...
    amazon-ebs: Adding tag: "os:distribution": "Ubuntu"
    amazon-ebs: Adding tag: "os:release": "14.04 LTS"
    amazon-ebs: Adding tag: "mesos:version": "0.21.1"
==> amazon-ebs: Terminating the source AWS instance...
==> amazon-ebs: Deleting temporary security group...
==> amazon-ebs: Deleting temporary keypair...
Build 'amazon-ebs' finished.

==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs: AMIs were created:

ap-northeast-1: ami-f26ca9f2
ap-southeast-1: ami-50467b02
ap-southeast-2: ami-79542943
eu-west-1: ami-99fc9cee
sa-east-1: ami-3944c124
us-east-1: ami-0a878262
us-west-1: ami-494eac0d
us-west-2: ami-734a7f43
```