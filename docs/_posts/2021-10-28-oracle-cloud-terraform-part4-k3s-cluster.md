---
layout: post
title: Deploy a k3s cluster on Oracle Cloud using terraform
description: Deploy a k3s cluster on Oracle Cloud using terraform and oracle cloud provider plugin
categories: [Oracle,Terraform,Cloud]
---

Welcome  to the last chapter of the series dedicated to the Oracle cloud infrastructure and terraform, if you have missed the previous chapters here you can find the links:

* [Setup Terraform Oracle Cloud provider]({{ site.baseurl }}/oracle-cloud-terraform-part1-setup).
* [Deploy an Oracle Cloud compute instance using terraform]({{ site.baseurl }}/oracle-cloud-terraform-part2-simple-instance) 
* [Deploy multiple Oracle Cloud compute instances using an instance pool using terraform]({{ site.baseurl }}/oracle-cloud-terraform-part3-instance-pool)

Now we have all the knowledge for deploying a [k3s](https://k3s.io/) cluster on Oracle Cloud infrastructure.

### Notes

**NOTE** This k3s setup is **non** high available, the k3s server is a single point of failure. This setup is for testing purposes **only**

### Environment setup

In [our](https://github.com/garutilorenzo/oracle-cloud-terraform-examples) repository, change directory and go inside the k3s-cluster directory:

```
cd oracle-cloud-terraform-examples/k3s-cluster/
```

Modify the vars.tf in the same way whe have modified the vars.tf file in the simple instance [example]({{ site.baseurl }}/oracle-cloud-terraform-part2-simple-instance) (to setup the vars.tf file from scratch follow the Variables setup section) 

### Extra variables

We have some extra variables in this example:

| Variable | Default | Description |
| -------- |  ------- | ----------- |
| `fault_domains` | `"FAULT-DOMAIN-1", "FAULT-DOMAIN-2", "FAULT-DOMAIN-3"`      | This variable is a list of fault domains where our instance pool will deploy our instances |
| `instance_pool_size` | `3`      |  Number of instances to launch in the instance pool. Number of k3s agents to deploy |
| `k3s_server_private_ip` | `10.0.0.50`      | private ip address that will be associated to the k3s-server |
| `k3s_token` | `2aaf122eed3409ds2c6fagfad4073-92dcdgade664d8c1c7f49z`      |  token used to install the k3s cluster |
| `install_longhorn` | `true`      |  boolean value, if true (default) will install [longhorn](https://longhorn.io/) block storage  |
| `longhorn_release` | `v1.2.2`      | longorn release version |


### Infrastructure overview

The infrastructure is the same as the instance-pool [example]({{ site.baseurl }}/oracle-cloud-terraform-part3-instance-pool), but in the network load balancer we have one more listener (port 443, HTTPS).

### Notes

Some important notes:

* By default firewall on the compute instances is disabled. On some test the firewall has created some problems
* k3s will be installed in all the instances
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

Plan: 21 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + k3s_agents_ips = [
      + (known after apply),
      + (known after apply),
      + (known after apply),
    ]
  + k3s_server_ip  = (known after apply)
  + lb_ip          = (known after apply)

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

  # data.oci_core_instance.k3s_agents_instances_ips[0] will be read during apply
  # (config refers to values not yet known)
 <= data "oci_core_instance" "k3s_agents_instances_ips"  {
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

oci_network_load_balancer_backend.k3s_http_backend[2]: Still creating... [1m40s elapsed]
oci_network_load_balancer_backend.k3s_https_backend[1]: Still creating... [1m51s elapsed]
oci_network_load_balancer_backend.k3s_https_backend[0]: Still creating... [1m51s elapsed]
oci_network_load_balancer_backend.k3s_http_backend[2]: Still creating... [1m50s elapsed]
oci_network_load_balancer_backend.k3s_http_backend[0]: Still creating... [1m50s elapsed]
oci_network_load_balancer_backend.k3s_https_backend[0]: Still creating... [2m1s elapsed]
oci_network_load_balancer_backend.k3s_https_backend[1]: Still creating... [2m1s elapsed]
oci_network_load_balancer_backend.k3s_http_backend[0]: Still creating... [2m0s elapsed]
oci_network_load_balancer_backend.k3s_http_backend[2]: Still creating... [2m0s elapsed]
oci_network_load_balancer_backend.k3s_http_backend[0]: Creation complete after 2m3s [id=networkLoadBalancers/ocid1.networkloadbalancer.oc1.eu-zurich-1.amaaaaaa5kjm7pyaglmtxbdrio5sp5vetj3b7hwhhxxhd7xtgytvqo4ckfsq/backendSets/k3s%20http%20backend/backends/ocid1.instance.oc1.eu-zurich-1.an5heljr5kjm7pycxldegwyajkogb3hdz6sup45qwpnqirxuck6l5y3jxwqq.80]
oci_network_load_balancer_backend.k3s_https_backend[1]: Creation complete after 2m6s [id=networkLoadBalancers/ocid1.networkloadbalancer.oc1.eu-zurich-1.amaaaaaa5kjm7pyaglmtxbdrio5sp5vetj3b7hwhhxxhd7xtgytvqo4ckfsq/backendSets/k3s%20https%20backend/backends/ocid1.instance.oc1.eu-zurich-1.an5heljr5kjm7pycs67mnhim6h46vimtx6akahatgssfprxkpti6ij5aqm5q.443]
oci_network_load_balancer_backend.k3s_https_backend[0]: Still creating... [2m11s elapsed]
oci_network_load_balancer_backend.k3s_http_backend[2]: Still creating... [2m10s elapsed]
oci_network_load_balancer_backend.k3s_https_backend[0]: Still creating... [2m21s elapsed]
oci_network_load_balancer_backend.k3s_http_backend[2]: Still creating... [2m20s elapsed]
oci_network_load_balancer_backend.k3s_http_backend[2]: Creation complete after 2m21s [id=networkLoadBalancers/ocid1.networkloadbalancer.oc1.eu-zurich-1.amaaaaaa5kjm7pyaglmtxbdrio5sp5vetj3b7hwhhxxhd7xtgytvqo4ckfsq/backendSets/k3s%20http%20backend/backends/ocid1.instance.oc1.eu-zurich-1.an5heljr5kjm7pyca6sjt44vji4axtni5ebyc2mq66hobokh2yfz5qsljnia.80]
oci_network_load_balancer_backend.k3s_https_backend[0]: Still creating... [2m31s elapsed]
oci_network_load_balancer_backend.k3s_https_backend[0]: Creation complete after 2m38s [id=networkLoadBalancers/ocid1.networkloadbalancer.oc1.eu-zurich-1.amaaaaaa5kjm7pyaglmtxbdrio5sp5vetj3b7hwhhxxhd7xtgytvqo4ckfsq/backendSets/k3s%20https%20backend/backends/ocid1.instance.oc1.eu-zurich-1.an5heljr5kjm7pycxldegwyajkogb3hdz6sup45qwpnqirxuck6l5y3jxwqq.443]

Apply complete! Resources: 21 added, 0 changed, 0 destroyed.

Outputs:

k3s_agents_ips = [
  "132.x.x.x",
  "152.x.x.x",
  "152.x.x.x",
]
k3s_server_ip = "132.x.x.x"
lb_ip = tolist([
  {
    "ip_address" = "152.x.x.x"
    "is_public" = true
    "reserved_ip" = tolist([])
  },
])
```

Now we can ssh in our k3s-server instance:

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

ubuntu@k3s-server:~$
```

After some minutes (at least one backend must be in HEALTH state) also the network load balancer will respond to our requests:

```
curl -v http://152.x.x.x/
*   Trying 152.x.x.x:80...
* TCP_NODELAY set
* Connected to 152.x.x.x (152.x.x.x) port 80 (#0)
> GET / HTTP/1.1
> Host: 152.x.x.x
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< Content-Type: text/plain; charset=utf-8
< X-Content-Type-Options: nosniff
< Date: Wed, 27 Oct 2021 13:20:05 GMT
< Content-Length: 19
< 
404 page not found
* Connection #0 to host 152.x.x.x left intact
```

**NOTE** 404 is a correct response since there are no deployment yet

### k3s cluster management

To manage the cluster, open a ssh connection to the k3s-server, here some basic kubectl commands:

**List the nodes**

```
root@k3s-server:~# kubectl get nodes
NAME                    STATUS   ROLES                  AGE   VERSION
inst-vr4sv-k3s-agents   Ready    <none>                 23m   v1.21.5+k3s2
inst-zkcyl-k3s-agents   Ready    <none>                 23m   v1.21.5+k3s2
k3s-server              Ready    control-plane,master   23m   v1.21.5+k3s2
inst-fhayc-k3s-agents   Ready    <none>                 23m   v1.21.5+k3s2
```

**Get the pods running on kube-system namespace**

```
kubectl get pods -n kube-system
NAME                                      READY   STATUS      RESTARTS   AGE
coredns-7448499f4d-jwgzt                  1/1     Running     0          34m
metrics-server-86cbb8457f-qjgr9           1/1     Running     0          34m
local-path-provisioner-5ff76fc89d-56c7n   1/1     Running     0          34m
helm-install-traefik-crd-9ftr8            0/1     Completed   0          34m
helm-install-traefik-2v48n                0/1     Completed   2          34m
svclb-traefik-2x9q9                       2/2     Running     0          33m
svclb-traefik-d72cf                       2/2     Running     0          33m
svclb-traefik-jq5wv                       2/2     Running     0          33m
svclb-traefik-xnhgs                       2/2     Running     0          33m
traefik-97b44b794-4dz2x                   1/1     Running     0          33m
```

**Get the pods running on longhorn-system namespace (optional)**

```
root@k3s-server:~# kubectl get pods -n longhorn-system
NAME                                        READY   STATUS             RESTARTS   AGE
longhorn-ui-788fd8cf9d-76x84                1/1     Running            0          29m
longhorn-manager-97vzd                      1/1     Running            0          29m
longhorn-driver-deployer-5dff5c7554-c7wbk   1/1     Running            0          29m
longhorn-manager-sq2xn                      1/1     Running            1          29m
csi-attacher-75588bff58-xv9sn               1/1     Running            0          28m
csi-resizer-5c88bfd4cf-ngm2j                1/1     Running            0          28m
engine-image-ei-d4c780c6-ktvs7              1/1     Running            0          28m
csi-provisioner-669c8cc698-mqvjx            1/1     Running            0          28m
longhorn-csi-plugin-9x5wj                   2/2     Running            0          28m
engine-image-ei-d4c780c6-r7r2t              1/1     Running            0          28m
csi-provisioner-669c8cc698-tvs9r            1/1     Running            0          28m
csi-resizer-5c88bfd4cf-h8g6w                1/1     Running            0          28m
instance-manager-e-7aca498c                 1/1     Running            0          28m
instance-manager-r-98153684                 1/1     Running            0          28m
longhorn-csi-plugin-wf24d                   2/2     Running            0          28m
csi-snapshotter-69f8bc8dcf-n85hq            1/1     Running            0          28m
longhorn-csi-plugin-82hv5                   2/2     Running            0          28m
longhorn-csi-plugin-rlcw2                   2/2     Running            0          28m
longhorn-manager-rttww                      1/1     Running            1          29m
instance-manager-e-e43d97f9                 1/1     Running            0          28m
longhorn-manager-47zxl                      1/1     Running            1          29m
instance-manager-r-de0dc83b                 1/1     Running            0          28m
engine-image-ei-d4c780c6-hp4mb              1/1     Running            0          28m
engine-image-ei-d4c780c6-hcwpg              1/1     Running            0          28m
instance-manager-r-464299ad                 1/1     Running            0          28m
instance-manager-e-ccb8666b                 1/1     Running            0          28m
instance-manager-r-3b35070e                 1/1     Running            0          28m
instance-manager-e-9d117ead                 1/1     Running            0          28m
```

### Dploy an example application

We are now deploying an example app (nginx) with:

* a persistent storage (longhorn NFS)
* a ingress route, exposed by [Traefik](https://doc.traefik.io/traefik/providers/kubernetes-ingress/) ingress controller

Create the deployment:

```yaml
cat <<EOF > test-deployment.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: longhorn-volv-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: longhorn
  resources:
    requests:
      storage: 2Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  selector:
    matchLabels:
      app: metrics
      department: oci
  replicas: 3
  template:
    metadata:
      labels:
        app: metrics
        department: oci
    spec:
      containers:
      - name: volume-test
        image: nginx:stable-alpine
        imagePullPolicy: IfNotPresent
        livenessProbe:
          exec:
            command:
              - ls
              - /data/
          initialDelaySeconds: 5
          periodSeconds: 5
        volumeMounts:
        - name: volv
          mountPath: /data
        ports:
        - containerPort: 80
      volumes:
      - name: volv
        persistentVolumeClaim:
          claimName: longhorn-volv-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: my-cip-service
spec:
  type: ClusterIP
  selector:
    app: metrics
    department: oci
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
kind: Ingress
apiVersion:  networking.k8s.io/v1
metadata:
  name: "foo"
spec:
  rules:
    - host: example.net
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name:  my-cip-service
                port:
                  number: 80
EOF
```

apply the deployment with:

```
kubectl apply -f test-deployment.yml 
persistentvolumeclaim/longhorn-volv-pvc created
deployment.apps/my-deployment created
service/my-cip-service created
ingress.networking.k8s.io/foo created
```

monitor the deployment status:

```
kubectl get deployments
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
my-deployment   3/3     3            3           48s

kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
my-deployment-5d75fc6b89-xmxzt   1/1     Running   0          46s
my-deployment-5d75fc6b89-xbll2   1/1     Running   0          46s
my-deployment-5d75fc6b89-4g2w8   1/1     Running   0          46s
```

Now test the reachability of the example.net domain:

```
curl -H 'Host: example.net' http://152.x.x.x # <- This is the network load balancer IP

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

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### Cleanup

To cleanup/destroy our infrastrucure:

```
terraform destroy
```