# Getting Strated
 The best way to get to know Pravega operator is to start it up and run a sample Pravega operator application.
 
# Running Pravega Operator is simple
 Verify the following prerequisite:

## Build Requirements:
 - [Go 1.1] 
 - [Operator-SDK](https://github.com/operator-framework/operator-sdk)
 
 - The instructions to install Operator SDK can be found [here](  https://github.com/operator-framework/operator-sdk#quick-start)
 
## Usage:
 
 ```bash
 mkdir -p $GOPATH/src/github.com/pravega
 cd $GOPATH/src/github.com/pravega
 git clone git@github.com:pravega/pravega-operator.git
 cd pravega-operator
 ```
 
### Get the operator Docker image

Create the Pravega operator image and force it to the kubernetes cluster.     
 
#### a. Build the image yourself
 
 ```bash
 operator-sdk build pravega/pravega-operator
  ```
```
docker image | grep Operator
``` 
#### b. Use the image from Docker Hub
 
 ```bash
 # No addition steps needed
 ```
#### Installation on Google GKE
 The Operator requires elevated privileges in order to watch for the custom resources.  
 According to Google Container Engine docs:
>Ensure the creation of RoleBinding as it grants all the permissions included in the role that we want to create. Because of the way Container Engine checks permissions when we create a Role or ClusterRole. 

> An example workaround is to create a RoleBinding that gives your Google identity a cluster-admin role before attempting to create additional Role or ClusterRole permissions.
> 
> This is a known issue in the Beta release of Role-Based Access Control in Kubernetes and Container Engine version 1.6.

 On Google GKE the following command must be run before installing the operator, **replacing the user with your own details.**
 
 ```kubectl create clusterrolebinding your-user-cluster-admin-binding --clusterrole=cluster-admin --user=your.google.cloud.email@example.org```
 
### Install the Operator
 
 The operator and required resources can be installed using the yaml files in the deploy directory:

 ```bash
 $ kubectl apply -f deploy
 ```
 View the pravega-operator Pod using the following command
```
 $ kubectl get pod
 NAME                                  READY     STATUS              RESTARTS   AGE
 pravega-operator-6787869796-mxqjv      1/1       Running             0          1m
 ```
  
### Requirements
 
 There are several required components that must exist before the operator can be used to provision a PravegaCluster:
 
#### Zookeeper 
 
 Pravega requires an existing Apache Zookeeper 3.5 cluster as it can be easily deployed using the [Pravega Zookeeper operator](https://github.com/pravega/zookeeper-operator).
   
 The ZookeeperURI for the cluster is provided as part of the PravegaCluster resource.
 
**Note:** The Zookeeper instance can be shared between multiple PravegaCluster instances.

### Usage:
 
```bash
mkdir -p $GOPATH/src/github.com/pravega
cd $GOPATH/src/github.com/pravega
git clone git@github.com:pravega/zookeeper-operator.git
cd zookeeper-operator
```
### Get the operator Docker image
#### a. Build the image yourself
 ```bash
operator-sdk build pravega/zookeeper-operator
```
#### b. Use the image from Docker Hub
```
 No additional steps are required to use the image from Docker Hub.
```
### Install the Kubernetes Resources
#### Deployment
Ensure to enable cluster role bindings if we are running on GKE:
```bash
 $ kubectl create clusterrolebinding your-user-cluster-admin-binding --clusterrole=cluster-admin --user=<your.google.cloud.email@example.org>
```
Install the operator components  and create Operator deployment, Roles, Service Account, and Custom Resource Definition for
a Zookeeper cluster as follows:
```bash
$ kubectl apply -f deploy
```
View the zookeeper-operator Pod using the following commands.
```bash
$ kubectl get pod
NAME                                  READY     STATUS              RESTARTS   AGE
zookeeper-operator-5c7b8cfd85-ttb5g   1/1       Running             0          5m
```
### The Zookeeper Custom Resource
 Using the follwoing `YAML` template, install a 3 node Zookeeper Cluster easily into the Kubernetes cluster:
```bash
apiVersion: "zookeeper.pravega.io/v1beta1"
kind: "ZookeeperCluster"
metadata:
  name: "example"
spec:
zookeeperUri: example-client:2181
  size: 3
```
View the cluster and its components using the following commands:
```bash
$ kubectl get zk
NAME      AGE
example   2s
```
$ kubectl get all -l app=example
NAME            READY     STATUS              RESTARTS   AGE
pod/example-0   1/1       Running             0          51m
pod/example-1   1/1       Running             0          55m
pod/example-2   1/1       Running             0          58m

NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/example-client     ClusterIP   x.x.x.x         <none>        2181/TCP            51m
service/example-headless   ClusterIP   None            <none>        2888/TCP,3888/TCP   51m

#### NFS Storage
The following example uses an NFS PVC provisioned by the [NFS Server Provisioner](https://github.com/kubernetes/charts/tree/master/stable/nfs-server-provisioner) 
helm chart to provide Tier2 storage:

```
helm install stable/nfs-server-provisioner
```
**Note:** This is ONLY intended as a demo and should NOT be used for production deployments.
#### Deployment
 Using the following `YAML` template we can easily install a small development Pravega Cluster (3 Bookies, 1 controller, 3 segmentstore)
 in our Kubernetes cluster. The cluster will be provisioned in to the same namespace as the resource.
  ```yaml
 kind: PersistentVolumeClaim
 apiVersion: v1
 metadata:
   name: pravega-tier2
 spec:
   storageClassName: ""
   storageClassName: "nfs"
   accessModes:
     - ReadWriteMany
   resources:
     requests:
            storage: 50Gi
      ---
      apiVersion: "pravega.pravega.io/v1alpha1"
      kind: "PravegaCluster"
      metadata:
        name: "example"
      spec:
        zookeeperUri: example-client:2181
      
        bookkeeper:
          image:
            repository: pravega/bookkeeper
            tag: 0.3.0
            pullPolicy: IfNotPresent
      
          replicas: 3
      
          storage:
            ledgerVolumeClaimTemplate:
              accessModes: [ "ReadWriteOnce" ]
              storageClassName: "standard"
              resources:
                requests:
                  storage: 10Gi
      
            journalVolumeClaimTemplate:
              accessModes: [ "ReadWriteOnce" ]
              storageClassName: "standard"
              resources:
                requests:
                  storage: 10Gi
      
          autoRecovery: true
      
        pravega:
          controllerReplicas: 1
          segmentStoreReplicas: 3
      
          cacheVolumeClaimTemplate:
            accessModes: [ "ReadWriteOnce" ]
            storageClassName: "standard"
            resources:
              requests:
                storage: 20Gi
      
          image:
            repository: pravega/pravega
            tag: 0.3.0
            pullPolicy: IfNotPresent
      
          tier2:
            filesystem:
              persistentVolumeClaim:
               claimName: pravega-tier2
     ```
     View the cluster instance and its components using the following command:
      ```
 View the cluster instance and its components using the following command.
```
     $ kubectl get PravegaCluster
     NAME      AGE
     example   2h
     ```
```
     $ kubectl get all -l pravega_cluster=example
     NAME                                              READY     STATUS    RESTARTS   AGE
     pod/example-bookie-0                              1/1       Running   0          2h
      pod/example-bookie-1                              1/1       Running   0          2h
      pod/example-bookie-2                              1/1       Running   0          2h
      pod/example-pravega-controller-6f58c4f464-2jg8f   1/1       Running   0          2h
      pod/example-segmentstore-0                        1/1       Running   0          2h
      pod/example-segmentstore-1                        1/1       Running   0          2h
      pod/example-segmentstore-2                        1/1       Running   0          2h
      
      NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)              AGE
      service/example-pravega-controller   ClusterIP   10.39.254.134   <none>        10080/TCP,9090/TCP   2h
      
      NAME                                               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
      deployment.extensions/example-pravega-controller   1         1         1            1           2h
      
      NAME                                                          DESIRED   CURRENT   READY     AGE
      replicaset.extensions/example-pravega-controller-6f58c4f464   1         1         1         2h
      
      NAME                                         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
      deployment.apps/example-pravega-controller   1         1         1            1           2h
      
      NAME                                                    DESIRED   CURRENT   READY     AGE
      replicaset.apps/example-pravega-controller-6f58c4f464   1         1         1         2h
      
      NAME                                    DESIRED   CURRENT   AGE
      statefulset.apps/example-bookie         3         3         2h
      statefulset.apps/example-segmentstore   3         3         2h
  ```    
      There are a few other things here, like a configmap, etc...
      ```
#### NFS: Google Filestore Storage
     Create a Persistent Volume (refer to https://cloud.google.com/filestore/docs/accessing-fileshares)  to provide Tier2 storage:
      ```yaml
     apiVersion: v1
     kind: PersistentVolume
     metadata:
       name: pravega
     spec:
       capacity:
         storage: 1T
       accessModes:
       - ReadWriteMany
       nfs:
         path: /vol1
         server: 10.123.189.202
     ```
      Deploy the persistent volume specification using the following command:
     ```kubectl create -f pv.yaml```

      **Note:** The "10.123.189.202:/vol1" is the Filestore that is created previously, and this is ONLY intended as a demo and should NOT be used for production deployments.

      #### Using The Pravega Instance

     A PravegaCluster instance is only accessible WITHIN the cluster (i.e. no outside access is allowed) using the following endpoint in 
     the PravegaClient:
      ```
     tcp://<cluster-name>-pravega-controller.<namespace>:9090
     ```
     The `REST` management interface is available at:
     ```
     http://<cluster-name>-pravega-controller.<namespace>:10080/
     ```
      
  #### Pravega Configuration
      
      Pravega has many configuration options for setting up metrics, tuning, etc.  The available options can be found
      [here](https://github.com/pravega/pravega/blob/3f5b65084ae17e74c8ef8e6a40e78e61fa98737b/config/config.properties) and are
      expressed through the pravega/options part of the resource specification.  All values must be expressed as Strings.
      
      ```
      spec:
          pravega:
              options:
                metrics.enableStatistics: "true"
                metrics.statsdHost: "telegraph.default"
                metrics.statsdPort: "8125"
      ```

