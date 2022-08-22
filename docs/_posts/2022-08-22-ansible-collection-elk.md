---
layout: post
title: Install and configure a high available Elasticsearch cluster with Ansible
description: This ansible collection will install and configure a high available Elasticsearch cluste. In most cases you may prefer ECK or Elastic Cloud but if Kubernetes for you is like kryptonite for Superman or if you are jelous about your data or even more if you don't trust any Cloud provider this is the right place...
categories: [Ansible Collection,Ansible,elk,Elasticsearch,Kibana,Logstash,Beats]
---

![elk-Logo]({{ site.baseurl }}/images/elk-logo.png)

This [ansible collection](https://github.com/garutilorenzo/ansible-collection-elk) will install and configure a high available [Elasticsearch cluster](https://www.elastic.co/elasticsearch/).

With this collection you can also install and configure:

* [Logstash](https://www.elastic.co/logstash/) is a free and open server-side data processing pipeline that ingests data from a multitude of sources, transforms it, and then sends it to your favorite "stash."
* [Kibana](https://www.elastic.co/kibana/) is a free and open user interface that lets you visualize your Elasticsearch data and navigate the Elastic Stack. Do anything from tracking query load to understanding the way requests flow through your apps

And also the Elastic Beats:

* [filebeat](https://www.elastic.co/beats/filebeat) - Whether you’re collecting from security devices, cloud, containers, hosts, or OT, Filebeat helps you keep the simple things simple by offering a lightweight way to forward and centralize logs and files.
* [metricbeat](https://www.elastic.co/beats/metricbeat) Collect metrics from your systems and services. From CPU to memory, Redis to NGINX, and much more, Metricbeat is a lightweight way to send system and service statistics.
* [heartbeath](https://www.elastic.co/beats/heartbeat) Monitor services for their availability with active probing. Given a list of URLs, Heartbeat asks the simple question: Are you alive? Heartbeat ships this information and response time to the rest of the Elastic Stack for further analysis.

In most cases you may prefer [ECK](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html) or [Elastic Cloud](https://www.elastic.co/cloud/) but if Kubernetes for you is like kryptonite for Superman or if you are jelous about your data or even more if you don't trust any Cloud provider this is the right place.

To test this collection we use [Vagrant](https://www.vagrantup.com/) and [Virtualbox](https://www.virtualbox.org/), but if you prefer you can also use your own VMs or your baremetal machines.

The first step is to download this [repo](https://github.com/garutilorenzo/ansible-collection-elk) and birng up all the VMs. But first in the Vagrantfile paste your public ssh key in the *CHANGE_ME* variable. You can also adjust the number of the vm deployed by changing the NNODES variable (in this exaple we will use 6 nodes). Now we are ready to provision the machines:

```
git clone https://github.com/garutilorenzo/ansible-collection-elk.git

cd nsible-collection-elk/

vagrant up
Bringing machine 'elk-ubuntu-0' up with 'virtualbox' provider...
Bringing machine 'elk-ubuntu-1' up with 'virtualbox' provider...
Bringing machine 'elk-ubuntu-2' up with 'virtualbox' provider...
Bringing machine 'elk-ubuntu-3' up with 'virtualbox' provider...
Bringing machine 'elk-ubuntu-4' up with 'virtualbox' provider...
Bringing machine 'elk-ubuntu-5' up with 'virtualbox' provider...

[...]
[...]

    elk-ubuntu-5: Inserting generated public key within guest...
==> elk-ubuntu-5: Machine booted and ready!
==> elk-ubuntu-5: Checking for guest additions in VM...
    elk-ubuntu-5: The guest additions on this VM do not match the installed version of
    elk-ubuntu-5: VirtualBox! In most cases this is fine, but in rare cases it can
    elk-ubuntu-5: prevent things such as shared folders from working properly. If you see
    elk-ubuntu-5: shared folder errors, please make sure the guest additions within the
    elk-ubuntu-5: virtual machine match the version of VirtualBox you have installed on
    elk-ubuntu-5: your host and reload your VM.
    elk-ubuntu-5:
    elk-ubuntu-5: Guest Additions Version: 6.0.0 r127566
    elk-ubuntu-5: VirtualBox Version: 6.1
==> elk-ubuntu-5: Setting hostname...
==> elk-ubuntu-5: Configuring and enabling network interfaces...
==> elk-ubuntu-5: Mounting shared folders...
    elk-ubuntu-5: /vagrant => C:/Users/Lorenzo Garuti/workspaces/simple-ubuntu
==> elk-ubuntu-5: Running provisioner: shell...
    elk-ubuntu-5: Running: inline script
==> elk-ubuntu-5: Running provisioner: shell...
    elk-ubuntu-5: Running: inline script
    elk-ubuntu-5: hello from node 5
```

Now if you don't have Ansible installed, install ansible and all the requirements:

```
apt-get install python3 python3-pip
pip3 install pipenv

pipenv shell
pip install -r requirements.txt
```

Now with Ansible installed we can download the collection directly from GitHub:

```
ansible-galaxy collection install git+https://github.com/garutilorenzo/ansible-collection-elk
```

Whit Ansible and the collection installed we can setup our inventory file (hosts.ini):

```
[elasticsearch_master]
elk-ubuntu-0 ansible_host=192.168.25.110
elk-ubuntu-1 ansible_host=192.168.25.111
elk-ubuntu-2 ansible_host=192.168.25.112

[elasticsearch_data]
elk-ubuntu-3 ansible_host=192.168.25.113
elk-ubuntu-4 ansible_host=192.168.25.114
elk-ubuntu-5 ansible_host=192.168.25.115

[elasticsearch_ca]
elk-ubuntu-0 ansible_host=192.168.25.110

[kibana]
elk-ubuntu-1 ansible_host=192.168.25.111
elk-ubuntu-4 ansible_host=192.168.25.114

[logstash]
elk-ubuntu-2 ansible_host=192.168.25.112
elk-ubuntu-5 ansible_host=192.168.25.115

[elasticsearch:children]
elasticsearch_master
elasticsearch_data
```

and the vars.yml file:

```yaml
---

disable_firewall: yes
disable_selinux: yes

elasticsearch_version: 8.3.3
kibana_version: 8.3.3
logstash_version: 8.3.3
beats_version: 8.3.3

elasticsearch_resolv_mode: hosts
elasticsearch_install_mode: local
elasticsearch_local_tar_path: ~/elk_tar_path
elasticsearch_monitoring_enabled: yes
elasticsearch_master_is_also_data_node: yes

kibana_install_mode: local
kibana_local_tar_path: ~/elk_tar_path
setup_kibana_dashboards: yes
kibana_url: http://elk-ubuntu-1:5601

heartbeat_number_of_shards: 3
heartbeat_number_of_replicas: 3

metricbeat_number_of_shards: 3
metricbeat_number_of_replicas: 3

filebeat_number_of_shards: 3
filebeat_number_of_replicas: 3

elasticsearch_hosts:
  - elk-ubuntu-0 
  - elk-ubuntu-1
  - elk-ubuntu-2 
  - elk-ubuntu-3 
  - elk-ubuntu-5 
  - elk-ubuntu-5
```

The final cluster will be made of:

* 6 elasticsearch nodes (the master nodes will be also data nodes)
* 2 kibana instances
* 2 logstash instances

and:

* all VMs will be monitored by the Elastic Beats
* all the index templates will have 3 shards and 3 replicas
* Since we don't have any DNS available, Ansible will insert all the node names in the /etc/hosts file
* firewall and selinux will be disabled

To preserve bandwidth we download elasticsearch and kibana on our Ansible machine:

```
mkdir -p ~/elk_tar_path # <- you can customize this path by changing elasticsearch_local_tar_path variable
curl  -o ~/elk_tar_path/kibana-8.3.3-linux-x86_64.tar.gz https://artifacts.elastic.co/downloads/kibana/kibana-8.3.3-linux-x86_64.tar.gz
curl  -o ~/elk_tar_path/elk_tar_path/elasticsearch-8.3.3-linux-x86_64.tar.gz https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.3.3-linux-x86_64.tar.gz
```

and we have to create the certificate directory, where elastic certificates will be stored:

```
mkdir -p ~/very_secure_dir # <- you can customize this path by changing elasticsearch_local_certs_dir variable
```

The final step before proceed with the installation is to create the site.yml file:

```yaml
---
- hosts: elasticsearch 
  become: yes
  remote_user: vagrant
  collections:
    - garutilorenzo.ansible_collection_elk
  vars_files:
    - vars.yml

  tasks:
    - import_role:
        name: elasticsearch

- hosts: kibana 
  become: yes
  remote_user: vagrant
  collections:
    - garutilorenzo.ansible_collection_elk
  vars_files:
    - vars.yml

  tasks:
    - import_role:
        name: kibana

- hosts: logstash 
  become: yes
  remote_user: vagrant
  collections:
    - garutilorenzo.ansible_collection_elk
  vars_files:
    - vars.yml

  tasks:
    - import_role:
        name: logstash

- hosts: elasticsearch 
  become: yes
  remote_user: vagrant
  collections:
    - garutilorenzo.ansible_collection_elk
  vars_files:
    - vars.yml

  tasks:
    - import_role:
        name: beats
```

We are finally ready to install the ELK stack with ansible, since we don't have any CA certificate we pass an extra variable *generateca* to our playbook:

```
export ANSIBLE_HOST_KEY_CHECKING=False # Ansible skip ssh-key validation
ansible-playbook site.yml -i hosts.ini -e "generateca=yes"

ansible-playbook site.yml -i hosts.ini

PLAY [elasticsearch] ***************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************
ok: [elk-ubuntu-0]
ok: [elk-ubuntu-1]
ok: [elk-ubuntu-2]
ok: [elk-ubuntu-3]
ok: [elk-ubuntu-4]
ok: [elk-ubuntu-5]

TASK [garutilorenzo.ansible_collection_elk.elasticsearch : setup] ******************************************************************************************
ok: [elk-ubuntu-2]
ok: [elk-ubuntu-0]
ok: [elk-ubuntu-3]
ok: [elk-ubuntu-1]
ok: [elk-ubuntu-4]
ok: [elk-ubuntu-5]

TASK [garutilorenzo.ansible_collection_elk.elasticsearch : include_tasks] **********************************************************************************
included: /home/lorenzo/workspaces-local/ansible-test-collection/elk/collections/ansible_collections/garutilorenzo/ansible_collection_elk/roles/elasticsearch/tasks/preflight.yml for elk-ubuntu-0, elk-ubuntu-1, elk-ubuntu-2, elk-ubuntu-3, elk-ubuntu-4, elk-ubuntu-5

TASK [garutilorenzo.ansible_collection_elk.elasticsearch : Put SELinux in permissive mode, logging actions that would be blocked.] *************************
skipping: [elk-ubuntu-0]
skipping: [elk-ubuntu-1]
skipping: [elk-ubuntu-2]
skipping: [elk-ubuntu-3]
skipping: [elk-ubuntu-4]
skipping: [elk-ubuntu-5]

TASK [garutilorenzo.ansible_collection_elk.elasticsearch : Disable SELinux] ********************************************************************************
skipping: [elk-ubuntu-0]
skipping: [elk-ubuntu-1]
skipping: [elk-ubuntu-2]
skipping: [elk-ubuntu-3]
skipping: [elk-ubuntu-4]
skipping: [elk-ubuntu-5]

TASK [garutilorenzo.ansible_collection_elk.elasticsearch : disable firewalld] ******************************************************************************
skipping: [elk-ubuntu-0]
skipping: [elk-ubuntu-1]
skipping: [elk-ubuntu-2]
skipping: [elk-ubuntu-3]
skipping: [elk-ubuntu-4]
skipping: [elk-ubuntu-5]

TASK [garutilorenzo.ansible_collection_elk.elasticsearch : disable ufw] ************************************************************************************
changed: [elk-ubuntu-3]
changed: [elk-ubuntu-0]
changed: [elk-ubuntu-2]
changed: [elk-ubuntu-4]
changed: [elk-ubuntu-1]
changed: [elk-ubuntu-5]

[...]
[...]
```

This can take a while, but if everything goes well the final Ansible output will be:

```
[...]
[...]

TASK [garutilorenzo.ansible_collection_elk.beats : enable and start heartbeat] *****************************************************************************
changed: [elk-ubuntu-0]
changed: [elk-ubuntu-1]
changed: [elk-ubuntu-3]
changed: [elk-ubuntu-4]
changed: [elk-ubuntu-5]
changed: [elk-ubuntu-2]

RUNNING HANDLER [garutilorenzo.ansible_collection_elk.beats : reload systemd] ******************************************************************************
ok: [elk-ubuntu-4]
ok: [elk-ubuntu-0]
ok: [elk-ubuntu-3]
ok: [elk-ubuntu-1]
ok: [elk-ubuntu-2]
ok: [elk-ubuntu-5]

RUNNING HANDLER [garutilorenzo.ansible_collection_elk.beats : reload filebeat] *****************************************************************************
changed: [elk-ubuntu-0]
changed: [elk-ubuntu-4]
changed: [elk-ubuntu-1]
changed: [elk-ubuntu-2]
changed: [elk-ubuntu-5]
changed: [elk-ubuntu-3]

RUNNING HANDLER [garutilorenzo.ansible_collection_elk.beats : reload heartbeat] ****************************************************************************
changed: [elk-ubuntu-2]
changed: [elk-ubuntu-1]
changed: [elk-ubuntu-0]
changed: [elk-ubuntu-3]
changed: [elk-ubuntu-4]
changed: [elk-ubuntu-5]

RUNNING HANDLER [garutilorenzo.ansible_collection_elk.beats : reload metricbeat] ***************************************************************************
changed: [elk-ubuntu-1]
changed: [elk-ubuntu-4]
changed: [elk-ubuntu-0]
changed: [elk-ubuntu-3]
changed: [elk-ubuntu-5]
changed: [elk-ubuntu-2]

PLAY RECAP *************************************************************************************************************************************************
elk-ubuntu-0               : ok=119  changed=61   unreachable=0    failed=0    skipped=12   rescued=0    ignored=1   
elk-ubuntu-1               : ok=135  changed=68   unreachable=0    failed=0    skipped=15   rescued=0    ignored=1   
elk-ubuntu-2               : ok=136  changed=72   unreachable=0    failed=0    skipped=14   rescued=0    ignored=1   
elk-ubuntu-3               : ok=114  changed=58   unreachable=0    failed=0    skipped=12   rescued=0    ignored=1   
elk-ubuntu-4               : ok=135  changed=68   unreachable=0    failed=0    skipped=15   rescued=0    ignored=1   
elk-ubuntu-5               : ok=136  changed=72   unreachable=0    failed=0    skipped=14   rescued=0    ignored=1 
```

Now we use Kibana to analyze our Elasticsearch data:

![elk-login]({{ site.baseurl }}/images/elk-login.png)

The password is the default bootstrap password *changeme*. You can customize this variable in the elasticsearch role, you have to modify the *elasticsearch_bootstrap_password* variable. **Remember** to set in the vars.yml file the same value form *elasticsearch_password* and *elasticsearch_bootstrap_password*.

This is the default Kibana dashboard:

![elk-home]({{ site.baseurl }}/images/elk-home.png)

Now we can *Discover* our data in Analytics -> Discover:

![elk-discover]({{ site.baseurl }}/images/elk-discover.png)

Here we can filter our data using the KQL syntax or use one of the many Kibana features.

We can also inspect our data using one of the many Dashboards provided by the various Beats (Analytics -> Dashboards). We can inspect for example the metricbeat system dashboard:

![elk-dashboard]({{ site.baseurl }}/images/elk-dashboard.png)

In Management -> Stack Management -> Index Management we can see all of our indexes. We have to enable the *Include hidden indices* because in our brand new cluster all the indexes are hidden (filebeat, heartbeat, metricbeat):

![elk-index-management]({{ site.baseurl }}/images/elk-index-management.png)

In Management -> Stack Monitoring we can enable the *self monitoring* feature:

![elk-self-monitoring.png]({{ site.baseurl }}/images/elk-self-monitoring.png)

And now we can inspect the cluster stats:

![elk-self-status.png]({{ site.baseurl }}/images/elk-self-status.png)

This is only a short introduction on the ELK Stack, you can read more on the [Elastic Docs](https://www.elastic.co/guide/index.html).

When you have finished you can destroy the cluster with:

```
vagrant destroy
    elk-ubuntu-5: Are you sure you want to destroy the 'elk-ubuntu-5' VM? [y/N] y
==> elk-ubuntu-5: Forcing shutdown of VM...
==> elk-ubuntu-5: Destroying VM and associated drives...
    elk-ubuntu-4: Are you sure you want to destroy the 'elk-ubuntu-4' VM? [y/N] y
==> elk-ubuntu-4: Forcing shutdown of VM...
==> elk-ubuntu-4: Destroying VM and associated drives...
    elk-ubuntu-3: Are you sure you want to destroy the 'elk-ubuntu-3' VM? [y/N] y
==> elk-ubuntu-3: Forcing shutdown of VM...
==> elk-ubuntu-3: Destroying VM and associated drives...
    elk-ubuntu-2: Are you sure you want to destroy the 'elk-ubuntu-2' VM? [y/N] y
==> elk-ubuntu-2: Forcing shutdown of VM...
==> elk-ubuntu-2: Destroying VM and associated drives...
    elk-ubuntu-1: Are you sure you want to destroy the 'elk-ubuntu-1' VM? [y/N] y
==> elk-ubuntu-1: Forcing shutdown of VM...
==> elk-ubuntu-1: Destroying VM and associated drives...
    elk-ubuntu-0: Are you sure you want to destroy the 'elk-ubuntu-0' VM? [y/N] y
==> elk-ubuntu-0: Forcing shutdown of VM...
==> elk-ubuntu-0: Destroying VM and associated drives...
```