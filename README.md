# Kubernetes Basics

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


### Useful YAML based commands:  
* `kubectl create -f <yml_file>` - Create what is defined in the yaml file  
	* `--save-config` - Saves metadata needed to run apply later on  
	* `--dry-run=client` - Create and then instantly destroy  
	* `--validate=<bool>` - Checks yaml syntax  

* `kubectl apply -f <yml_file>` - Create what is defined in the yaml file or update the pods if they exist   

* `kubectl delete -f <yml_file>` - Delete what is defined in the yaml file  

* `kubectl edit -f <yml_file>` - Edit the yaml file  


## YAML for Pods:  

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
