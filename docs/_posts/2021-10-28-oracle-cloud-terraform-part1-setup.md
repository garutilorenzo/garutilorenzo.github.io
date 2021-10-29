---
layout: post
title: Setup Terraform Oracle Cloud provider
description: Setp terraform Oracle Cloud provider, this is the first post of a series dedicated to Oracle Cloud infrastructure and Terraform.
categories: [Oracle,Terraform,Cloud]
---

This is the first post of a series dedicated to Oracle Cloud infrastructure and Terraform.

### Oracle Cloud series index

* Setup Terraform Oracle Cloud provider
* Deploy an Oracle Cloud compute instance using terraform. Go to [part 2]({{ site.baseurl }}/oracle-cloud-terraform-part2-simple-instance)
* Deploy multiple Oracle Cloud compute instances using an instance pool using terraform Go to [part 3]({{ site.baseurl }}/oracle-cloud-terraform-part3-instance-pool)
* Deploy a k3s cluster on Oracle Cloud using terraform [part 4]({{ site.baseurl }}/oracle-cloud-terraform-part4-k3s-cluster)

## Sign-up to Oracle Cloud

Go to [https://cloud.oracle.com/](https://cloud.oracle.com/) and create a new account:

![Oracle Signup Page]({{ site.baseurl }}/images/oracle-sign-up.png)

## Account setup

Once you are logged in we need to create a new user and a new group with limited grants. To do so go to Identity & Security -> Identity:

![Oracle Identity]({{ site.baseurl }}/images/oracle-identity.png)

Under Group, create a new group called terraform:

![Oracle Group]({{ site.baseurl }}/images/oracle-group.png)

click "Create"

Under Policys, create a new policy named terraform-policy

![Oracle Policy]({{ site.baseurl }}/images/oracle-policy.png)

set the description to "terraform-users-policy", click on show manual editor and paste this lines:

```
Allow group terraform to manage virtual-network-family in tenancy
Allow group terraform to manage instance-family in tenancy
Allow group terraform to manage compute-management-family in tenancy
Allow group terraform to manage volume-family in tenancy
```

This policy could be too open, take it only as example for this tutorial. You can get more details [here](https://docs.oracle.com/en-us/iaas/Content/Identity/Concepts/policysyntax.htm#three)

Now create a new user, under user create a new user called terraform:

![Oracle User]({{ site.baseurl }}/images/oracle-user.png)

Choose IAM user, set the name to terraform and the description to "terraform user".

Now in the username details (click on terraform in the User table), click "Edit user capabilites" and un-check:

* Local Password
* SMTP credentials
* Customer Secret Keys
* OAuth 2.0 Client Credentials

![Oracle User]({{ site.baseurl }}/images/oracle-user-capabilities.png)

Now click "Add User to Group" and choose the terraform group.

![Oracle user detail]({{ site.baseurl }}/images/oracle-user-group.png)

### RSA key generation

To use terraform with the Oracle Cloud infrastructure you need to generate an RSA key. Generate the rsa key with:

```
openssl genrsa -out ~/.oci/terraform-oracle-cloud.pem 4096
chmod 600 ~/.oci/terraform-oracle-cloud.pem
openssl rsa -pubout -in ~/.oci/terraform-oracle-cloud.pem -out ~/.oci/terraform-oracle-cloud_public.pem
```

**NOTE** ~/.oci/terraform-oracle-cloud_public.pem this string will be used on the *terraform.tfvars* used by the Oracle provider plugin, so please take note of this string.

Now copy the content of  ~/.oci/terraform-oracle-cloud_public.pem:

```
cat  ~/.oci/terraform-oracle-cloud_public.pem
```

In the Oracle Cloud Console, under user select the terraform user. Under API Keys click on "Add API Key" and paste the content of your public RSA key:

![Oracle user detail]({{ site.baseurl }}/images/oracle-user-key.png)

Now you should see your configuration details:

```
[DEFAULT]
user=<user ocid...>
fingerprint=<fingerprint..>
tenancy=<tenecny ocid...>
region=<region>
key_file=<path to your private keyfile> # ~/.oci/terraform-oracle-cloud_public.pem
```

### Terraform setup

The first step is to [install](https://learn.hashicorp.com/tutorials/terraform/install-cli) terraform. Once terraform is installed download [this](https://github.com/garutilorenzo/oracle-cloud-terraform-examples) repository:

```
git clone https://github.com/garutilorenzo/oracle-cloud-terraform-examples.git
cd oracle-cloud-terraform-examples/
```

In the root dir of this repository yoi will find three subdirectory, now we move into the simple-instance directory:

```
cd simple-instance/
```

on this directory we find all the necessary files for deploying our first instance, we see more in detail in the [next]({{ site.baseurl }}/oracle-cloud-terraform-part2-simple-instance) session.

Now in this directory we have to create a file named "terraform.tfvars":

```
touch terraform.tfvars
```

edit the file and paste your configuration details, the file will look like:

```
fingerprint      = <fingerprint..>
private_key_path = "~/.oci/terraform-oracle-cloud.pem"
user_ocid        = "<user ocid...>"
tenancy_ocid     = <tenecny ocid...>
compartment_ocid = <compartment ocid...>
```

**NOTE** The compartment_ocid is the same as tenency_ocid.

Now we have setup the terraform Oracle provider and we are ready for our [first]({{ site.baseurl }}/oracle-cloud-terraform-part2-simple-instance) deployment.