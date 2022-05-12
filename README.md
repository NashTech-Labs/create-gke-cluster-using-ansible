<img src="https://www.knoldus.com/wp-content/uploads/Knoldus-logo-1.png" style="height: 90px">

# Create GKE Cluster Using Ansible ![Ansible](https://www.vectorlogo.zone/logos/ansible/ansible-ar21.svg)![GCP](https://www.vectorlogo.zone/logos/google_cloud/google_cloud-icon.svg)   ![kubernetes](https://www.vectorlogo.zone/logos/kubernetes/kubernetes-icon.svg)

Ansible Role to provision GKE (Gcloud Kubernetes Engine).

## Prerequisites 

* GCP project with enabled billing
* GCP Service account with attached roles for GKE admin
* Service account Key 
* Ansible installed in you system

## The GCP modules require both the requests and the google-auth libraries to be installed.
```bash
    pip install requests google-auth
```
**To create service account refer to this line** https://developers.google.com/identity/protocols/oauth2/service-account#creatinganaccount

## Dyanmic variable for Gke Cluster [inventory](inventory/gcp_value.yaml)
**This file contains variable which can be changed as per the requirements**
 ``` bash
---
all:
  vars:
    # changes according to requirements
    zone: us-central1-c
    region: us-centeral1
    project_id: <project.id> #enter you project id
    gcloud_sa_path: "~/gcpserviceaccount.json" # Enter path to you service account json file
    gcloud_service_account: "service-account@project-id.iam.gserviceaccount.com"
    credential: "{{lookup('env','HOME') }}/{{gcloud_sa_path}}"
    

    #Cluster information
    cluster_name: "gkepratice" #Name of the cluster
    initial_node_count: 1   #Number of node for cluster
    disk_size_gb: 100
    disk_type: "pd-ssd"  #disk types 
    machine_type: "e2-medium"  #image types
 ```

## Create Role for k8s cluster and Node [Role](roles/kubernetes/tasks/main.yaml)
**Role file for kubernetes**
For creating cluster and node pool,Two module are being used 

**google.cloud.gcp_container_cluster** 

**google.cloud.gcp_container_node_pool**
```bash
    
- name: create a cluster
  google.cloud.gcp_container_cluster:
    name: "{{ cluster_name }}"
    initial_node_count: "{{ initial_node_count }}"
    location: "{{ zone }}"
    network: "{{ network.name }}"
    project: "{{ project_id }}"
    auth_kind: serviceaccount
    service_account_file: "{{ credential }}"
    state: present
  register: cluster  

#create node pool
- name: create a node pool
  google.cloud.gcp_container_node_pool:
    name: my-pool-{{ cluster_name }}
    initial_node_count: "{{ initial_node_count }}"
    cluster: "{{ cluster }}"
    config:
      disk_size_gb: "{{ disk_size_gb }}"
      disk_type: "{{ disk_type }}"
      machine_type: "{{ machine_type }}"
    location: "{{ zone }}"
    project: "{{ project_id }}"
    auth_kind: serviceaccount
    service_account_file: "{{ credential }}"
    state: present

```

## Create Role for k8s networks  [Network](roles/network/tasks/main.yaml)
```bash
  - name: Create network
  google.cloud.gcp_compute_network:
    name: network-{{cluster_name}}
    auto_create_subnetworks: 'true'
    project: "{{project_id}}"
    auth_kind: serviceaccount
    service_account_file: "{{credential}}"
    state: present
  register: network 

```
## Create playbook [kubernetes](k8s.yml)
**First we create network then spin up the kubernetes cluster**
```bash
    ---
- name: create cluster
  hosts: localhost
  gather_facts: false
  environment:   #cred for serviceaccount.json
    GOOGLE_CREDENTIALS: "{{ credential }}"

  roles:
    - network
    - kubernetes

```
## Run ansible-playbook for Setup GKE
```bash
ansible-playbook  k8s.yml -i inventory/gcp_value.yaml
```