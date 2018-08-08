Enabling IBM API Connect on IBM Cloud Private Version 2.1.0.3
=============================================================

# Introduction

This document provides guidance for installing **IBM API Connect v2018.3.3** on **IBM Cloud Private 2.1.0.3**. Also, it covers the topic of using all the components of IBM API Connect and tips for troubleshooting issues.

This document presumes that following two main pre-requisites are completed:

- [Install and Configure IBM Cloud Private on Redhat Linux systems](./Install%20ICP%20on%20RedHat.md)
- [Install and Configure GlusterFS on Redhat Linux systems](./Install%20GlusterFS%20for%20ICP.md)

This document can be used as a reference for setting up IBM API Connect non-production environment in IBM Cloud Private.

**Note:** When API Connect is run in demo mode only, **hostpath** can be used as an alternate storage provider instead of **GlusterFS**. Also, based on the customer requirement it is possible to use other storage providers like **ceph**. 

# Environment

A typical IBM Cloud Private Environment includes Boot/Master node, Management node, Proxy node and Worker Nodes. In addition to that IBM API Connect setup involves 3 additional nodes that providing dynamic storage.

The following set of systems are used in building an environment that runs IBM API Connect on IBM Cloud Private.

| Node type | Number of nodes | CPU | Memory (GB) | Disk (GB) |
| :---: | :---: | :---: | :---: | :---: |
|	Boot/Master	| 1	| 8	| 32 | 250 |
|	Management | 1	| 8	| 32 | 300 |
|	Proxy	| 1	| 4	| 16 | 250 |
|	Worker | 4 | 8 | 32	| 250 |
|	GlusterFS | 3| 4 | 16 | 40 + 500 |
|	Total |	10	| 64| 256	| 3370 |

**Note:** It is very critical to have minimum 4 worker nodes having **8 cores** and **32 GB RAM** is made available running IBM API Connect in IBM Cloud Private.


# Setup

The following tasks are performed for enabling IBM API Connect in IBM Cloud Private .

