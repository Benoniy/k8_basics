# Kubernetes Basics
## Contents:  
1. [What is Kubenetes?](https://github.com/Benoniy/k8_basics#what-is-kubernetes-k8)
    1. [Kubectl cheat sheet](https://github.com/Benoniy/k8_basics#useful-cli-based-commands)
    2. [Running YAML files](https://github.com/Benoniy/k8_basics#running-yaml-files)


2. [YAML for pods](https://github.com/Benoniy/k8_basics#yaml-for-pods)
    1. [Liveness Probes](https://github.com/Benoniy/k8_basics#liveness-probes)
    2. [Readiness Probes](https://github.com/Benoniy/k8_basics#readiness-probe)


3. [Deployments](https://github.com/Benoniy/k8_basics#deployments)
    1. [Basic Deployment](https://github.com/Benoniy/k8_basics#basic-deployment-yaml)
    2. [More advanced Deployment](https://github.com/Benoniy/k8_basics#a-more-complete-deployment)


4. [Services](https://github.com/Benoniy/k8_basics#services)
    1. [Service Types](https://github.com/Benoniy/k8_basics#service-types)
    2. [Kubectl for Services](https://github.com/Benoniy/k8_basics#kubectl-for-services)
    3. [YAML for Services](https://github.com/Benoniy/k8_basics#yaml-for-services)


5. [Storage Options](https://github.com/Benoniy/k8_basics#storage-options)
    1. [Volumes](https://github.com/Benoniy/k8_basics#volumes)
        1. [YAML for emptyDir](https://github.com/Benoniy/k8_basics#yaml-for-emptydir)
        2. [YAML for hostPath](https://github.com/Benoniy/k8_basics#yaml-for-hostpath)
        3. [YAML for Cloud](https://github.com/Benoniy/k8_basics#yaml-for-cloud)
    2. [Persistent Volumes (PV) and Persistent volume claim's (PVC)](https://github.com/Benoniy/k8_basics#persistent-volumes-pv-and-persistent-volume-claims-pvc)
        1. [YAML for PV's](https://github.com/Benoniy/k8_basics#yaml-for-pvs)
        2. [YAML for PVC's](https://github.com/Benoniy/k8_basics#yaml-for-pvcs)


## What is Kubernetes (K8)?
* Orchestration of containers  

* Deployment separated from infrastructure  

* Nodes contain Pods, Pods contain Containers  


### Useful CLI based commands:  
* `Set-Alias -Name kube -Value kubectl`  

* `kubectl run <pod_name> --image=<docker_img>` - Create and run a pod  

* `kubectl port-forward <pod_name> <external>:<internal>` - Forward a port temporarily  

* `kubectl delete <resource> <name>` - Delete resource  
	* `pod`
  * `deployment`

* `kubectl get <resource>` - Get a list of existing resources  
	* `all`  
	* `pods`  
  * `deployments`  
    * `--show-labels`
    * `-l <key>=<value>` - Filter by label
	* `services`  

* `kubectl describe <resource> <name>` - Get information about a resource    

* `kubectl exec <pod_name> -it sh` - SSH into a pod  

* `kubectl scale <resource> <name> --replicas=<amount>`  
  * `pod`
  * `deployment`


### Running YAML files:  
* `kubectl create -f <yml_file>` - Create what is defined in the yaml file  
	* `--save-config` - Saves metadata needed to run apply later on  
	* `--dry-run=client` - Create and then instantly destroy  
	* `--validate=<bool>` - Checks yaml syntax  

* `kubectl apply -f <yml_file>` - Create what is defined in the yaml file or update the pods if they exist   

* `kubectl delete -f <yml_file>` - Delete what is defined in the yaml file  

* `kubectl edit -f <yml_file>` - Edit the yaml file  


## YAML for Pods:  

For information on mounting volumes see the YAML in the [volumes](https://github.com/Benoniy/k8_basics#volumes) section.


### Basic pod yaml:  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers: # This section is universal to deployment's and pod's
  - name: my-nginx
    image: nginx:alpine

```  


### Liveness Probe's:
***Checks to see if the pod needs to be fixed/replaced***


* `HTTP GET` - See if a webpage is reachable:  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers:
  - name: my-nginx
    image: nginx:alpine

    # Define the Liveness Probe
    livenessProbe:
    	httpGet: # Check <local>:80/index.html
    	  path: /index.html
        port: 80
      initialDelaySeconds: 15 # Delay for startup
      timeoutSeconds: 2 # Fail after 2 seconds
      periodSeconds: 5 # Perform this every 5 seconds
      failureThreshold: 1 # It can fail once

```


* `EXEC` - Run commands, if they fail its dead:  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers:
  - name: my-nginx
    image: nginx:alpine


    # Define some commands to run on launch
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30;
      rm -rf /tmp/healthy; sleep 600

    # Define the Liveness Probe
    livenessProbe:
      exec: # Execute some commands in the container
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5 # Delay for startup
      periodSeconds: 5 # Perform this every 5 seconds

```


### Readiness probe:
***Checks to see if the pod is ready to be used.***  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers:
  - name: my-nginx
    image: nginx:alpine

    # Define the readiness Probe
    readinessProbe:
      httpGet: # Check <local>:80/index.html
        path: /index.html
        port: 80
      initialDelaySeconds: 2 # Delay for startup
      periodSeconds: 5 # Perform this every 5 seconds

```


## Deployments:  
* A declarative way of managing pods using replica sets 

* A replica set controls pod's  
  * Manages amount of pods  
  * Provide self healing  
  * Provide fault tolerance  
  * Can scale pods horizontally  
  * Uses a pod template  

* Deployments wrap replica set's  
  * Scales replica sets  
  * Supports zero downtime updates  
  * Rollback functionality  
  * Creates labels for pods and sets  


## YAML for deployments:  

### Basic deployment YAML:  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
spec:
  selector: # Select pod template labels
  template: # Define a template
    spec:
      containers:
      - name: my-nginx
        image: nginx:alpine

```


### A more complete deployment:
```yaml
apiVersion: apps/v1
kind: Deployment

# Describes the deployment
metadata:
  name: frontend
  labels:
    app: my-nginx
    tier: frontend

spec:
  #replicas: <amount> # Scale this deployment

  # Selects the template that has the label 'tier: frontend'
  selector:
    matchLabels:
      tier: frontend
  
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      # Here you can define a pod like before
      containers:
      - name: my-nginx
        image: nginx:alpine
        livenessProbe: # You can even add liveness checks
          httpGet:
          path: /index.html
          port: 80
        initialDelaySeconds: 15
        timeoutSeconds: 2
        periodSeconds: 5
        failureThreshold: 1

```


## Services:  
* A single point of entry for accessing pods  
* Helps us avoid IP address hell  
* Services have IP's to access them  
* They remember the IP of any pod assigned to them  
* Function as a load balancer  


### Service Types:  
* `ClusterIP` - Service can communicate with internal IP's (pods)  
  * Default  
  * Only pod's inside the cluster can talk to the service  


* `NodePort` - Service is exposed on a node at a static port  
  * Automatically allocates a port  
  * Allows external callers to access the node using a port  


* `LoadBalancer` - Service has an external IP and acts as a load balancer  
  * Exists outside of the nodes  
  * Splits traffic between `NodePort` services  
  * Can combine with cloud provider load balancers


* `ExternalName` - Maps a service to a DNS  
  * Allows access to an external service  
  * Assigns a Name to an external resource  


### Kubectl for services:  

* `kubectl port-forward <pod_name> <external>:<internal>`    
  * Creates a temporary service that allows port access  
  * Basically creates a NodePort service  


### YAML for services:  

```yaml
apiVersion: apps/v1
kind: Service

# Describes the service
metadata:
  name: my-nginx # This name is used as a DNS for the service <name>:<port>
  labels:
    app: my-nginx

spec:
  # What type? (ClusterIp | NodePort | LoadBalancer | ExternalName)
  # type: # Type Here

  # externalName: api.something.com # This is used if type is ExternalName

  # What Pod template should it use?
  selector:
    app: my-nginx 

  # What is the Container target port and service port?
  ports: 
  # ClusterIP setup
  - name: http
    port: 80
    targetPort: 80

  # NodePort setup
  #- port: 80
  #  targetPort: 80
  #  nodePort: 31000 # This is optional

  # LoadBalancer setup
  #- port: 80 # This is the external port
  #  targetPort: 80

  # ExternalName setup
  #- port: 9000

```


## Storage Options:  
* Can be used to store data for access by pods  
* Avoids data loss as the pod filesystem is ephemeral  
* One pod can have multiple volumes  
* Containers user a mountPath to access volumes (Like plugging a USB into a PC)  
* There are four types  
  - Volumes  
  - PersistentVolumes  
  - PersistentVolumeClaims  
  - StorageClasses  


### Volumes:  
* There are 7 types of volume:
  * `emptyDir`
    * Exists inside a pod's file system
    * Only exists as long as the pod is alive  
    * Is useful for inter-container file sharing  
  * `hostPath`
    * Exists inside a node's file system
    * if the node dies so does the volume
  * `nfs`
    * Exists as a separate networked volume
  * `configMap/secret`
    * Provide a Pod with access to K8 resources
  * `persistentVolumeClaim`
    * A persistent storage option
    * Abstracted from the details of where the volume is  
  * `Cloud`
    * A cluster wide storage option  
* Volumes are defined when you create Pods


#### YAML for emptyDir: 
```yaml
apiVersion: v1
kind: Pod
spec:
  # Here is where we define volumes
  volumes:
    - name: html
      emptyDir: {}


  # Define our containers normally
  containers:
  - name: my-nginx
    image: nginx:alpine

    # Here is where we mount volumes to containers
    volumeMounts:
      - name: html # Name of the volume we want to mount
        mountPath: /usr/share/nginx/html # Internal location that we want to mount the volume to
        readOnly: true


  - name: html-updater
    image: alpine
    command: ["/bin/sh", "-c"]
    args:
      - while true: do date >> /html/index.html;
          sleep 10; done

    # Multiple containers can mount the same volume
    volumeMounts:
      - name: html # Name of the volume we want to mount
        mountPath: /html # Internal location that we want to mount the volume to

```


#### YAML for hostPath: 
```yaml
apiVersion: v1
kind: Pod
spec:
  # Here is where we define volumes
  volumes:
    - name: docker-socket
      hostPath:
        path: /var/run/docker.sock
        type: Socket


  # Define our containers normally
  containers:
  - name: docker
    image: docker
    command: ["sleep"]
    args : ["1000"]

    # Here is where we mount volumes to containers
    volumeMounts:
      - name: docker-socket # Name of the volume we want to mount
        mountPath: /var/run/docker.sock # Internal location that we want to mount the volume to

```


#### YAML for Cloud:
```yaml
apiVersion: v1
kind: Pod
spec:
  # Here is where we define volumes
  volumes:
    - name: cloud-volume

      # AWS
      awsElasticBlockStore:
        volumeID: <volumeID>
        fsType: ext4

      # # Azure
      # azureFile:
      #   secretName: <secret>
      #   shareName: <shareName>
      #   readOnly: false

      # # Google Cloud Platform
      # gcePersistentDisk:
      #   pdName: <pdName>
      #   fsType: ext4


  # Define our containers normally
  containers:
  - name: cloud-app
    image: image
    # Here is where we mount volumes to containers
    volumeMounts:
      - name: cloud-volume # Name of the volume we want to mount
        mountPath: /data/storage # Internal location that we want to mount the volume to

```


### Persistent Volumes (PV) and Persistent volume claim's (PVC):
* A cluster wide storage unit  
* Relies on NAS (network attached storage)
* Provisioned by an administrator  
* Its life-cycle is completely separated from the pods that use it  
* A working persistent volume system is made of two parts
    1. A Persistent volume - The volume itself
    2. A Persistent volume claim (PVC)
      * Setup within a Deployment or Pod
      * Requests access to a Persistent volume


#### YAML for PV's:  
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv

spec:
  capacity: 10gi

  accessModes:
    - ReadWriteOnce # One client can read/write
    - ReadOnlyMany # Unlimited clients can read only

  # Even if the claim is deleted don't delete the volume from the cloud
  persistentVolumeReclaimPolicy: retain

  # Define your cloud service provider
  # AWS
  awsElasticBlockStore:
    volumeID: <volumeID>
    fsType: ext4

  # # Azure
  # azureFile:
  #   secretName: <secret>
  #   shareName: <shareName>
  #   readOnly: false

  # # Google Cloud Platform
  # gcePersistentDisk:
  #   pdName: <pdName>
  #   fsType: ext4

```


#### YAML for PVC's:  
1. First we create a claim:
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: pp-dd-account-hdd-5g
      annotations:
        # This may be depreciated, see this website if you have problems.
        # https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/
        volume.beta.kubernetes.io/storage-class: accounthdd

    spec:
      accessModes:
      - ReadWriteOnce # We want read/write access

      resources:
        requests:
          storage: 5Gi # We want 5gb of storage

    ```


2. Then we attach our claim to a pod or deployment:
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-with-claim
      labels:
        name: storage

    spec:
      # Define whatever you need in the container and then define a volume mount
      containers:
      - image: image
        name: whatever

        # Mount PVC using its assigned internal name
        volumeMounts:
        - name: blobdisk01
          mountPath: /mnt/blobdisk
      
      # Define our volume
      volumes:
      - name: blobdisk01 # Give it a name that can be used internally
        persistentVolumeClaim:
          claimName: pp-dd-account-hdd-5g

    ```