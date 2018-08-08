# apic2018v3.2

# Install IBM API Connect 2018.3.1 on ICP 2.1.0.3

Bookmark this [https://ibm.box.com/v/APIConnectICP](https://ibm.box.com/v/APIConnectICP)

Note: I will keep this updated as I get feedback from the community. Please periodically refresh this.

## Official guides from Knowledge Center

You might want to check this out in companion to this note.

- [Requirements for deploying API Connect into a Kubernetes runtime environment](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.install.doc/tapic_install_reqs_Kubernetes.html)
- [Installing API Connect into a Kubernetes runtime environment](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.install.doc/tapic_install_Kubernetes.html)
- [Management subsystem settings](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.install.doc/management_subsystem_settings_icp.html)
- [Other subsystem settings](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.install.doc/other_subsystem_settings_icp.html)

## Video

[Installing API Connect 2018.x for Containers](https://youtu.be/pHamBypib1Q)
View this video for an overview of how InstallAssist works. This guidance is provided for ICP so it will be slightly different compared to the video.

## Download the tar files from Passport Advantage / Fix Central

I got most of my files from Fix Central, with the exception of gateway-images-icp.tgz which is in XL.

- [IBM API Connect V2018.3.1 is available](https://www-01.ibm.com/support/docview.wss?uid=ibm10716597)
- [Fix Central](http://www.ibm.com/support/fixcentral/swg/quickorder?parent=ibm%7EWebSphere&product=ibm/WebSphere/IBM+API+Connect&release=2018.2.11&platform=All&function=fixId&fixids=portal-images-kubernetes_v2018.3.1,management-images-kubernetes_v2018.3.1,apicup-linux_v2018.3.1,apic-linux_v2018.3.1,analytics-images-kubernetes_v2018.3.1&includeRequisites=1&includeSupersedes=0&downloadMethod=mget&source=fc)
- For Datapower, please refer to upload notes below to pull directly from docker hub

## Tested on

- [IBM Cloud Private using Vagrant](https://github.com/IBM/deploy-ibm-cloud-private)
- [IBM Cloud Private on Fyre](https://github.ibm.com/kokwai/icponfyre)

Due to resource constraint, we can run api manager and gateway locally.
I have a MacBook Pro 2.7 GHz Intel Core i7 with 16 GB.

## Prepare terminal for IBM Cloud Private

```console
bx pr login --skip-ssl-validation -a https://mycluster.icp:8443 -u admin -p admin -c id-mycluster-account
bx pr cluster-config mycluster
docker login mycluster.icp:8500 -u admin -p admin
```

### Troubleshooting docker login or push

If docker push or login doesn't work, please consider the steps here. [Configuring authentication for the Docker CLI](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0.3/manage_images/configuring_docker_cli.html)

## Storage setup

There are a few options for storage.

### IBM Cloud Private on Fyre

Make sure you have enough storage. The below is the currently used storage for current setup.

```console
pvc-12cf67fd-3862-11e8-a095-00163e01b4f0   13G        RWO            Delete           Bound     apiconnect/db-rc4ea5d1d3e-apic-portal-db-0               glusterfs-storage             13h
pvc-12d75eb3-3862-11e8-a095-00163e01b4f0   3G         RWO            Delete           Bound     apiconnect/dblogs-rc4ea5d1d3e-apic-portal-db-0           glusterfs-storage             13h
pvc-12dff552-3862-11e8-a095-00163e01b4f0   6G         RWO            Delete           Bound     apiconnect/web-rc4ea5d1d3e-apic-portal-www-0             glusterfs-storage             13h
pvc-12e1ab71-3862-11e8-a095-00163e01b4f0   6G         RWO            Delete           Bound     apiconnect/backup-rc4ea5d1d3e-apic-portal-www-0          glusterfs-storage             13h
pvc-12e2c18a-3862-11e8-a095-00163e01b4f0   2G         RWO            Delete           Bound     apiconnect/admin-rc4ea5d1d3e-apic-portal-www-0           glusterfs-storage             13h
pvc-53dbd8b7-3891-11e8-a095-00163e01b4f0   54G        RWO            Delete           Bound     apiconnect/data-re266d79975-analytics-storage-data-0     glusterfs-storage             7h
pvc-53e767ac-3891-11e8-a095-00163e01b4f0   2G         RWO            Delete           Bound     apiconnect/data-re266d79975-analytics-storage-master-0   glusterfs-storage             7h
pvc-933c0485-3749-11e8-a095-00163e01b4f0   11G        RWO            Delete           Bound     apiconnect/data-r31f4a26f5e-apim-elasticsearch-0         glusterfs-storage             1d
pvc-956d1811-3749-11e8-a095-00163e01b4f0   11G        RWO            Delete           Bound     apiconnect/pv-claim-r31f4a26f5e-apiconnect-cc-0          glusterfs-storage             1d
```

### IBM Cloud Private using Vagrant

Might need to provision for analytics-storage-data.

```bash
#execute in 192.168.27.100
mkdir /storage/analytics-storage-data -p
chmod 777 /storage/analytics-storage-data
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
   name: vol-analytics-storage-data
spec:
   capacity:
      storage: 60Gi
   accessModes:
      - ReadWriteOnce
   persistentVolumeReclaimPolicy: Recycle
   nfs:
      path: /storage/analytics-storage-data
      server: 192.168.27.100
EOF
```

### Single node setup

```bash
#get ready folders
mkdir /storage
chmod 777 /storage

mkdir /storage/local01 -p
chmod 777 /storage/local01
mkdir /storage/local02 -p
chmod 777 /storage/local02
mkdir /storage/local03 -p
chmod 777 /storage/local03
mkdir /storage/local04 -p
chmod 777 /storage/local04
mkdir /storage/local05 -p
chmod 777 /storage/local05
mkdir /storage/local06 -p
chmod 777 /storage/local06
mkdir /storage/local07 -p
chmod 777 /storage/local07
mkdir /storage/local08 -p
chmod 777 /storage/local08
mkdir /storage/local09 -p
chmod 777 /storage/local09

#create PV
cat <<EOF | kubectl apply -f -
kind: PersistentVolume
apiVersion: v1
metadata:
  name: vollocal01
  labels:
    type: local
spec:
  capacity:
    storage: 60Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: "/storage/local01"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: vollocal02
  labels:
    type: local
spec:
  capacity:
    storage: 60Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: "/storage/local02"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: vollocal03
  labels:
    type: local
spec:
  capacity:
    storage: 60Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: "/storage/local03"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: vollocal04
  labels:
    type: local
spec:
  capacity:
    storage: 60Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: "/storage/local04"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: vollocal05
  labels:
    type: local
spec:
  capacity:
    storage: 60Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: "/storage/local05"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: vollocal06
  labels:
    type: local
spec:
  capacity:
    storage: 60Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: "/storage/local06"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: vollocal07
  labels:
    type: local
spec:
  capacity:
    storage: 60Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: "/storage/local07"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: vollocal08
  labels:
    type: local
spec:
  capacity:
    storage: 60Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: "/storage/local08"
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: vollocal09
  labels:
    type: local
spec:
  capacity:
    storage: 60Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: "/storage/local09"
---
EOF
```

### Dynamic provisioner

You can set any dynamic provisioner as default storage.

## Storage references

If you need some education on storage

- [Creating a storage class in IBM Cloud Private](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.3/manage_cluster/create_storage_class.html)
- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [Dynamic Volume Provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)

## Install apicup

```bash
chmod 755 apicup-linux
alias apicup=~/apicup-linux_v2018.2.9
chmod 755 apic-linux
alias apic=~/apic-linux_v2018.2.9
```

If Windows, ignore the above and make sure to have apicup.exe in your path.

## Create namespace in IBM Cloud Private

```bash
kubectl create namespace apiconnect
#privileged is required for portal. Best practice might require create a new role and be more selective with permissions.
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: apiconnect-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: privileged
subjects:
- kind: ServiceAccount
  name: default
  namespace: apiconnect
EOF
```

### Create via ICP Console

- [Creating a namespace](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.3/user_management/create_project.html)

## Create registry secret

```bash
kubectl create secret docker-registry apiconnect-icp-secret --docker-server=mycluster.icp:8500 --docker-username=admin --docker-password=admin --docker-email=admin@admin.com --namespace apiconnect
```

## Setup helm

You can use the steps from [USING ICP HELM W/ TLS ENABLED](http://icp-content-playbook.rch.stglabs.ibm.com/using-icp-helm-w-tls-enabled/) OR steps below

```bash
docker run -e LICENSE=accept --net=host -v ~/bin:/data ibmcom/icp-helm-api:1.0.0 cp /usr/src/app/public/cli/linux-amd64/helm /data
echo $PATH
#Initialize your Helm CLI.
~/bin/helm init --client-only --skip-refresh
#Workaround for ICP 2.1.0.3
mv ~/bin/helm ~/bin/helm-icp
wget https://www.datsi.fi.upm.es/~frosal/sources/shc-3.8.9b.tgz
tar xvfz shc-3.8.9b.tgz
cd shc-3.8.9b
make
echo -e '#!/bin/bash\n '$(echo $HOME)'/bin/helm-icp "$@" --tls\n' | cat > helm.sh
./shc -f helm.sh
mv helm.sh.x ~/bin/helm
sudo mv ~/bin/helm /usr/local/bin/helm
#Verify that the Helm CLI is initialized
helm version
helm list
```

## Upload the tar files to the image registry

```bash
# version 2018.2.9
apicup registry-upload management management-images-kubernetes_v2018.3.1.tgz mycluster.icp:8500 --accept-license --debug
apicup registry-upload portal portal-images-kubernetes_v2018.3.1.tgz mycluster.icp:8500 --accept-license --debug
apicup registry-upload analytics analytics-images-kubernetes_v2018.3.1.tgz mycluster.icp:8500 --accept-license --debug

# pull datapower images from docker hub if it is online install
docker pull ibmcom/datapower:7.7.1.1.300826
docker tag ibmcom/datapower:7.7.1.1.300826 mycluster.icp:8500/apiconnect/datapower-api-gateway:7.7.1.1-300826-release
docker push mycluster.icp:8500/apiconnect/datapower-api-gateway:7.7.1.1-300826-release
# workaround if it is offline install, use the bx pr load-ppa-archive to load the images to local, then use apicup to push to registry
# bx pr load-ppa-archive --archive gateway-images-icp.tgz -namespace apiconnect
# apicup registry-upload gateway gateway-images-icp.tgz mycluster.icp:8500 --accept-license --debug
```

## Verify images are loaded correctly

```bash
kubectl get images -n apiconnect
```

## First add a management service

```bash
mkdir ./myProject
cd ./myProject
apicup init
apicup subsys create mgmt management --k8s
apicup endpoints set mgmt platform-api apicplatform.$(kubectl cluster-info | grep "Kubernetes master" | sed -e "s/[^/]*\/\/\([^@]*@\)\?\([^:/]*\).*/\2/").nip.io
apicup endpoints set mgmt api-manager-ui apicmanager.$(kubectl cluster-info | grep "Kubernetes master" | sed -e "s/[^/]*\/\/\([^@]*@\)\?\([^:/]*\).*/\2/").nip.io
apicup endpoints set mgmt cloud-admin-ui apiccloud.$(kubectl cluster-info | grep "Kubernetes master" | sed -e "s/[^/]*\/\/\([^@]*@\)\?\([^:/]*\).*/\2/").nip.io
apicup endpoints set mgmt consumer-api apicconsumer.$(kubectl cluster-info | grep "Kubernetes master" | sed -e "s/[^/]*\/\/\([^@]*@\)\?\([^:/]*\).*/\2/").nip.io
apicup subsys set mgmt registry mycluster.icp:8500
apicup subsys set mgmt namespace apiconnect
apicup subsys set mgmt registry-secret apiconnect-icp-secret
apicup subsys set mgmt cassandra-max-memory-gb 16
apicup subsys set mgmt cassandra-cluster-size 1
apicup subsys set mgmt cassandra-volume-size-gb 16
#apicup subsys set mgmt search-max-memory-gb 2
#apicup subsys set mgmt search-volume-size-gb 10
#apicup subsys set mgmt portal-base-uri http://apicportal.$(kubectl cluster-info | grep "Kubernetes master" | sed -e "s/[^/]*\/\/\([^@]*@\)\?\([^:/]*\).*/\2/").nip.io
apicup subsys set mgmt mode dev
##############################################################
## Skip the next line if you setup storage earlier
##############################################################
#apicup subsys set mgmt storage-class $(kubectl get storageclass -o yaml | grep name: | awk '{ print $2}')

apicup subsys install mgmt --debug
```

### Making sure mgmt subsys is available

It is better to ensure the subsys is initialised and running properly before we continue. Otherwise there might be too much contention for resources.

```bash
#make sure all PVC have status Bound
kubectl -n apiconnect get pvc

#make sure all pod running
kubectl -n apiconnect get pod

#you can describe pod if not running
kubectl -n apiconnect describe pod <pod-name>

#you can log for more information
kubectl -n apiconnect log <pod-name>
```

## Login to API Cloud

Once everything is ready, you can login with default user name `admin` and password `7iron-hide`

### To get the login url

```bash
helm status $(helm list | grep apiconnect-2 | awk '{print $1}') | grep apiconnect-cm-ui | awk '{print "http://"$2}'
```

### Configure email

You can use email providers like Gmail or Sendgrid
[Configuring an email server for notifications](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.cmc.doc/config_emailserver.html)

## Configure a Gateway Service Endpoint and API Endpoint in DataPower

Enable gateway web ui

```bash
cat > dgw-extras.yaml <<EOF
datapower:
  # Gateway MGMT variables
  # This value should either be 'enabled' or 'disabled'. Default is disabled
  webGuiManagementState: "enabled"
  webGuiManagementPort: 9090
  webGuiManagementLocalAddress: 0.0.0.0
  # This value should either be 'enabled' or 'disabled'. Default is disabled
  gatewaySshState: "enabled"
  gatewaySshPort: 9022
  gatewaySshLocalAddress: 0.0.0.0
  # This value should either be 'enabled' or 'disabled'. Default is disabled
  restManagementState: "enabled"
  restManagementPort: 5554
  restManagementLocalAddress: 0.0.0.0
EOF
```

```bash
apicup subsys create gwy gateway --k8s
apicup endpoints set gwy api-gateway gw.$(kubectl cluster-info | grep "Kubernetes master" | sed -e "s/[^/]*\/\/\([^@]*@\)\?\([^:/]*\).*/\2/").nip.io
apicup endpoints set gwy apic-gw-service gwd.$(kubectl cluster-info | grep "Kubernetes master" | sed -e "s/[^/]*\/\/\([^@]*@\)\?\([^:/]*\).*/\2/").nip.io
apicup subsys set gwy namespace apiconnect
apicup subsys set gwy registry mycluster.icp:8500
apicup subsys set gwy registry-secret apiconnect-icp-secret
apicup subsys set gwy replica-count 3
apicup subsys set gwy max-cpu 4
apicup subsys set gwy max-memory-gb 6
apicup subsys set gwy v5-compatibility-mode off
apicup subsys set gwy enable-tms on
apicup subsys set gwy mode dev

apicup subsys set gwy extra-values-file dgw-extras.yaml

apicup subsys install gwy --debug
```

### Configure gateway

```bash
######################################################################################
## Create a gateway service. Navigate to `Topology -> Register Service -> Gateway`.
##
## For SNI, select the default TLS profile
## SNI host = *
## SNI TLS server profile = default
######################################################################################
#API Endpoint Base
kubectl get ingress -n apiconnect | grep dynamic-gateway-service-gw | awk '{print "https://"$2}'
#Endpoint
kubectl get svc -n apiconnect | grep dynamic-gateway-service-ingress | awk '{split($5, var1, "/");print "https://"$1".apiconnect:"var1[1]}'
```

[Registering a gateway service](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.cmc.doc/config_gateway.html)

## [Creating a provider organization account](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.cmc.doc/create_organization.html)

Refer to link. I usually perform option 2.

## [Creating and configuring Catalogs](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.apionprem.doc/create_env.html)

Refer to link. You can skip the analytics and portal portion at this step as it is not installed yet.

## Simple tutorial to test API functionality

[Tutorial: Creating a proxy REST API definition](
https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.apionprem.doc/tutorial_apionprem_apiproxy.html)

## Install analytics subsys

```bash
apicup subsys create analytics analytics --k8s
apicup endpoints set analytics analytics-ingestion apicai.$(kubectl cluster-info | grep "Kubernetes master" | sed -e "s/[^/]*\/\/\([^@]*@\)\?\([^:/]*\).*/\2/").nip.io
apicup endpoints set analytics analytics-client apicac.$(kubectl cluster-info | grep "Kubernetes master" | sed -e "s/[^/]*\/\/\([^@]*@\)\?\([^:/]*\).*/\2/").nip.io
apicup subsys set analytics registry mycluster.icp:8500
apicup subsys set analytics namespace apiconnect
apicup subsys set analytics registry-secret apiconnect-icp-secret
apicup subsys set analytics coordinating-max-memory-gb 6
apicup subsys set analytics data-max-memory-gb 8
apicup subsys set analytics data-storage-size-gb 50
apicup subsys set analytics master-max-memory-gb 8
apicup subsys set analytics master-storage-size-gb 1
apicup subsys set analytics mode dev
##############################################################
## Skip the next line if you setup storage earlier
##############################################################
#apicup subsys set analytics storage-class $(kubectl get storageclass -o yaml | grep name: | awk '{ print $2}')


// OPTIONAL: Write the configuration to an output file to inspect myProject/apiconnect-up.yaml prior to installation
apicup subsys install analytics --out analytics-out
apicup subsys install analytics --plan-dir ./myProject/analytics-out

//If output file is not used, enter command below to start the installation
apicup subsys install analytics --debug
```

### Configure analytics

[Registering an analytics service](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.cmc.doc/config_analytics.html)

```bash
######################################################################################
## Create a analytics service. Navigate to `Topology -> Register Service -> analytics`.
######################################################################################
#Endpoint
kubectl get svc -n apiconnect | grep analytics-mtls-gw | awk '{split($5, var1, "/"); print "https://"$1".apiconnect.svc:"var1[1]}'
```

## Add a portal subsys

```bash
apicup subsys create ptl portal --k8s
apicup endpoints set ptl portal-admin apicpadmin.$(kubectl cluster-info | grep "Kubernetes master" | sed -e "s/[^/]*\/\/\([^@]*@\)\?\([^:/]*\).*/\2/").nip.io
apicup endpoints set ptl portal-www apicportal.$(kubectl cluster-info | grep "Kubernetes master" | sed -e "s/[^/]*\/\/\([^@]*@\)\?\([^:/]*\).*/\2/").nip.io
apicup subsys set ptl registry mycluster.icp:8500
apicup subsys set ptl namespace apiconnect
apicup subsys set ptl registry-secret apiconnect-icp-secret
apicup subsys set ptl www-storage-size-gb     5
apicup subsys set ptl backup-storage-size-gb  5
apicup subsys set ptl db-storage-size-gb      12
apicup subsys set ptl db-logs-storage-size-gb 2
apicup subsys set ptl mode dev

// OPTIONAL: Write the configuration to an output file to inspect myProject/apiconnect-up.yaml prior to installation
apicup subsys install ptl --out portal-out
apicup subsys install ptl --plan-dir ./myProject/portal-out

//If output file is not used, enter command below to start the installation
apicup subsys install ptl --debug
```

### Troubleshoot portal service

Portal usually takes a bit longer to startup even when Install Assist completed.

```bash
#Use below command to monitor portal startup
kubectl -n apiconnect log $(kubectl -n apiconnect get pod --selector=app=rc4ea5d1d3e-apic-portal-www -o jsonpath='{.items[0].metadata.name}') -c admin -f
# wait until you see the following appear in the log
# Starting portal rest server in secure mode
# The portal rest server is listening on https://localhost:3009
```

### Configure a portal service

```bash
######################################################################################
## Create a portal service. Navigate to `Topology -> Register Service -> portal`.
######################################################################################
#Director Endpoint
kubectl get svc -n apiconnect | grep apic-portal-director | awk '{split($5, var1, "/");print "https://"$1".apiconnect.svc:"var1[1]}'
#Web Endpoint
kubectl get ingress -n apiconnect | grep apic-portal-web | awk '{print "https://"$2}'
helm status $(helm list | grep apic-portal) | grep apic-portal-web | awk '{print "http://"$2}'
```

[Registering a portal service](https://www.ibm.com/support/knowledgecenter/en/SSMNED_2018/com.ibm.apic.cmc.doc/config_portal.html)

## Sample apiconnect-up.yaml

```yaml
kind: apiconnect-up
subsystems:
  analytics:
    endpoints:
    - hostname: apicai.9.30.255.233.nip.io
      name: analytics-ingestion
    - hostname: apicac.9.30.255.233.nip.io
      name: analytics-client
    kvs:
      coordinating-max-memory-gb: "6"
      data-max-memory-gb: "8"
      data-storage-size-gb: "50"
      enable-persistence: "true"
      extra-values-file: ""
      ingress-type: ingress
      master-max-memory-gb: "8"
      master-storage-size-gb: "1"
      mode: dev
      namespace: apiconnect
      registry: mycluster.icp:8500
      registry-secret: apiconnect-icp-secret
      storage-class: '-'
    target: k8s
    type: analytics
  gwy:
    endpoints:
    - hostname: gw.9.30.255.233.nip.io
      name: api-gateway
    - hostname: gwd.9.30.255.233.nip.io
      name: apic-gw-service
    kvs:
      enable-tms: "on"
      extra-values-file: dgw-extras.yaml
      image-pull-policy: Always
      image-repository: ""
      image-tag: latest
      ingress-type: ingress
      max-cpu: "4"
      max-memory-gb: "6"
      mode: dev
      namespace: apiconnect
      registry: mycluster.icp:8500
      registry-secret: apiconnect-icp-secret
      replica-count: "3"
      storage-class: '-'
      tms-peering-storage-size-gb: "10"
      v5-compatibility-mode: "off"
    target: k8s
    type: gateway
  mgmt:
    endpoints:
    - hostname: apicplatform.9.30.255.233.nip.io
      name: platform-api
    - hostname: apicconsumer.9.30.255.233.nip.io
      name: consumer-api
    - hostname: apiccloud.9.30.255.233.nip.io
      name: cloud-admin-ui
    - hostname: apicmanager.9.30.255.233.nip.io
      name: api-manager-ui
    kvs:
      cassandra-backup-auth-secret: ""
      cassandra-backup-host: ""
      cassandra-backup-path: /backups
      cassandra-backup-port: "22"
      cassandra-backup-protocol: sftp
      cassandra-backup-schedule: 0 0 * * *
      cassandra-cluster-size: "1"
      cassandra-max-memory-gb: "16"
      cassandra-postmortems-auth-secret: ""
      cassandra-postmortems-host: ""
      cassandra-postmortems-path: /cassandra-postmortems
      cassandra-postmortems-port: "22"
      cassandra-postmortems-protocol: sftp
      cassandra-volume-size-gb: "16"
      create-crd: "true"
      enable-persistence: "true"
      external-cassandra-host: ""
      extra-values-file: ""
      ingress-type: ingress
      migration-image-repository: ""
      migration-image-tag: ""
      migration-volume-size-gb: ""
      mode: dev
      namespace: apiconnect
      portal-base-uri: http://myportal.mycompany.com
      registry: mycluster.icp:8500
      registry-secret: apiconnect-icp-secret
      storage-class: '-'
    target: k8s
    type: management
  ptl:
    endpoints:
    - hostname: apicpadmin.9.30.255.233.nip.io
      name: portal-admin
    - hostname: apicportal.9.30.255.233.nip.io
      name: portal-www
    kvs:
      admin-storage-size-gb: "1"
      backup-storage-size-gb: "5"
      db-logs-storage-size-gb: "2"
      db-root-pw: root
      db-storage-size-gb: "12"
      enable-persistence: "true"
      extra-values-file: ""
      ingress-type: ingress
      mode: dev
      namespace: apiconnect
      registry: mycluster.icp:8500
      registry-secret: apiconnect-icp-secret
      storage-class: '-'
      www-storage-size-gb: "5"
    target: k8s
    type: portal
version: 3.0.0+build.2018.3-132.time.2018-07-05T22-29-10Z.commit.42d0836
```
