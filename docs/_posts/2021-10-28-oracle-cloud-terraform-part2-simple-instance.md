---
layout: post
title: Deploy an Oracle Cloud compute instance using terraform
description: Deploy an Oracle Cloud compute instance using terraform and oracle cloud provider plugin
categories: [Oracle,Terraform,Cloud]
---

Welcome  to the second part of the series dedicated to the Oracle cloud infrastructure and terraform, if you have missed it you can read the first part [here]({{ site.baseurl }}/oracle-cloud-terraform-part1-setup). 

Once we have [setup]({{ site.baseurl }}/oracle-cloud-terraform-part1-setup) our terraform Oracle cloud provider, we are now ready to deploy our first instance. 

### Clone the repository

I you haven't already done so, download this [this](https://github.com/garutilorenzo/oracle-cloud-terraform-examples) repository, and enter in the simple-instance directory:

```
git clone https://github.com/garutilorenzo/oracle-cloud-terraform-examples.git
cd oracle-cloud-terraform-examples/simple-instance/
```

### Infrastructure overview

This example will deploy:

* one VCN network (cidr 10.0.0.0/16), you can customize the cidr changing the oci_core_vcn_cidr variable (see Variable setup)
* two subnets (cidr 10.0.0.0/24 and cidr 10.0.1.0/24). you can customize the subnets cidr changing oci_core_subnet_cidr10 and oci_core_subnet_cidr11 variables (see Variable setup)
* one internet gateway associated with the VCN network
* one route table associated with the VCN network
* one security list (see notes about security list)
* one Ocacle compute instance, VM.Standard.A1.Flex with 6GB of ram and 1 CPU (ARM processor)

**notes about security** the security list rules are:

* By default only the incoming ICMP, SSH and HTTP traffic is allowed from your public ip. You can setup your public ip in my_public_ip_address variable.
* By default all the outgoing traffic is allowed
* A second security list rule (Custom security list) open all the incoming http traffic
* Both default security list and the custom security list are associated on both subnets
* Network flow from the private VCN subnet is allowed

### Notes

Some important notes:

* By default firewall on the compute instances is disabled. On some test the firewall has created some problems
* Nginx will be installed by default (nginx is used for testing the security list rules)
* The operating system used is Ubuntu 20.04

### Variables setup

Before we can proceed we have to modify some variables in the vars.tf, the variables to modify are:

| Variable | Default | Description |
| -------- |  ------- | ----------- |
| `region` | `N.D.`      | Set the correct region based on your needs |
| `availability_domain` |  `N.D.`    |  set you availability domain, you can get the availability domain string in the "*Create instance* form. Once you are in the create instance procedure under the placement section click "Edit" and copy the string that begin with *iAdc:*. Example iAdc:EU-ZURICH-1-AD-1 |
| `default_fault_domain` | `FAULT-DOMAIN-1`     | set de default fault domain, choose one of: FAULT-DOMAIN-1, FAULT-DOMAIN-2, FAULT-DOMAIN-3 |
| `PATH_TO_PUBLIC_KEY` | `~/.ssh/id_rsa.pub`     | this variable have to point at your ssh public key |
| `oci_core_vcn_cidr` | `10.0.0.0/16`     | set the default VCN subnet cidr  |
| `oci_core_subnet_cidr10` | `10.0.0.0/24`     |set the default subnet cidr |
| `oci_core_subnet_cidr11` | `10.0.1.0/24`     | set the secondary subnet cidr |
| `tutorial_tag_key` | `oracle-tutorial`     | set a key used to tag all the deployed resources |
| `tutorial_tag_value` | `terraform`     | set the value of the tutorial_tag_key |
| `my_public_ip_address` | `N.D.`     |  set your public ip address |

**NOTE** this variables have to be set also in the [part3]({{ site.baseurl }}/oracle-cloud-terraform-part3-instance-pool) and [part4]({{ site.baseurl }}/oracle-cloud-terraform-part4-k3s-cluster), remember to modify the vars.tf files also in this directories.

### Deploy

Now [create]({{ site.baseurl }}/oracle-cloud-terraform-part1-setup) the terraform.tfvars file (Terraform setup section), and initialize terraform:

```
terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/oci...
- Installing hashicorp/oci v4.50.0...
- Installed hashicorp/oci v4.50.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

we are now ready to launch our first instance:

```
terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # oci_core_default_route_table.default_oci_core_default_route_table will be created
  + resource "oci_core_default_route_table" "default_oci_core_default_route_table" {
      + compartment_id             = (known after apply)
      + defined_tags               = (known after apply)
      + display_name               = (known after apply)
      + freeform_tags              = (known after apply)
      + id                         = (known after apply)
      + manage_default_resource_id = (known after apply)
      + state                      = (known after apply)
      + time_created               = (known after apply)

      + route_rules {
          + cidr_block        = (known after apply)
          + description       = (known after apply)
          + destination       = "0.0.0.0/0"
          + destination_type  = "CIDR_BLOCK"
          + network_entity_id = (known after apply)
        }
    }


<TRUNCATED OUTPUT>

Plan: 8 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + instance_ip = (known after apply)
```

if we have no error run:

```
terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # oci_core_default_route_table.default_oci_core_default_route_table will be created
  + resource "oci_core_default_route_table" "default_oci_core_default_route_table" {
      + compartment_id             = (known after apply)
      + defined_tags               = (known after apply)
      + display_name               = (known after apply)
      + freeform_tags              = (known after apply)
      + id                         = (known after apply)
      + manage_default_resource_id = (known after apply)
      + state                      = (known after apply)
      + time_created               = (known after apply)

      + route_rules {
          + cidr_block        = (known after apply)
          + description       = (known after apply)
          + destination       = "0.0.0.0/0"
          + destination_type  = "CIDR_BLOCK"
          + network_entity_id = (known after apply)
        }
    }

