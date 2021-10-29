---
layout: post
title: Deploy multiple Oracle compute instances using an instance pool and terraform
description: Deploy multiple Oracle compute instances using an instance pool, terraform and oracle cloud provider plugin
categories: [Oracle,Terraform,Cloud]
---

Welcome  to the third chapter of the series dedicated to the Oracle cloud infrastructure and terraform, if you have missed the previous chapters here you can find the links:

* [Setup Terraform Oracle Cloud provider]({{ site.baseurl }}/oracle-cloud-terraform-part1-setup).
* [Deploy an Oracle Cloud compute instance using terraform]({{ site.baseurl }}/oracle-cloud-terraform-part2-simple-instance) 

After we have successfully launched our [first instance ]({{ site.baseurl }}/oracle-cloud-terraform-part2-simple-instance) we are now ready for a more complicated example.

### Environment setup

In [our](https://github.com/garutilorenzo/oracle-cloud-terraform-examples) repository, change directory and go inside the instance-pool directory:

```
cd oracle-cloud-terraform-examples/instance-pool/
```

Modify the vars.tf in the same way whe have modified the vars.tf file in the simple instance [example]({{ site.baseurl }}/oracle-cloud-terraform-part2-simple-instance) (to setup the vars.tf file from scratch follow the Variables setup section) 

### Extra variables

We have some extra variables in this example:

| Variable | Default | Description |
| -------- |  ------- | ----------- |
| `fault_domains` | `"FAULT-DOMAIN-1", "FAULT-DOMAIN-2", "FAULT-DOMAIN-3"`      | This variable is a list of fault domains where our instance pool will deploy our instances |
| `instance_pool_size` | `2`      |  Number of instances to launch in the instance pool |

### Infrastructure overview

The infrastructure is the same as the simple instance [example]({{ site.baseurl }}/oracle-cloud-terraform-part2-simple-instance) but we have also:

* one network load balancer, that will route the traffic from the internet to our instance pool instances
* one instance configuration used by the instance pool
* one instance pool
* two Oracle compute instances launched by the instance pool

The network load balancer is made by:

* one listener (port 80)
* one backed set
* one backed for each of the instances in the instance pool

### Notes

Some important notes:

* By default firewall on the compute instances is disabled. On some test the firewall has created some problems
* Nginx will be installed by default (nginx is used for testing the security list rules, and for testing the network load balancer setup)
* The operating system used is Ubuntu 20.04

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

we are now ready to deploy our infrastructure:

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

Plan: 14 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + instances_ips = [
      + (known after apply),
      + (known after apply),
    ]
  + lb_ip         = (known after apply)

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```

if we have no error run:

```
terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create
 <= read (data resources)


Terraform will perform the following actions:

  # data.oci_core_instance.ubuntu_instance_pool_instances_ips[0] will be read during apply
  # (config refers to values not yet known)
 <= data "oci_core_instance" "ubuntu_instance_pool_instances_ips"  {
      + agent_config                        = (known after apply)
      + async                               = (known after apply)
      + availability_config                 = (known after apply)
      + availability_domain                 = (known after apply)
      + boot_volume_id                      = (known after apply)
      + capacity_reservation_id             = (known after apply)
      + compartment_id                      = (known after apply)
      + create_vnic_details                 = (known after apply)
      + dedicated_vm_host_id                = (known after apply)
      + defined_tags                        = (known after apply)
      + display_name                        = (known after apply)
      + extended_metadata                   = (known after apply)
      + fault_domain                        = (known after apply)
      + freeform_tags                       = (known after apply)
      + hostname_label                      = (known after apply)
      + id                                  = (known after apply)
      + image                               = (known after apply)
      + instance_id                         = (known after apply)
      + instance_options                    = (known after apply)
      + ipxe_script                         = (known after apply)
      + is_pv_encryption_in_transit_enabled = (known after apply)
      + launch_mode                         = (known after apply)
      + launch_options                      = (known after apply)
      + metadata                            = (known after apply)
      + platform_config                     = (known after apply)
      + preemptible_instance_config         = (known after apply)
      + preserve_boot_volume                = (known after apply)
      + private_ip                          = (known after apply)
      + public_ip                           = (known after apply)
      + region                              = (known after apply)
      + shape                               = (known after apply)
      + shape_config                        = (known after apply)
      + source_details                      = (known after apply)
      + state                               = (known after apply)
      + subnet_id                           = (known after apply)
      + system_tags                         = (known after apply)
      + time_created                        = (known after apply)
      + time_maintenance_reboot_due         = (known after apply)
    }

<TRUNCATED OUTPUT>

oci_network_load_balancer_listener.test_listener: Creation complete after 25s [id=networkLoadBalancers/ocid1.networkloadbalancer.oc1.eu-zurich-1.amaaaaaa5kjm7pyarkfapfnqqxrwaowlnmj5mnd3etmig5nfcwd3m5yb7uha/listeners/LB%20test%20listener]
oci_network_load_balancer_backend.test_backend[1]: Still creating... [31s elapsed]
oci_network_load_balancer_backend.test_backend[0]: Still creating... [31s elapsed]
oci_network_load_balancer_backend.test_backend[0]: Still creating... [41s elapsed]
oci_network_load_balancer_backend.test_backend[1]: Still creating... [41s elapsed]
oci_network_load_balancer_backend.test_backend[0]: Creation complete after 42s [id=networkLoadBalancers/ocid1.networkloadbalancer.oc1.eu-zurich-1.amaaaaaa5kjm7pyarkfapfnqqxrwaowlnmj5mnd3etmig5nfcwd3m5yb7uha/backendSets/Backend%20set%20test/backends/ocid1.instance.oc1.eu-zurich-1.an5heljr5kjm7pycu5exolhnubsq5isqo6nveddlmlsblkz7geb6vbwsvbtq.80]
oci_network_load_balancer_backend.test_backend[1]: Still creating... [51s elapsed]
oci_network_load_balancer_backend.test_backend[1]: Still creating... [1m1s elapsed]
oci_network_load_balancer_backend.test_backend[1]: Still creating... [1m11s elapsed]
oci_network_load_balancer_backend.test_backend[1]: Creation complete after 1m14s [id=networkLoadBalancers/ocid1.networkloadbalancer.oc1.eu-zurich-1.amaaaaaa5kjm7pyarkfapfnqqxrwaowlnmj5mnd3etmig5nfcwd3m5yb7uha/backendSets/Backend%20set%20test/backends/ocid1.instance.oc1.eu-zurich-1.an5heljr5kjm7pycft5ixge6ssknpyb5s6q3eihuccogpqrvv2ntqdlww72a.80]

Apply complete! Resources: 14 added, 0 changed, 0 destroyed.

Outputs:

instances_ips = [
  "132.x.x.x",
  "152.x.x.x",
]
lb_ip = tolist([
  {
    "ip_address" = "140.x.x.x"
    "is_public" = true
    "reserved_ip" = tolist([])
  },
])
```

Now we can ssh in one of the deployed instances:

```
ssh ubuntu@132.x.x.x

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

ubuntu@inst-ikudx-ubuntu-instance-pool:~$
```

After some minutes (at least one backend must be in HEALTH state) also the network load balancer will respond to our requests:

```
curl -v 140.x.x.x
*   Trying 140.x.x.x:80...
* TCP_NODELAY set
* Connected to 140.x.x.x (140.x.x.x) port 80 (#0)
> GET / HTTP/1.1
> Host: 140.x.x.x
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.18.0 (Ubuntu)
< Date: Wed, 27 Oct 2021 15:39:51 GMT
< Content-Type: text/html
< Content-Length: 672
< Last-Modified: Wed, 27 Oct 2021 15:33:26 GMT
< Connection: keep-alive
< ETag: "61797146-2a0"
< Accept-Ranges: bytes
...
...
...
```

### Cleanup

To cleanup/destroy our infrastrucure:

```
terraform destroy
```