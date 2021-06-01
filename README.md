# Kubernetes Basics

### Useful CLI based commands:  
* `Set-Alias -Name kube -Value kubectl`  


* `kubectl run <pod_name> --image=<docker_img>` - Create and run a pod  


* `kubectl port-forward <pod_name> <external>:<internal>` - Forward a port temporarily 


* `kubectl delete <pod_name>` - Delete a pod  


* `kubectl get <resource>` - Get a list of existing resources
	* `all`
	* `pods`
	* `services`


### Useful YAML based commands:  
* `kubectl create -f <yml_file>` - Create what is defined in the yaml file  
	* `--save-config` - Saves metadata needed to run apply later on  
	* `--dry-run=client` - Create and then instantly destroy  
	* `--validate=<bool>` - Checks yaml syntax  


* `kubectl apply -f <yml_file>` - Create what is defined in the yaml file or update the pods if they exist   


* `kubectl delete -f <yml_file>` - Delete what is defined in the yaml file  