<TRUNCATED OUTPUT>

oci_core_default_route_table.default_oci_core_default_route_table: Creation complete after 1s [id=ocid1.routetable.oc1.eu-zurich-1.aaaaaaaa6jdgqpdgwsbwbnxcnkml2zhcbgtzpm4jynea6vid56p2ywrit3za]
oci_core_subnet.oci_core_subnet11: Creation complete after 1s [id=ocid1.subnet.oc1.eu-zurich-1.aaaaaaaam3443d65qq7aoc7kf4sftrwj3splrp7wiwzvboufoiu5tc5f7iaq]
oci_core_subnet.default_oci_core_subnet10: Creation complete after 4s [id=ocid1.subnet.oc1.eu-zurich-1.aaaaaaaapyx2u7vih7g6hflzvhe2qhs6utx5qqxezjf2lm5uyswrispjtnpq]
oci_core_instance.ubuntu_oci_instance: Creating...
oci_core_instance.ubuntu_oci_instance: Still creating... [10s elapsed]
oci_core_instance.ubuntu_oci_instance: Still creating... [20s elapsed]
oci_core_instance.ubuntu_oci_instance: Still creating... [30s elapsed]
oci_core_instance.ubuntu_oci_instance: Still creating... [40s elapsed]
oci_core_instance.ubuntu_oci_instance: Still creating... [50s elapsed]
oci_core_instance.ubuntu_oci_instance: Still creating... [1m0s elapsed]
oci_core_instance.ubuntu_oci_instance: Still creating... [1m10s elapsed]
oci_core_instance.ubuntu_oci_instance: Still creating... [1m20s elapsed]
oci_core_instance.ubuntu_oci_instance: Still creating... [1m30s elapsed]
oci_core_instance.ubuntu_oci_instance: Still creating... [1m40s elapsed]
oci_core_instance.ubuntu_oci_instance: Still creating... [1m50s elapsed]
oci_core_instance.ubuntu_oci_instance: Still creating... [2m0s elapsed]
oci_core_instance.ubuntu_oci_instance: Creation complete after 2m8s [id=ocid1.instance.oc1.eu-zurich-1.an5heljr5kjm7pycoborgwavcmh5xrjgd3ozciyvcsjsclxbvbtmrrbpzomq]

Apply complete! Resources: 8 added, 0 changed, 0 destroyed.

Outputs:

instance_ip = "152.x.x.x"

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```

Now we can ssh into our instance:

```
ssh ubuntu@152.x.x.x

...
35 updates can be applied immediately.
25 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@ubuntu-instance:~$ 
```

If we have setup all the variables correctly we are now inside our Oracle compute instance, we can now check if nginx is running:

```
ubuntu@ubuntu-instance:~$ systemctl status nginx.service 
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2021-10-29 09:11:21 UTC; 3h 58min ago
       Docs: man:nginx(8)
   Main PID: 8469 (nginx)
      Tasks: 2 (limit: 6861)
     Memory: 2.3M
     CGroup: /system.slice/nginx.service
             ├─8469 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             └─8470 nginx: worker process

Oct 29 09:11:21 ubuntu-instance systemd[1]: Starting A high performance web server and a reverse proxy server...
Oct 29 09:11:21 ubuntu-instance systemd[1]: Started A high performance web server and a reverse proxy server.
```

And from our workstation we can try to reach via browser (or curl) our public ip address on port 80 (HTTP):

```
curl http://152.x.x.x
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

...
...
...
```

### Cleanup

To cleanup/destroy our infrastrucure:

```
terraform destroy

<TRUNCATED OUTPUT>

Plan: 0 to add, 0 to change, 8 to destroy.

Changes to Outputs:
  - instance_ip = "152.x.x.x" -> null

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: 

<TRUNCATED OUTPUT>


oci_core_instance.ubuntu_oci_instance: Destruction complete after 1m6s
oci_core_subnet.default_oci_core_subnet10: Destroying... [id=ocid1.subnet.oc1.eu-zurich-1.aaaaaaaapyx2u7vih7g6hflzvhe2qhs6utx5qqxezjf2lm5uyswrispjtnpq]
oci_core_subnet.default_oci_core_subnet10: Destruction complete after 0s
oci_core_security_list.custom_security_list: Destroying... [id=ocid1.securitylist.oc1.eu-zurich-1.aaaaaaaa5qfkat6fldch3jkkderuwruxjnxdgxjq5auennrv2krwqvnsqt3q]
oci_core_default_security_list.default_security_list: Destroying... [id=ocid1.securitylist.oc1.eu-zurich-1.aaaaaaaaklnlfa5y36tmdxyimbfiobgegmrikbt3mnaoaf3kot4i74yxtqga]
oci_core_security_list.custom_security_list: Destruction complete after 1s
oci_core_default_security_list.default_security_list: Destruction complete after 1s
oci_core_vcn.default_oci_core_vcn: Destroying... [id=ocid1.vcn.oc1.eu-zurich-1.amaaaaaa5kjm7pyalfrmvv5hlg5bo35v4pgnqrkdrjaepbaqjt3c32ai7qaq]
oci_core_vcn.default_oci_core_vcn: Destruction complete after 1s

Destroy complete! Resources: 8 destroyed.
```