1. [Install IBM Cloud CLI](#1-install-ibm-cloud-cli)
2. [Install IBM Cloud Private CLI](#2-install-ibm-cloud-private-cli)
3. [Start terminal session to deploy IBM API Connect](#3-start-terminal-session-to-deploy-ibm-api-connect)
4. [Create namespace and update roles for the default user](#4-create-namespace-and-update-roles-for-the-default-user)
5. [Update environment to use storage class](#5-update-environment-to-use-storage-class)
6. [Create registry secret for IBM API Connect](#6-create-registry-secret-for-ibm-api-connect)
7. [Create helm tls secret for IBM API Connect](#7-create-helm-tls-secret-for-ibm-api-connect)
8. [Download API Connect images](#8-download-api-connect-images)
9. [Load API Connect images](#9-load-api-connect-images)
10. [Install the API Connect Helm chart](#10-install-the-api-connect-helm-chart)
11. [Verify API Connect Install](#11-verify-api-connect-install)
12. [Troubleshooting API Connect Install](#12-troubleshooting-api-connect-install)
13. [Install and Configure SMTP](#13-install-and-configure-smtp)
14. [Login to the Cloud Manager](#14-login-to-the-cloud-manager)
15. [Login to the API Manager](#15-login-to-the-api-manager)
16. [Login to the Developer Portal](#16-login-to-the-developer-portal)


### 1. Install IBM Cloud CLI

The [Getting started with IBM Cloud CLI link](https://console.bluemix.net/docs/cli/reference/bluemix_cli/get_started.html#getting-started) has details about installing IBM Cloud CLI. The installer can be downloaded for Linux and it can be kicked by exacting the package and running the *install_bluemix_cli* script on the Master Node.

The commands used are as follows:

```
tar -xf IBM_Cloud_CLI_amd64.tar
cd Bluemix_CLI
./install_bluemix_cli
```

The screen shot having the output of the aforesaid commands is listed below.

![](./images/install_bluemix_cli.png)


### 2. Install IBM Cloud Private CLI

The following steps can be followed to download and install the plugin for IBM Cloud Private CLI.

- Logon to https://mycluster.icp:8443/console
- Select the menu option *"Command Line Tools"* -> *"Cloud Private CLI"*  from the Navigation bar on the left hand side
- Download the plugin for the Linux 64 bit platform to the folder *./downloads*
- Run the following commands to install the plugin on the Master Node.

```
ibmcloud plugin install ./icp-linux-amd64
ibmcloud plugin show icp
```

**Note** It is critical for **IBM Cloud Private CLI** version to match the version used to install **IBM Cloud Private**.

The output of the aforesaid commands is listed below.

~~~
[root@rsun-rhel-bootmaster01 downloads]# ibmcloud plugin install ./icp-linux-amd64
Installing binary...
OK
Plug-in 'icp 2.1.343' was successfully installed into /root/.bluemix/plugins/icp. Use 'ibmcloud plugin show icp' to show its details.
[root@rsun-rhel-bootmaster01 downloads]# ibmcloud plugin show icp

Plugin                         icp   
Version                        2.1.343   
SDK Version                       
Minimal CLI version required   0.4.9   

Commands:
pr api                          View the API endpoint and API version for the service.   
pr certificate-delete           Delete a user certificate.   
pr certificates                 List user certificates.   
pr cluster-config               Download the Kubernetes configuration and configure kubectl for a specified cluster.   
pr cluster-get                  View details for a cluster.   
pr clusters                     List all the clusters in your account.   
pr credentials-set              Set the infrastructure account credentials for the OpenStack or VMware cloud provider   
pr credentials-set-openstack    Set the infrastructure account credentials for the OpenStack cloud provider.   
pr credentials-set-vmware       Set the infrastructure account credentials for the VMware cloud provider.   
pr credentials-unset            Remove cloud provider credentials. After you remove the credentials, you cannot access the cloud provider.   
pr delete-helm-chart            Deletes a Helm chart from the IBM Cloud Private internal registry.   
pr init                         Initialize the IBM Cloud Private plugin with the API endpoint.   
pr load-helm-chart              Loads a Helm chart archive to an IBM Cloud Private cluster.   
pr load-images                  Loads Docker images in to an IBM Cloud Private internal Docker registry.   
pr load-ppa-archive             Load Docker images and Helm charts compressed file that you downloaded from Passport Advantage.   
pr locations                    List available locations.   
pr login                        Log user in.   
pr logout                       Log user out.   
pr machine-type-add             Add a machine type. A machine type determines the number of CPUs, the amount of memory, and disk space that is available to the node.   
pr machine-type-add-openstack   Add an openstack machine type. A machine type determines the number of CPUs, the amount of memory, and disk space that is available to the node.   
pr machine-type-add-vmware      Add a vmware machine type. A machine type determines the number of CPUs, the amount of memory, and disk space that is available to the node.   
pr machine-types                List available machine types for a location. A machine type determines the number of CPUs, the amount of memory, and disk space that is available to the master/worker node.   
pr master-get                   View the details about a master node.   
pr masters                      List all master nodes in an existing cluster.   
pr password-rule-rm             Remove a password rule for a cluster namespace.   
pr password-rule-set            Set a password rule for a cluster namespace.   
pr password-rules               List the password rules for a cluster namespace.   
pr proxies                      List all proxy nodes in an existing cluster.   
pr proxy-add                    Add a proxy node to a cluster.   
pr proxy-get                    View the details about a proxy node.   
pr proxy-rm                     Remove proxy nodes from an existing cluster.   
pr registry-init                Initialize cluster image registry.   
pr iam roles                    List roles   
pr iam service-api-key          List details of a service API key   
pr iam service-api-key-create   Create a service API key   
pr iam service-api-key-delete   Delete a service API key   
pr iam service-api-key-update   Update a service API key   
pr iam service-api-keys         List all API keys of a service   
pr iam service-id               Display details of a service ID   
pr iam service-id-create        Create a service ID   
pr iam service-id-delete        Delete a service ID   
pr iam service-id-update        Update a service ID   
pr iam service-ids              List all service IDs.   
pr iam service-policies         List all service policies of specified service   
pr iam service-policy           Display details of a service policy   
pr iam service-policy-create    Create a service policy   
pr iam service-policy-delete    Delete a service policy   
pr iam service-policy-update    Update a service policy   
pr iam services                 List services   
pr target                       Set or view the targeted namespace.   
pr tokens                       Display the oauth tokens for the current session. Run ibmcloud pr login to retrieve the tokens.   
pr update-secret                Update a secret and restart deployments that use the secret.   
pr worker-add                   Add a worker node to a cluster.   
pr worker-get                   View the details about a worker node.   
pr worker-rm                    Remove worker nodes from an existing cluster.   
pr workers                      List all worker nodes in an existing cluster.
~~~


### 3. Start terminal session to deploy IBM API Connect

The following commands can be run to prepare the terminal session to deploy IBM API Connect.

**NOTE** The value *mycluster.icp* should be updated to match your environment.

```
ibmcloud pr login -a https://mycluster.icp:8443 --skip-ssl-validation
<<Provide username and password when prompted and select 1 to select the ICP account.>>
docker login mycluster.icp:8500
```

The screen shot having the output of the aforesaid commands is listed below.

![](./images/start_session.png)

Also, logon to the IBM Cloud Private console using the link https://mycluster.icp:8443/console and click on the user icon on the left and copy the configuration commands which would look like something similar to the following:

![](./images/configure_client.png)

```
kubectl config set-cluster cluster.local --server=https://MASTER_NODE_IP:8001 --insecure-skip-tls-verify=true
kubectl config set-context cluster.local-context --cluster=cluster.local
kubectl config set-credentials admin --token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhdF9oYXNoIjoicDlqcGtud202YTJlc3VzeG1na2siLCJyZWFsbU5hbWUiOiJjdXN0b21SZWFsbSIsInVuaXF1ZVNlY3VyaXR5TmFtZSI6ImFkbWluIiwiaXNzIjoiaHR0cHM6Ly9teWNsdXN0ZXIuaWNwOjk0NDMvb2lkYy9lbmRwb2ludC9PUCIsImF1ZCI6IjNiNjUwOGM0ZTA0MTNiMWUzODFmN2U2NzVkMmY3NGM4IiwiZXhwIjoxNTI3OTI5NTA2LCJpYXQiOjE1Mjc5MDA3MDYsInN1YiI6ImFkbWluIiwidGVhbVJvbGVNYXBwaW5ncyI6W119.JgC-E4aIlZ4-AAyBcGgMAs_QMrOgOOlpfEVk0q5O-r5V_QUgBfktMORFaGt03aLYck3EzyWX2haWbZH9_Ptb4Shvfv9APWch64D6BD-vF_eOUMhYwznhaeLqG_F7i0byQ7m5k4Df_vEZFKQy1y_CdlVFR5Wby49kMu8jYVz0fQtpM4n48RssQpQRVZ2lHOnBFnQCzT_Rvg4TtFhLGTxrtFhh5O-qsTI3fpmJgGSMgdSJsmlrrlvTCnb-mVpCWtBRWOYOSwDJcao34rcIy92muGajabdtGUCn0v-CDAoCdpUIqjDBekoEoNT7DvLZ9x9Bkv6w84Jaf7B0MeSqIsVCAg
kubectl config set-context cluster.local-context --user=admin --namespace=apiconnect
kubectl config use-context cluster.local-context
```

The screen shot having the output of the aforesaid commands is listed below.

![](./images/run_kubectl_commands.png)


### 4. Create namespace and update roles for the default user

The following commands can be used to create the namespace **apiconnect** in IBM Cloud Private and update the role for the default user.

```
kubectl create namespace apiconnect
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: apiconnect-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: default
  namespace: apiconnect

EOF
```
The screen shot having the output of the aforesaid commands is listed below.

![](./images/create_namespace.png)


### 5. Update environment to use storage class

API Connect requires storage to be configures so that it can persist data. 

The storage provider options **hostpath** and **GlusterFS** are included as reference. Note that it is required to have only one storage provider configured. 


#### 5.1 Update environment to use hostpath storage class

The following commands can be used to create the resources required to provision storage provider using *hostport*.

**Note:** Before running the following commands, the directory **/apiconnect** should be created in all the worker node(s) with full access rights (*chmod 777 apiconnect*). 

**Note:** The files required can be downloaded using the following link:  

- [hostpath-storageclass.yaml](./setup/hostpath-storageclass.yaml)
- [hostpath-provisioner.yaml](./setup/hostpath-provisioner.yaml)
- [hostpath-rbac.yaml](./setup/hostpath-rbac.yaml)

```
kubectl create -f hostpath-storageclass.yaml
kubectl create -f hostpath-provisioner.yaml 
kubectl create -f hostpath-rbac.yaml 
```

The output of the aforesaid commands is listed below.

```
# kubectl create -f hostpath-storageclass.yaml
storageclass "hostpath" created
# kubectl create -f hostpath-provisioner.yaml 
deployment "hostpath-provisioner" created
# kubectl create -f hostpath-rbac.yaml 
clusterrole "hostpath-provisioner" created
clusterrolebinding "hostpath-provisioner" created
```

#### 5.2 Update environment to use GlusterFS storage class

The following command is run to create Heketo secret.

**Note:** The **user** and **key** should be changed to match your environment. It should be
base64 encoding of username and password for accessing GlusterFS systems.

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: heketi-secret
  namespace: apiconnect
data:
  user: YWRtaW4=
  key: YWRtaW5hZG1pbg==
type: kubernetes.io/glusterfs  
EOF
```

**Note:** The environment variables *HEKETI_CLI_SERVER*, *HEKETI_CLI_USER*, *HEKETI_CLI_KEY* shoud be updated to match your Heketi environment.

The following command to get the clusterID of glusterFS from the management node.

```
export HEKETI_CLI_SERVER=http://localhost:8081
export HEKETI_CLI_USER=admin
export HEKETI_CLI_KEY=adminadmin
heketi-cli cluster list
```

**Note:** The *CLUSTE_ID* can be set using the output of the command *heketi-cli cluster list*. Also, *resturl* should be updated to match your environment.

```
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: apic.shared.storage
  namespace: apiconnect
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://MASTER_NODE_IP:8081"
  clusterid: "CLUSTER_ID"
  restuser: "admin"
  secretNamespace: "apiconnect"
  secretName: "heketi-secret"
  volumetype: "replicate:3"  
EOF
```

The following command is run to set the default storage class:

```
kubectl patch storageclass apic.shared.storage -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-apiconnect-class":"true"}}}'
```

The screen shot having the output of the aforesaid commands is listed below.

![](./images/storage_commands.png)


### 6. Create registry secret for IBM API Connect

The following command can be run to create the registry secret: **apiconnect-icp-secret**

**NOTE** The values *mycluster.icp*, *docker-username*, *docker-password* should be updated to match your environment.

```
kubectl create secret docker-registry apiconnect-icp-secret --docker-server=mycluster.icp:8500 --docker-username=admin --docker-password=admin --docker-email=admin@admin.com --namespace apiconnect
```

The screen shot having the output of the aforesaid commands is listed below.

![](./images/create_secret.png)


### 7. Create helm tls secret for IBM API Connect

This step is required for the release **IBM Connect v2018.2.9** or latter.

The following command can be run to create the registry secret: **helm-tls-secret**

```
ibmcloud pr clusters
ibmcloud pr cluster-config mycluster
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
 name: helm-tls-secret
 namespace: apiconnect
data:
 ca.pem: $(cat ~/.helm/ca.pem | base64 --wrap=0)
 cert.pem: $(cat ~/.helm/cert.pem | base64 --wrap=0)
 key.pem: $(cat ~/.helm/key.pem | base64 --wrap=0)
EOF
```
The screen shot having the output of the aforesaid commands is listed below.

![](./images/create_helm_tls_secret.png)


### 8. Download API Connect images

The link [IBM API Connect V2018.3.3 is available](http://www-01.ibm.com/support/docview.wss?uid=ibm10718281) has additional details of **IBM API Connect V2018.3.3**

The archive file **IBM_API_Connect_ICP_Enterprise_v2018.3.3.zip** is downloaded from the [Fix Central](http://www.ibm.com/support/fixcentral/quickorder?product=ibm%2FWebSphere%2FIBM+API+Connect&fixids=IBM_API_Connect_ICP_Enterprise_v2018.3.3&source=SAR) and moved to the master node of the IBM Cloud Private environment.

The following is the list of IBM API Connect images included in the archive file  **IBM_API_Connect_ICP_Enterprise_v2018.3.3.zip**

- *analytics-images-icp.tgz*
- *gateway-images-icp.tgz*
- *ibm-apiconnect-ent.tgz*
- *management-images-icp.tgz*
- *portal-images-icp.tgz*

**Note:** The archive file **IBM_API_Connect_ICP_Enterprise_v2018.3.3.zip** can be unzipped to a directory say **/home/admin/downloads**


### 9. Load API Connect images

The following commands are run to load the API Connect images.  

```
cd /home/admin/downloads
ibmcloud pr load-ppa-archive --archive analytics-images-icp.tgz --clustername mycluster.icp --namespace apiconnect
ibmcloud pr load-ppa-archive --archive gateway-images-icp.tgz --clustername mycluster.icp --namespace apiconnect
ibmcloud pr load-ppa-archive --archive ibm-apiconnect-ent.tgz --clustername mycluster.icp --namespace apiconnect
ibmcloud pr load-ppa-archive --archive management-images-icp.tgz --clustername mycluster.icp --namespace apiconnect
ibmcloud pr load-ppa-archive --archive portal-images-icp.tgz --clustername mycluster.icp --namespace apiconnect
```

The output of the aforesaid commands is listed below.

```
[root@rsun-rhel-bootmaster01 admin]# cd /home/admin/downloads
[root@rsun-rhel-bootmaster01 downloads]# ibmcloud pr load-ppa-archive --archive analytics-images-icp.tgz --clustername mycluster.icp --namespace apiconnect
Expanding archive
OK

Importing docker images
  Processing image: apiconnect/openresty:alpine
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/openresty:alpine
Pushing image mycluster.icp:8500/apiconnect/apiconnect/openresty:alpine
  Processing image: apiconnect/analytics-ingestion:2018-07-17-09-35-08-7bab1b4b063a0676a913a75cd4871d9b740ccfc5
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/analytics-ingestion:2018-07-17-09-35-08-7bab1b4b063a0676a913a75cd4871d9b740ccfc5
Pushing image mycluster.icp:8500/apiconnect/apiconnect/analytics-ingestion:2018-07-17-09-35-08-7bab1b4b063a0676a913a75cd4871d9b740ccfc5
  Processing image: apiconnect/analytics-mq-kafka:2018-06-25-21-55-05-5a40eb7c682b8f9c503ab4c49c00ca10c7a34d4b
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/analytics-mq-kafka:2018-06-25-21-55-05-5a40eb7c682b8f9c503ab4c49c00ca10c7a34d4b
Pushing image mycluster.icp:8500/apiconnect/apiconnect/analytics-mq-kafka:2018-06-25-21-55-05-5a40eb7c682b8f9c503ab4c49c00ca10c7a34d4b
  Processing image: apiconnect/analytics-mq-zookeeper:2018-06-25-21-55-05-5a40eb7c682b8f9c503ab4c49c00ca10c7a34d4b
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/analytics-mq-zookeeper:2018-06-25-21-55-05-5a40eb7c682b8f9c503ab4c49c00ca10c7a34d4b
Pushing image mycluster.icp:8500/apiconnect/apiconnect/analytics-mq-zookeeper:2018-06-25-21-55-05-5a40eb7c682b8f9c503ab4c49c00ca10c7a34d4b
  Processing image: apiconnect/analytics-storage:2018-07-17-09-30-13-1ca8b7953e9ce21606168c2b68647cf99b92b675
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/analytics-storage:2018-07-17-09-30-13-1ca8b7953e9ce21606168c2b68647cf99b92b675
Pushing image mycluster.icp:8500/apiconnect/apiconnect/analytics-storage:2018-07-17-09-30-13-1ca8b7953e9ce21606168c2b68647cf99b92b675
  Processing image: apiconnect/analytics-cronjobs:2018-06-25-21-53-38-32c0717d88ef569e2945fd161b1f7992c53552fd
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/analytics-cronjobs:2018-06-25-21-53-38-32c0717d88ef569e2945fd161b1f7992c53552fd
Pushing image mycluster.icp:8500/apiconnect/apiconnect/analytics-cronjobs:2018-06-25-21-53-38-32c0717d88ef569e2945fd161b1f7992c53552fd
  Processing image: apiconnect/analytics-client:2018-07-17-10-20-19-ead606c012ec7204c2811baf69c6165139cef936
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/analytics-client:2018-07-17-10-20-19-ead606c012ec7204c2811baf69c6165139cef936
Pushing image mycluster.icp:8500/apiconnect/apiconnect/analytics-client:2018-07-17-10-20-19-ead606c012ec7204c2811baf69c6165139cef936
OK

Uploading helm charts
  Processing chart: charts/ibm-apiconnect-ent-2.0.8.tgz
  Updating chart values.yaml
  Uploading chart
Loaded helm chart
OK

Synch charts
Synch started
OK

Archive finished processing
[root@rsun-rhel-bootmaster01 downloads]# ibmcloud pr load-ppa-archive --archive gateway-images-icp.tgz --clustername mycluster.icp --namespace apiconnect
Expanding archive
OK

Importing docker images
  Processing image: apiconnect/datapower-api-gateway:7.7.1.1-300826-release
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/datapower-api-gateway:7.7.1.1-300826-release
Pushing image mycluster.icp:8500/apiconnect/apiconnect/datapower-api-gateway:7.7.1.1-300826-release
OK

Uploading helm charts
  Processing chart: charts/ibm-apiconnect-ent-2.0.8.tgz
  Updating chart values.yaml
  Uploading chart
Loaded helm chart
OK

Synch charts
Synch started
OK

Archive finished processing
[root@rsun-rhel-bootmaster01 downloads]#ibmcloud pr load-ppa-archive --archive ibm-apiconnect-ent.tgz --clustername mycluster.icp --namespace apiconnect
Expanding archive
OK

Importing docker images
  Processing image: apiconnect/apiconnect-operator:2018-07-19-09-55-55-1d0c43f772fd3a1c9822260af3682794ed5d4c01
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/apiconnect-operator:2018-07-19-09-55-55-1d0c43f772fd3a1c9822260af3682794ed5d4c01
Pushing image mycluster.icp:8500/apiconnect/apiconnect/apiconnect-operator:2018-07-19-09-55-55-1d0c43f772fd3a1c9822260af3682794ed5d4c01
OK

Uploading helm charts
  Processing chart: charts/ibm-apiconnect-ent-2.0.8.tgz
  Updating chart values.yaml
  Uploading chart
Loaded helm chart
OK

Synch charts
Synch started
OK

Archive finished processing
[root@rsun-rhel-bootmaster01 downloads]# ibmcloud pr load-ppa-archive --archive management-images-icp.tgz --clustername mycluster.icp --namespace apiconnect
Expanding archive
OK

Importing docker images
  Processing image: apiconnect/lur:2018.3-42-453f476
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/lur:2018.3-42-453f476
Pushing image mycluster.icp:8500/apiconnect/apiconnect/lur:2018.3-42-453f476
  Processing image: apiconnect/cassandra-health-check:2018-03-20-00-46-39-master-0-gbb58e73
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/cassandra-health-check:2018-03-20-00-46-39-master-0-gbb58e73
Pushing image mycluster.icp:8500/apiconnect/apiconnect/cassandra-health-check:2018-03-20-00-46-39-master-0-gbb58e73
  Processing image: apiconnect/ui:2018-07-18-22-27-27-2d8cc82fc74955d96735b3a5eb7ec5aedabc54d2
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/ui:2018-07-18-22-27-27-2d8cc82fc74955d96735b3a5eb7ec5aedabc54d2
Pushing image mycluster.icp:8500/apiconnect/apiconnect/ui:2018-07-18-22-27-27-2d8cc82fc74955d96735b3a5eb7ec5aedabc54d2
  Processing image: apiconnect/analytics-proxy:2018-06-21-18-40-34-2018.3-0-gb21ef52
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/analytics-proxy:2018-06-21-18-40-34-2018.3-0-gb21ef52
Pushing image mycluster.icp:8500/apiconnect/apiconnect/analytics-proxy:2018-06-21-18-40-34-2018.3-0-gb21ef52
  Processing image: apiconnect/apim:2018.3-68-d72d0fa
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/apim:2018.3-68-d72d0fa
Pushing image mycluster.icp:8500/apiconnect/apiconnect/apim:2018.3-68-d72d0fa
  Processing image: apiconnect/cassandra-health-check:2018-03-20-00-46-39-master-0-gbb58e73
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/cassandra-health-check:2018-03-20-00-46-39-master-0-gbb58e73
Pushing image mycluster.icp:8500/apiconnect/apiconnect/cassandra-health-check:2018-03-20-00-46-39-master-0-gbb58e73
  Processing image: apiconnect/client-downloads-server:2018-07-18-22-42-21-2018.3-0-gc54cabf
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/client-downloads-server:2018-07-18-22-42-21-2018.3-0-gc54cabf
Pushing image mycluster.icp:8500/apiconnect/apiconnect/client-downloads-server:2018-07-18-22-42-21-2018.3-0-gc54cabf
  Processing image: apiconnect/juhu:2018-07-17-18-25-23-2018.3-0-ga44bda5
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/juhu:2018-07-17-18-25-23-2018.3-0-ga44bda5
Pushing image mycluster.icp:8500/apiconnect/apiconnect/juhu:2018-07-17-18-25-23-2018.3-0-ga44bda5
  Processing image: apiconnect/busybox:latest
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/busybox:latest
Pushing image mycluster.icp:8500/apiconnect/apiconnect/busybox:latest
  Processing image: apiconnect/ldap:2018.3-7-9f27c35
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/ldap:2018.3-7-9f27c35
Pushing image mycluster.icp:8500/apiconnect/apiconnect/ldap:2018.3-7-9f27c35
  Processing image: apiconnect/cassandra-operator:2018-07-16-14-28-13-296d0e231215976a3af6f58158797a4d08a19951
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/cassandra-operator:2018-07-16-14-28-13-296d0e231215976a3af6f58158797a4d08a19951
Pushing image mycluster.icp:8500/apiconnect/apiconnect/cassandra-operator:2018-07-16-14-28-13-296d0e231215976a3af6f58158797a4d08a19951
  Processing image: apiconnect/cassandra:2018-07-16-14-28-13-296d0e231215976a3af6f58158797a4d08a19951
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/cassandra:2018-07-16-14-28-13-296d0e231215976a3af6f58158797a4d08a19951
Pushing image mycluster.icp:8500/apiconnect/apiconnect/cassandra:2018-07-16-14-28-13-296d0e231215976a3af6f58158797a4d08a19951
OK

Uploading helm charts
  Processing chart: charts/ibm-apiconnect-ent-2.0.8.tgz
  Updating chart values.yaml
  Uploading chart
Loaded helm chart
OK

Synch charts
Synch started
OK

Archive finished processing
[root@rsun-rhel-bootmaster01 downloads]#ibmcloud pr load-ppa-archive --archive portal-images-icp.tgz --clustername mycluster.icp --namespace apiconnect
Expanding archive
OK

Importing docker images
  Processing image: apiconnect/portal-db:2018.3-bf7f938738faabe4cc083599a0424a063a031cec-28
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/portal-db:2018.3-bf7f938738faabe4cc083599a0424a063a031cec-28
Pushing image mycluster.icp:8500/apiconnect/apiconnect/portal-db:2018.3-bf7f938738faabe4cc083599a0424a063a031cec-28
  Processing image: apiconnect/portal-dbproxy:2018.3-bf7f938738faabe4cc083599a0424a063a031cec-28
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/portal-dbproxy:2018.3-bf7f938738faabe4cc083599a0424a063a031cec-28
Pushing image mycluster.icp:8500/apiconnect/apiconnect/portal-dbproxy:2018.3-bf7f938738faabe4cc083599a0424a063a031cec-28
  Processing image: apiconnect/portal-admin:2018.3-5096b4cd43fd681098a6aaf98ff2cae5aed5f8a1-134
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/portal-admin:2018.3-5096b4cd43fd681098a6aaf98ff2cae5aed5f8a1-134
Pushing image mycluster.icp:8500/apiconnect/apiconnect/portal-admin:2018.3-5096b4cd43fd681098a6aaf98ff2cae5aed5f8a1-134
  Processing image: apiconnect/portal-web:2018.3-5096b4cd43fd681098a6aaf98ff2cae5aed5f8a1-134
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/portal-web:2018.3-5096b4cd43fd681098a6aaf98ff2cae5aed5f8a1-134
Pushing image mycluster.icp:8500/apiconnect/apiconnect/portal-web:2018.3-5096b4cd43fd681098a6aaf98ff2cae5aed5f8a1-134
  Processing image: apiconnect/openresty:alpine
    Loading Image
    Tagging Image
    Pushing image as: mycluster.icp:8500/apiconnect/apiconnect/openresty:alpine
Pushing image mycluster.icp:8500/apiconnect/apiconnect/openresty:alpine
OK

Uploading helm charts
  Processing chart: charts/ibm-apiconnect-ent-2.0.8.tgz
  Updating chart values.yaml
  Uploading chart
Loaded helm chart
OK

Synch charts
Synch started
OK

Archive finished processing
```

The following commands can be used to verify if the images are loaded correctly.

```
kubectl get images -n apiconnect
```

The screen shot having the output of the aforesaid commands is listed below.

![](./images/verify_load_ppa_archives.png)

If required, the following utility can be run to delete all the images within the namespace *apiconnect*.

[Delete All Images: deleteImages.sh](./utils/deleteImages.sh)


### 10. Install the API Connect Helm chart

Perform the following steps to install API Connect Helm chart

#### 10.1 Extract API Connect Helm Chart

Unarchive the file **downloads/ibm-apiconnect-ent.tgz** to the directory **downloads/apiconnect** to extract the Helm chart **ibm-apiconnect-ent**

**Note:** The value *ibm-apiconnect-ent-2.0.8.tgz* could change based on the version used for the installation.

```
cd ~/downloads
mkdir apiconnect
cd apiconnect
tar -xzf ../ibm-apiconnect-ent.tgz
cd charts
tar -xzf ./ibm-apiconnect-ent-2.0.8.tgz
```

#### 10.2 Update **values.yaml** that suits your environment.

The attached excel document [APIC_Settings.xlsx](./APIC_Settings.xlsx) includes all the configuration parameters collected via *values.yaml*

The attached  *values.yaml* can be used as reference. The following values needs to be updated to suit your environment and build.

  * repository
  * registry
  * tag

Also, *INGRESS_CONTROLLER_IP* should be replaced with the IP address of the node where Ingres controller is running. It will be mostly Proxy node and/or the Master node.

**Note** The *mode* can be set to *standard* when there is a need to deploy typical development environment with high availability.

**Note** When the *mode* is set to *demo* it is possible to use hostpath as the storage provider. The *values.yaml* can be updated to include the value  *hostpath* for *storageClass*. 

~~~
global:
  registry: "mycluster.icp:8500/apiconnect/"
  registrySecret: "apiconnect-icp-secret"
  createCrds: true
  storageClass: "apic.shared.storage"
  mode: demo

# apiconnect-operator
operator:
  arch: amd64
  image: mycluster.icp:8500/apiconnect/apiconnect/apiconnect-operator
  tag: 2018-07-19-09-55-55-1d0c43f772fd3a1c9822260af3682794ed5d4c01
  pullPolicy: IfNotPresent
  helmTlsSecret: "helm-tls-secret"

# management subsystem
management:
  enabled: true
  platformApiEndpoint: "apicplatform.INGRESS_CONTROLLER_IP.nip.io"
  consumerApiEndpoint: "apicconsumer.INGRESS_CONTROLLER_IP.nip.io"
  cloudAdminUiEndpoint: "apiccloud.INGRESS_CONTROLLER_IP.nip.io"
  apiManagerUiEndpoint: "apicmanager.INGRESS_CONTROLLER_IP.nip.io"
  externalCassandraHost: ""
cassandra:
  cassandraClusterSize: "1"
  cassandraMaxMemoryGb: "8"
  cassandraVolumeSizeGb: "16"
cassandraBackup:
  cassandraBackupAuthSecret: ""
  cassandraBackupHost: ""
  cassandraBackupPath: /backups
  cassandraBackupPort: "22"
  cassandraBackupProtocol: sftp
  cassandraBackupSchedule: 0 0 * * *
cassandraPostmortems:
  cassandraPostmortemsAuthSecret: ""
  cassandraPostmortemsHost: ""
  cassandraPostmortemsPath: /cassandra-postmortems
  cassandraPostmortemsPort: "22"
  cassandraPostmortemsProtocol: sftp

# portal subsystem
portal:
  enabled: true
  portalDirectorEndpoint: "apicpadmin.INGRESS_CONTROLLER_IP.nip.io"
  portalWebEndpoint: "apicportal.INGRESS_CONTROLLER_IP.nip.io"
  adminStorageSizeGb: "1"
  backupStorageSizeGb: "5"
  dbLogsStorageSizeGb: "2"
  dbRootPw: root
  dbStorageSizeGb: "12"
  wwwStorageSizeGb: "5"

# analytics subsystem
analytics:
  enabled: true
  analyticsIngestionEndpoint: "apicai.INGRESS_CONTROLLER_IP.nip.io"
  analyticsClientEndpoint: "apicac.INGRESS_CONTROLLER_IP.nip.io"
  coordinatingMaxMemoryGb: "3"
  dataMaxMemoryGb: "4"
  dataStorageSizeGb: "50"
  masterMaxMemoryGb: "8"
  masterStorageSizeGb: "1"

# gateway subsystem
gateway:
  enabled: true
  apiGatewayEndpoint: "apicapi-gateway.INGRESS_CONTROLLER_IP.nip.io"
  gatewayServiceEndpoint: "iapicapic-gateway-director.INGRESS_CONTROLLER_IP.nip.io"
  maxCpu: "2"
  maxMemoryGb: "6"
  v5CompatibilityMode: "on"
  enableTms: "off"
  imageRepository: ""
  imageTag: ""
  imagePullPolicy: IfNotPresent
~~~

#### 10.3 Deploy the Helm chart

Run the following commands to install the API Connect Helm chart.

~~~
rm -fr ~/.helm
helm init --client-only
ibmcloud pr cluster-config mycluster
helm version --tls
helm install --name apic-v2018-3-3 --namespace apiconnect ibm-apiconnect-ent --tls --debug  
~~~

The output of the aforesaid commands is listed below.

~~~
[root@rsun-rhel-bootmaster01 charts]# rm -fr ~/.helm
[root@rsun-rhel-bootmaster01 charts]# helm init --client-only
Creating /root/.helm 
Creating /root/.helm/repository 
Creating /root/.helm/repository/cache 
Creating /root/.helm/repository/local 
Creating /root/.helm/plugins 
Creating /root/.helm/starters 
Creating /root/.helm/cache/archive 
Creating /root/.helm/repository/repositories.yaml 
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com 
Adding local repo with URL: http://127.0.0.1:8879/charts 
$HELM_HOME has been configured at /root/.helm.
Not installing Tiller due to 'client-only' flag having been set
Happy Helming!
[root@rsun-rhel-bootmaster01 charts]# ibmcloud pr cluster-config mycluster
Configuring kubectl: /root/.bluemix/plugins/icp/clusters/mycluster/kube-config
Property "clusters.mycluster" unset.
Property "users.mycluster-user" unset.
Property "contexts.mycluster-context" unset.
Cluster "mycluster" set.
User "mycluster-user" set.
Context "mycluster-context" created.
Switched to context "mycluster-context".

Cluster mycluster configured successfully.

Configuring helm: /root/.helm
Helm configured successfully

OK

[root@rsun-rhel-bootmaster01 charts]# helm version --tls
Client: &version.Version{SemVer:"v2.7.3+icp", GitCommit:"27442e4cfd324d8f82f935fe0b7b492994d4c289", GitTreeState:"dirty"}
Server: &version.Version{SemVer:"v2.7.3+icp", GitCommit:"27442e4cfd324d8f82f935fe0b7b492994d4c289", GitTreeState:"dirty"}
[root@rsun-rhel-bootmaster01 charts]# helm install --name apic-v2018-3-3 --namespace apiconnect ibm-apiconnect-ent --tls --debug 
[debug] Created tunnel using local port: '40463'

[debug] SERVER: "127.0.0.1:40463"

[debug] Original chart version: ""
[debug] Key="/root/.helm/key.pem", Cert="/root/.helm/cert.pem", CA="/root/.helm/ca.pem"

[debug] CHART PATH: /home/admin/downloads/apic/apiconnect/charts/ibm-apiconnect-ent

NAME:   apic-v2018-3-3
REVISION: 1
RELEASED: Wed Aug  1 22:09:00 2018
CHART: ibm-apiconnect-ent-2.0.8
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
analytics:
  analyticsClientEndpoint: apicac.172.16.247.254.nip.io
  analyticsIngestionEndpoint: apicai.172.16.247.254.nip.io
  coordinatingMaxMemoryGb: "3"
  dataMaxMemoryGb: "4"
  dataStorageSizeGb: "50"
  enabled: true
  masterMaxMemoryGb: "8"
  masterStorageSizeGb: "1"
cassandra:
  cassandraClusterSize: "1"
  cassandraMaxMemoryGb: "8"
  cassandraVolumeSizeGb: "16"
cassandraBackup:
  cassandraBackupAuthSecret: ""
  cassandraBackupHost: ""
  cassandraBackupPath: /backups
  cassandraBackupPort: "22"
  cassandraBackupProtocol: sftp
  cassandraBackupSchedule: 0 0 * * *
cassandraPostmortems:
  cassandraPostmortemsAuthSecret: ""
  cassandraPostmortemsHost: ""
  cassandraPostmortemsPath: /cassandra-postmortems
  cassandraPostmortemsPort: "22"
  cassandraPostmortemsProtocol: sftp
gateway:
  apiGatewayEndpoint: apicapi-gateway.172.16.247.254.nip.io
  enableTms: "off"
  enabled: true
  gatewayServiceEndpoint: iapicapic-gateway-director.172.16.247.254.nip.io
  imagePullPolicy: IfNotPresent
  imageRepository: ""
  imageTag: ""
  maxCpu: "2"
  maxMemoryGb: "6"
  v5CompatibilityMode: "on"
global:
  createCrds: true
  mode: demo
  registry: mycluster.icp:8500/apiconnect/
  registrySecret: apiconnect-icp-secret
  storageClass: apic.shared.storage
management:
  apiManagerUiEndpoint: apicmanager.172.16.247.254.nip.io
  cloudAdminUiEndpoint: apiccloud.172.16.247.254.nip.io
  consumerApiEndpoint: apicconsumer.172.16.247.254.nip.io
  enabled: true
  externalCassandraHost: ""
  platformApiEndpoint: apicplatform.172.16.247.254.nip.io
operator:
  arch: amd64
  helmTlsSecret: helm-tls-secret
  image: mycluster.icp:8500/apiconnect/apiconnect/apiconnect-operator
  pullPolicy: IfNotPresent
  tag: 2018-07-19-09-55-55-1d0c43f772fd3a1c9822260af3682794ed5d4c01
portal:
  adminStorageSizeGb: "1"
  backupStorageSizeGb: "5"
  dbLogsStorageSizeGb: "2"
  dbRootPw: root
  dbStorageSizeGb: "12"
  enabled: true
  portalDirectorEndpoint: apicpadmin.172.16.247.254.nip.io
  portalWebEndpoint: apicportal.172.16.247.254.nip.io
  wwwStorageSizeGb: "5"

HOOKS:
---
# apic-v2018-3-3-test
apiVersion: v1
kind: Pod
metadata:
  name: "apic-v2018-3-3-test"
  labels:
    heritage: Tiller
    release: apic-v2018-3-3
    chart: ibm-apiconnect-ent-2.0.8
    app: "apic-v2018-3-3-test"
  annotations:
    "helm.sh/hook": test-success
spec:
  containers:
    - name: "apic-v2018-3-3-test"
      image: alpine
      command: ["sh", "-c", 'apk --no-cache add openssl && wget --no-check-certificate https://apicplatform.172.16.247.254.nip.io/admin']
  restartPolicy: Never
---
# apic-v2018-3-3-ibm-apiconnect-ent-delete-cluster
apiVersion: batch/v1
kind: Job
metadata:
  name: apic-v2018-3-3-ibm-apiconnect-ent-delete-cluster
  labels:
    app: ibm-apiconnect-ent
    chart: ibm-apiconnect-ent-2.0.8
    release: apic-v2018-3-3
    heritage: Tiller
  annotations:
    "helm.sh/hook": pre-delete
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    metadata:
      labels:
        app: ibm-apiconnect-ent
        release: apic-v2018-3-3
      annotations:
        productName: "API Connect Enterprise"
        productID: "1575d1b24f8fef20747d1f85f9358500"
        productVersion: "2018.3.3"
    spec:
      serviceAccountName: apic-v2018-3-3-ibm-apiconnect-ent
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: beta.kubernetes.io/arch
                operator: In
                values:
                  - amd64
      restartPolicy: Never
      imagePullSecrets:
        - name: "apiconnect-icp-secret"
      initContainers:
        - name: "apic-v2018-3-3-delete-cluster"
          image: "mycluster.icp:8500/apiconnect/mycluster.icp:8500/apiconnect/apiconnect/apiconnect-operator:2018-07-19-09-55-55-1d0c43f772fd3a1c9822260af3682794ed5d4c01"
          imagePullPolicy: IfNotPresent
          command: [ "/apicop/initApicOp.sh" ]
          args: [ "v1.10.0+icp-ee", "clean", "--debug", "--clustername=apic-v2018-3-3-apic-cluster", "--namespace=apiconnect" ]
          env:
            - name: HELM_HOME
              value: "/home/apic/.helm"
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          volumeMounts:
          - name: helm-tls
            mountPath: "/home/apic/.helm"
      volumes:
      - name: helm-tls
        secret:
          secretName: helm-tls-secret
          defaultMode: 0644
          items:
            - key: cert.pem
              path: cert.pem
            - key: ca.pem
              path: ca.pem
            - key: key.pem
              path: key.pem
      containers:
        - name: "apic-v2018-3-3-delete-cluster-cr"
          image: "mycluster.icp:8500/apiconnect/mycluster.icp:8500/apiconnect/apiconnect/apiconnect-operator:2018-07-19-09-55-55-1d0c43f772fd3a1c9822260af3682794ed5d4c01"
          imagePullPolicy: IfNotPresent
          command:
            - 'bash'
            - '-c'
            - "kubectl delete apiconnectcluster apic-v2018-3-3-apic-cluster || true"
MANIFEST:

---
# Source: ibm-apiconnect-ent/templates/service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: apic-v2018-3-3-ibm-apiconnect-ent
  labels:
    app: ibm-apiconnect-ent
    chart: ibm-apiconnect-ent-2.0.8
    release: apic-v2018-3-3
    heritage: Tiller
---
# Source: ibm-apiconnect-ent/templates/rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: apic-v2018-3-3-ibm-apiconnect-ent
  labels:
    chart: "ibm-apiconnect-ent-2.0.8"
    app: "apic-v2018-3-3-ibm-apiconnect-ent"
    heritage: "Tiller"
    release: "apic-v2018-3-3"
rules:

- apiGroups:
  - apiextensions.k8s.io
  resources:
  - customresourcedefinitions
  verbs:
  - get
  - create

- apiGroups:
  - ""
  resources:
  - pods
  - pods/portforward
  verbs:
  - "get"
  - "list"
  - "create"
---
# Source: ibm-apiconnect-ent/templates/rbac.yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: apic-v2018-3-3-ibm-apiconnect-ent-cluster-binding
  labels:
    chart: "ibm-apiconnect-ent-2.0.8"
    app: "apic-v2018-3-3-ibm-apiconnect-ent"
    heritage: "Tiller"
    release: "apic-v2018-3-3"
subjects:
- kind: ServiceAccount
  name: apic-v2018-3-3-ibm-apiconnect-ent
  namespace: apiconnect
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: apic-v2018-3-3-ibm-apiconnect-ent
---
# Source: ibm-apiconnect-ent/templates/rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: apic-v2018-3-3-ibm-apiconnect-ent
  labels:
    app: ibm-apiconnect-ent
    chart: ibm-apiconnect-ent-2.0.8
    release: apic-v2018-3-3
    heritage: Tiller
rules:
- apiGroups:
  - apic.ibm.com
  resources:
  - apiconnectclusters
  - apiconnectclusters/finalizers
  verbs:
  - "*"
- apiGroups:
  - ""
  - "batch"
  resources:
  - pods
  - services
  - jobs
  - endpoints
  - persistentvolumeclaims
  - events
  - secrets
  verbs:
  - "*"
---
# Source: ibm-apiconnect-ent/templates/rbac.yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: apic-v2018-3-3-ibm-apiconnect-ent
  labels:
    app: ibm-apiconnect-ent
    chart: ibm-apiconnect-ent-2.0.8
    release: apic-v2018-3-3
    heritage: Tiller
subjects:
- kind: ServiceAccount
  name: apic-v2018-3-3-ibm-apiconnect-ent
  namespace: apiconnect
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: apic-v2018-3-3-ibm-apiconnect-ent
---
# Source: ibm-apiconnect-ent/templates/apic-operator-deployment.yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: apic-v2018-3-3-ibm-apiconnect-ent-operator
  labels:
    app: ibm-apiconnect-ent
    chart: ibm-apiconnect-ent-2.0.8
    release: apic-v2018-3-3
    heritage: Tiller
    component: ibm-apiconnect-ent-operator
spec:
  replicas: 1
  selector:
    matchLabels:
      release: apic-v2018-3-3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: ibm-apiconnect-ent
        chart: ibm-apiconnect-ent-2.0.8
        release: apic-v2018-3-3
        heritage: Tiller
        component: ibm-apiconnect-ent-operator
      annotations:
        productName: "API Connect Enterprise"
        productID: "1575d1b24f8fef20747d1f85f9358500"
        productVersion: "2018.3.3"
    spec:
      serviceAccountName: apic-v2018-3-3-ibm-apiconnect-ent
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: beta.kubernetes.io/arch
                operator: In
                values:
                  - amd64
      imagePullSecrets:
        - name: "apiconnect-icp-secret"
      containers:
        - name: ibm-apiconnect-ent
          image: "mycluster.icp:8500/apiconnect/mycluster.icp:8500/apiconnect/apiconnect/apiconnect-operator:2018-07-19-09-55-55-1d0c43f772fd3a1c9822260af3682794ed5d4c01"
          imagePullPolicy: IfNotPresent
          command: [ "/apicop/initApicOp.sh" ]
          args: [ "v1.10.0+icp-ee", "server", "--debug", "--create-crd=true" ]
          env:
            - name: HELM_HOME
              value: "/home/apic/.helm"
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - containerPort: 1776
          livenessProbe:
            httpGet:
              path: /
              port: 1777
          readinessProbe:
            httpGet:
              path: /
              port: 1777
          resources:
            limits:
              cpu: 100m
              memory: 128Mi
            requests:
              cpu: 100m
              memory: 128Mi
          volumeMounts:
          - name: helm-tls
            mountPath: "/home/apic/.helm"
      volumes:
      - name: helm-tls
        secret:
          secretName: helm-tls-secret
          defaultMode: 0644
          items:
            - key: cert.pem
              path: cert.pem
            - key: ca.pem
              path: ca.pem
            - key: key.pem
              path: key.pem
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        supplementalGroups:
          - 1001
---
# Source: ibm-apiconnect-ent/templates/apic-cluster-create.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "apic-v2018-3-3-ibm-apiconnect-ent-create-cluster"
  labels:
    app: ibm-apiconnect-ent
    chart: ibm-apiconnect-ent-2.0.8
    release: apic-v2018-3-3
    heritage: Tiller
spec:
  template:
    metadata:
      labels:
        app: ibm-apiconnect-ent
        release: apic-v2018-3-3
      annotations:
        productName: "API Connect Enterprise"
        productID: "1575d1b24f8fef20747d1f85f9358500"
        productVersion: "2018.3.3"
    spec:
      serviceAccountName: apic-v2018-3-3-ibm-apiconnect-ent
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: beta.kubernetes.io/arch
                operator: In
                values:
                  - amd64
      restartPolicy: Never
      imagePullSecrets:
        - name: "apiconnect-icp-secret"
      initContainers:
        - name: "apic-v2018-3-3-crd-wait"
          image: "mycluster.icp:8500/apiconnect/mycluster.icp:8500/apiconnect/apiconnect/apiconnect-operator:2018-07-19-09-55-55-1d0c43f772fd3a1c9822260af3682794ed5d4c01"
          imagePullPolicy: IfNotPresent
          command: ["bash", "-c", "until kubectl get crd apiconnectclusters.apic.ibm.com; do echo waiting for crd to register; sleep 2; done;"]
      containers:
        - name: "apic-v2018-3-3-create-cluster"
          image: "mycluster.icp:8500/apiconnect/mycluster.icp:8500/apiconnect/apiconnect/apiconnect-operator:2018-07-19-09-55-55-1d0c43f772fd3a1c9822260af3682794ed5d4c01"
          imagePullPolicy: IfNotPresent
          command:
            - 'bash'
            - '-c'
            - |
              cat <<EOF | kubectl apply -f -
              apiVersion: apic.ibm.com/v1
              kind: APIConnectCluster
              metadata:
                name: apic-v2018-3-3-apic-cluster
              spec:
                subsystems:
                  analytics:
                    endpoints:
                    - hostname: "apicai.172.16.247.254.nip.io"
                      name: analytics-ingestion
                    - hostname: "apicac.172.16.247.254.nip.io"
                      name: analytics-client
                    kvs:
                      coordinating-max-memory-gb: "3"
                      data-max-memory-gb: "4"
                      data-storage-size-gb: "50"
                      enable-persistence: "true"
                      ingress-type: "ingress"
                      master-max-memory-gb: "8"
                      master-storage-size-gb: "1"
                      mode: dev
                      registry: "mycluster.icp:8500/apiconnect/"
                      registry-secret: "apiconnect-icp-secret"
                      storage-class: "apic.shared.storage"
                    extra-values:
                      global: '{ productName: "API Connect Enterprise", productID: "1575d1b24f8fef20747d1f85f9358500", productVersion: "2018.3.3" }'
                  gateway:
                    endpoints:
                    - hostname: "apicapi-gateway.172.16.247.254.nip.io"
                      name: api-gateway
                    - hostname: "iapicapic-gateway-director.172.16.247.254.nip.io"
                      name: apic-gw-service
                    kvs:
                      enable-tms: "off"
                      image-pull-policy: "IfNotPresent"
                      registry: "mycluster.icp:8500/apiconnect/"
                      image-repository: ""
                      image-tag: "latest"
                      ingress-type: "ingress"
                      max-cpu: "2"
                      max-memory-gb: "6"
                      registry-secret: "apiconnect-icp-secret"
                      replica-count: "1"
                      mode: dev
                      storage-class: "apic.shared.storage"
                      tms-peering-storage-size-gb: "10"
                      v5-compatibility-mode: "on"
                    extra-values: {}
                  management:
                    endpoints:
                    - hostname: "apicplatform.172.16.247.254.nip.io"
                      name: platform-api
                    - hostname: "apicconsumer.172.16.247.254.nip.io"
                      name: consumer-api
                    - hostname: "apiccloud.172.16.247.254.nip.io"
                      name: cloud-admin-ui
                    - hostname: "apicmanager.172.16.247.254.nip.io"
                      name: api-manager-ui
                    kvs:
                      cassandra-backup-auth-secret: ""
                      cassandra-backup-host: ""
                      cassandra-backup-path: "/backups"
                      cassandra-backup-port: "22"
                      cassandra-backup-protocol: "sftp"
                      cassandra-backup-schedule: "0 0 * * *"
                      cassandra-cluster-size: "1"
                      cassandra-max-memory-gb: "8"
                      cassandra-postmortems-auth-secret: ""
                      cassandra-postmortems-host: ""
                      cassandra-postmortems-path: "/cassandra-postmortems"
                      cassandra-postmortems-port: "22"
                      cassandra-postmortems-protocol: "sftp"
                      cassandra-volume-size-gb: "16"
                      create-crd: "true"
                      enable-persistence: "true"
                      external-cassandra-host: ""
                      ingress-type: "ingress"
                      migration-image-repository: ""
                      migration-image-tag: ""
                      migration-volume-size-gb: ""
                      mode: dev
                      portal-base-uri: ""
                      registry: "mycluster.icp:8500/apiconnect/"
                      registry-secret: "apiconnect-icp-secret"
                      storage-class: "apic.shared.storage"
                    extra-values: {
                      global: '{ productName: "API Connect Enterprise", productID: "1575d1b24f8fef20747d1f85f9358500", productVersion: "2018.3.3" }'
                    }
                  portal:
                    endpoints:
                    - hostname: "apicpadmin.172.16.247.254.nip.io"
                      name: portal-admin
                    - hostname: "apicportal.172.16.247.254.nip.io"
                      name: portal-www
                    kvs:
                      admin-storage-size-gb: "1"
                      backup-storage-size-gb: "5"
                      db-logs-storage-size-gb: "2"
                      db-root-pw: "root"
                      db-storage-size-gb: "12"
                      enable-persistence: "true"
                      ingress-type: "ingress"
                      mode: dev
                      registry: "mycluster.icp:8500/apiconnect/"
                      registry-secret: "apiconnect-icp-secret"
                      storage-class: "apic.shared.storage"
                      www-storage-size-gb: "5"
                    extra-values: {
                      global: '{ productName: "API Connect Enterprise", productID: "1575d1b24f8fef20747d1f85f9358500", productVersion: "2018.3.3" }'
                    }
              EOF
LAST DEPLOYED: Wed Aug  1 22:09:00 2018
NAMESPACE: apiconnect
STATUS: DEPLOYED

RESOURCES:
==> v1beta2/Deployment
NAME                                        DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
apic-v2018-3-3-ibm-apiconnect-ent-operator  1        1        1           0          0s

==> v1/Job
NAME                                              DESIRED  SUCCESSFUL  AGE
apic-v2018-3-3-ibm-apiconnect-ent-create-cluster  1        0           0s

==> v1/Pod(related)
NAME                                                    READY  STATUS    RESTARTS  AGE
apic-v2018-3-3-ibm-apiconnect-ent-create-cluster-lmbjd  0/1    Init:0/1  0         0s

==> v1/ServiceAccount
NAME                               SECRETS  AGE
apic-v2018-3-3-ibm-apiconnect-ent  1        0s

==> v1beta1/ClusterRole
NAME                               AGE
apic-v2018-3-3-ibm-apiconnect-ent  0s

==> v1beta1/ClusterRoleBinding
NAME                                               AGE
apic-v2018-3-3-ibm-apiconnect-ent-cluster-binding  0s

==> v1beta1/Role
NAME                               AGE
apic-v2018-3-3-ibm-apiconnect-ent  0s

==> v1beta1/RoleBinding
NAME                               AGE
apic-v2018-3-3-ibm-apiconnect-ent  0s


NOTES:

## API Connect Management subsystem ##

Platform API:   https://apicplatform.172.16.247.254.nip.io/api
Consumer API:   https://apicconsumer.172.16.247.254.nip.io/consumer-api
Cloud Admin UI: https://apiccloud.172.16.247.254.nip.io/admin
API Manager UI: https://apicmanager.172.16.247.254.nip.io/manager
## API Connect Portal subsystem ##

Portal Director API:   https://apicpadmin.172.16.247.254.nip.io
Portal Web UI:   https://apicportal.172.16.247.254.nip.io
## API Connect Analytics subsystem ##

Analytics Ingestion API:   https://apicai.172.16.247.254.nip.io
Analytics Client API:   https://apicac.172.16.247.254.nip.io
## API Connect Gateway subsystem ##

Gateway Service API:   https://iapicapic-gateway-director.172.16.247.254.nip.io
API Gateway:   https://apicapi-gateway.172.16.247.254.nip.io
~~~


### 11. Verify API Connect Install

The following command can be used to get all the pods and their states. It will be good to analyze the pods that are NOT in the Running state. If all the pods are not in the *Running* state, the [Troubleshooting tips section](#12-troubleshooting-api-connect-install) can be referred.

~~~
kubectl get pods -n apiconnect
~~~

![](./images/pods_list.png)

The following command can be used to get the details of storage classes used in the system.

~~~
kubectl get sc -n apiconnect
~~~

![](./images/sc_list.png)

The following command can be used to get the details of Persistant Storage Volumes used in the system.

~~~
kubectl get pvc -n apiconnect
~~~

![](./images/pvc_list.png)

The following commands can be used to get details of the current API Connect setup.

~~~
kubectl get services -n apiconnect
~~~

![](./images/service_list.png)

The following command can be used to get the list of installed Helm Charts. IBM API Connect should be listed if it was successfully installed.

~~~
helm list --tls
~~~

![](./images/helm_list.png)

The following commands is used to get details about the API Connect cluster.

~~~
kubectl describe apiconnectclusters -n apiconnect
~~~

The output of the aforesaid commands is listed below.

~~~
[root@rsun-rhel-bootmaster01 charts]# kubectl describe apiconnectclusters -n apiconnect
Name:         apic-v201833-apic-cluster
Namespace:    apiconnect
Labels:       <none>
Annotations:  <none>
API Version:  apic.ibm.com/v1
Kind:         APIConnectCluster
Metadata:
  Cluster Name:        
  Creation Timestamp:  2018-06-04T19:43:59Z
  Generation:          1
  Resource Version:    16079
  Self Link:           /apis/apic.ibm.com/v1/namespaces/apiconnect/apiconnectclusters/apic-v201833-apic-cluster
  UID:                 a038a8fa-682f-11e8-9616-005056a5241f
Spec:
  Secrets:
  Subsystems:
    Analytics:
      Endpoints:
        Hostname:  apicai.172.16.247.254.nip.io
        Name:      analytics-ingestion
        Hostname:  apicac.172.16.247.254.nip.io
        Name:      analytics-client
      Extra - Values:
      Kvs:
        Coordinating - Max - Memory - Gb:  3
        Data - Max - Memory - Gb:          4
        Data - Storage - Size - Gb:        50
        Enable - Persistence:              true
        Ingress - Type:                    ingress
        Master - Max - Memory - Gb:        8
        Master - Storage - Size - Gb:      1
        Mode:                              demo
        Registry:                          mycluster.icp:8500/apiconnect/
        Registry - Secret:                 apiconnect-icp-secret
        Storage - Class:                   apic.shared.storage
      Name:                                
    Gateway:
      Endpoints:
        Hostname:  apicapi-gateway.172.16.247.254.nip.io
        Name:      gateway
        Hostname:  iapicapic-gateway-director.172.16.247.254.nip.io
        Name:      gateway-director
      Extra - Values:
      Kvs:
        Ingress - Type:     ingress
        Max - Cpu:          2
        Max - Memory - Gb:  6
        Mode:               demo
        Registry:           mycluster.icp:8500/apiconnect/
        Registry - Secret:  apiconnect-icp-secret
        Replica - Count:    1
        Storage - Class:    apic.shared.storage
      Name:                 
    Management:
      Endpoints:
        Hostname:  apicplatform.172.16.247.254.nip.io
        Name:      platform-api
        Hostname:  apicconsumer.172.16.247.254.nip.io
        Name:      consumer-api
        Hostname:  apiccloud.172.16.247.254.nip.io
        Name:      cloud-admin-ui
        Hostname:  apicmanager.172.16.247.254.nip.io
        Name:      api-manager-ui
      Extra - Values:
      Kvs:
        Cassandra - Backup - Auth - Secret:       
        Cassandra - Backup - Host:                
        Cassandra - Backup - Path:                /backups
        Cassandra - Backup - Port:                22
        Cassandra - Backup - Protocol:            sftp
        Cassandra - Backup - Schedule:            0 0 * * *
        Cassandra - Cluster - Size:               1
        Cassandra - Max - Memory - Gb:            8
        Cassandra - Postmortems - Auth - Secret:  
        Cassandra - Postmortems - Host:           
        Cassandra - Postmortems - Path:           /cassandra-postmortems
        Cassandra - Postmortems - Port:           22
        Cassandra - Postmortems - Protocol:       sftp
        Cassandra - Volume - Size - Gb:           16
        Create - Crd:                             true
        Enable - Persistence:                     true
        External - Cassandra - Host:              
        Ingress - Type:                           ingress
        Mode:                                     demo
        Portal - Base - Uri:                      
        Registry:                                 mycluster.icp:8500/apiconnect/
        Registry - Secret:                        apiconnect-icp-secret
        Search - Max - Memory - Gb:               2
        Search - Volume - Size - Gb:              50
        Storage - Class:                          apic.shared.storage
      Name:                                       
    Portal:
      Endpoints:
        Hostname:  apicpadmin.172.16.247.254.nip.io
        Name:      portal-admin
        Hostname:  apicportal.172.16.247.254.nip.io
        Name:      portal-www
      Extra - Values:
      Kvs:
        Admin - Storage - Size - Gb:      1
        Backup - Storage - Size - Gb:     5
        Db - Logs - Storage - Size - Gb:  2
        Db - Root - Pw:                   root
        Db - Storage - Size - Gb:         12
        Enable - Persistence:             true
        Ingress - Type:                   ingress
        Mode:                             demo
        Registry:                         mycluster.icp:8500/apiconnect/
        Registry - Secret:                apiconnect-icp-secret
        Storage - Class:                  apic.shared.storage
        Www - Storage - Size - Gb:        5
      Name:                               
Status:
  Ready:  false
  Applied Chart Hash:
    Apic - Analytics:             0aa80909653df1ca5e4ab574d530bda6014c80d06a49782e8d6a910a148221d2
    Apic - Portal:                f96513e50979e5d78cc05d906679d126aca5d52f80a496939b90891d032686c7
    Apiconnect:                   9eb713a85aedd0bdfe617d6ca24f0402387d1b2cd30c12949df9b34a3a7d71c1
    Cassandra - Operator:         85509e64d89eb377f4f9d9d4d19a03b1cf3f4837e37c432d1240f6502e884ce6
    Dynamic - Gateway - Service:  d33f88fe55e5bd0c19ddaac557e4100f9d7a26e285f9dd63e04613901bba45fa
Events:                           <none>
~~~


### 12. Troubleshooting API Connect Install

This section can be used as reference if there are issues when deploying IBM API Connect Helm chart.

#### 12.1 Useful Commands


The following command can be used to get more details about the failing pod.

~~~
kubectl -n apiconnect describe pods <failing_pod_name>
kubectl -n apiconnect logs <failing_pod_name>
~~~

The following command can be used to get more details about the failing *Persistent Storage Volume* that are NOT bound.

~~~
kubectl -n apiconnect describe pvc <pending_pvc>
~~~

If required, the following command can be used to delete the Helm chart and reinstall the IBM API Connect.

~~~
helm delete --purge <old_helm_release>
~~~

#### 12.2 GlusterFS Storage cleanup

If required, the following utilities can be run to cleanup the GlusterFS storage used so far within the namespace **apiconnect**.

- [Delete all Persistent Volume Chains: deletePVCs.sh](./utils/deletePVCs.sh)
- [Delete all GlusterFS Volumes: deleteGlusterFSVolumes.sh](./utils/deleteGlusterFSVolumes.sh)

The Heketi environment must be set using the following commands before running the *deleteVolumes.sh* script.

**Note:** The environment variables *HEKETI_CLI_SERVER*, *HEKETI_CLI_USER*, *HEKETI_CLI_KEY* shoud be updated to match your Heketi environment.

~~~
export HEKETI_CLI_SERVER=http://localhost:8081
export HEKETI_CLI_USER=admin
export HEKETI_CLI_KEY=adminadmin
~~~

The following command can be run on all the GlusterFS servers to cleanup the disk (sdX) :

~~~
wipefs -a /dev/sdX
~~~

The Heketi database can be cleaned up using the following commands:

~~~
rm -fr /var/lib/heketi/*
systemctl enable --now heketi
heketi-cli topology load --json=topology.json
~~~


### 13. Install and Configure SMTP.

**Note** This task is NOT required if an SMTP server is already available.

If SMTP server is NOT available, the following steps can be executed on the Management node to enable **FakeSMTP** as the reference SMTP server.

The following link has details on the SMTP Server setup.

http://nilhcem.com/FakeSMTP/download.html

Also, the following commands can be run to download *java* and install on the Redhat systems.

~~~
yum install java-1.7.0-openjdk-devel
~~~

The following commands are run to enable FakeSMTP after the utility is downloaded.

~~~
mkdir /tmp/emails
java -jar fakeSMTP-1.13.jar -s -b -p 2525 -o /tmp/emails &
~~~


### 14. Login to the Cloud Manager

The login URL is:

https://apiccloud.INGRESS_CONTROLLER_IP.nip.io/admin/

The credentials user name `admin` and password `7iron-hide` can be used to login to API Connect cloud manager.

The screen shot of the home page after logging onto the IBM API Connect Cloud Manager is used is listed below.

![](./images/cloudmanager_home.png)

#### 14.1 Configure eMail server

The following link can be used as a reference to setup the eMail server:

- [Configuring an email server for notifications](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.cmc.doc/config_emailserver.html)

The screen shot having values used in the current setup is listed below.

![](./images/configure_email_server.png)

The following link can be used as reference to configure the *Notification*.

![](./images/configure_notifications.png)

- [Configuring notifications](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.cmc.doc/task_cmc_config_notifications.html)

#### 14.2 Register Gateway service

The following link can be used as reference to set up Gateway Service:

- [Registering a gateway service](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.cmc.doc/config_gateway.html)

The value of **API Endpoint Base** can be set to the value used in the parameter *apiGatewayEndpoint* of *values.yaml*.

The value of **Endpoint** can be set to the value *"https://<DYNAMIC_GATEWAY_SERVICE_INGRESS_NAME>.<NAMESPACE>.svc:3000"*.

**Note:** DYNAMIC_GATEWAY_SERVICE_INGRESS_NAME can be retrieved using the output of the following command:

~~~
kubectl get services -n apiconnect | grep dynamic-gateway-service-ingress | awk -F' ' '{print $1 }'
~~~

The screen shot having values used in the current setup is listed below.

![](./images/configure_gateway_service.png)

#### 14.3 Register Analytics service

The following link can be used as reference to set up Analytics Service:

- [Registering an analytics service](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.cmc.doc/config_analytics.html)

The value of **Endpoint** can be set to the value used in the parameter *analyticsClientEndpoint* of *values.yaml*.

The default *TLS Analytics Client* profile can be chosen.

The screen shot having values used in the current setup is listed below.

![](./images/configure_analytics_service.png)

#### 14.4 Register Portal service

The following link can be used as reference to set up Portal Service:

- [Registering a portal service](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.cmc.doc/config_portal.html)

The value of **Web Endpoint** can be set to the value used in the parameter *portalWebEndpoint* of *values.yaml*.

The value of **Director Endpoint** can be set to the value used in the parameter *portalDirectorEndpoint* of *values.yaml*.

The screen shot having values used in the current setup is listed below.

![](./images/configure_portal_service.png)

#### 14.5 Create Provider Organization

The following link can be used as a reference to setup a Provider Organization. Option #2 listed in the link can be used to invite the new Organization owner.

- [Creating a provider organization account](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.cmc.doc/create_organization.html)

The screen shot having activation link is listed below.

![](./images/create_provider_org.png)


### 15. Login to the API Manager

The Activation link received in the previous section can be used to register the new Provider Organization and logon to API Manager.

The screen shot of the home page after the activation link is used is listed below.

![](./images/apimanager_home.png)

#### 15.1 Configure Default Gateway for the sandbox catalog

The following link can be used as a reference to configure the Gateway for the sandbox catalog.

- [Configuring default gateway services for catalogs](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.cmc.doc/task_cmc_config_catalogDefaults.html)

The screen shot having default gateway is listed below.

![](./images/configure_default_gateway_service.png)

#### 15.2 Import API and Product

**Note:** Sample API [hello_1.0.1.yaml](./samples/hello_1.0.1.yaml) and the Product [samples_1.0.1.yaml](./samples/samples_1.0.1.yaml) can be used for the import.

The following link can be used as a reference to import API and Product.

- [Importing an API](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.apionprem.doc/tutorial_apionprem_import_api.html)

- [Importing an Product](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.apionprem.doc/task_apionprem_upload_product.html)

The screen shot after the import is listed below.

![](./images/import_api.png)
![](./images/import_product.png)

#### 15.3 Publish Product

The following links can be used as a reference to publish an API.

- [Publish a Product](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.apionprem.doc/task_publishing_a_product.html)

The screen shot after the publish is listed below.

![](./images/publish_api.png)

#### 15.4 Test the APIs

After the API is published, the API can be tested from the Assembly and/or by invoking the URL directly.

Results are attached below.

![](./images/test_api_from_assembly.png)
![](./images/test_api_direct.png)

#### 15.5 Create Portal Site

The following link can be used as a reference to create Developer Portal.

- [Creating and configuring Catalogs](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.apionprem.doc/create_env.html)

The screen shot after the creation of Portal is listed below.

![](./images/create_portal1.png)
![](./images/create_portal2.png)

#### 15.6 Create Consumer Organization

The following link can be used as a reference to create Consumer Organization using the "Create Organization" flow.

- [Creating Consumer Organization ](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.apionprem.doc/task_apionprem_create_organization.html)

The screen shot after the creation of Consumer Organization is listed below.

![](./images/create_consumer_org.png)


### 16. Login to the Developer Portal

The login URL is:

https://apicportal.INGRESS_CONTROLLER_IP.nip.io/PROVIDER_ORG_SHORT_NAME/CATALAG_NAME

The credentials of the Consumer Org owner created in the previous section can be be used to login to the Developer Portal.

The screen shot of the home page after logging onto the IBM API Connect Cloud Manager is used is listed below.

![](./images/portal_site.png)


## References

The following links can be used as reference:

- [Deploying in the IBM Cloud Private environment with the catalog](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.install.doc/tapim_icp_installing.html)
- [Installing API Connect into a Kubernetes runtime environment](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.install.doc/tapic_install_Kubernetes.html)
- [Requirements for deploying API Connect into a Kubernetes runtime environment](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.install.doc/tapic_install_reqs_Kubernetes.html)
