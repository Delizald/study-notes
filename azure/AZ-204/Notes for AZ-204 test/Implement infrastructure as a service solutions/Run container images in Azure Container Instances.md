# Learning objectives
- Describe the benefits of Azure Container Instances and how resources are grouped.
- Deploy a container instance in Azure by using the Azure CLI.
- Start and stop containers using policies.
- Set environment variables in your container instances.
- Mount file shares in your container instances.


# Explore Azure Container Instances
Here are some of the benefits:
- Fast startup: ACI can start containers in Azure in seconds, without the need to provision and manage VMs
- Container access: ACI enables exposing your container groups directly to the internet with an IP address and a fully qualified domain name (FQDN)
- Hypervisor-level security: Isolate your application as completely as it would be in a VM
- Customer data: The ACI service stores the minimum customer data required to ensure your container groups are running as expected
- Custom sizes: ACI provides optimum utilization by allowing exact specifications of CPU cores and memory
- Persistent storage: Mount Azure Files shares directly to a container to retrieve and persist state
- Linux and Windows: Schedule both Windows and Linux containers using the same API.

For scenarios where you need full container orchestration, including service discovery across multiple containers, automatic scaling, and coordinated application upgrades, we recommend Azure Kubernetes Service (AKS) (https://docs.microsoft.com/en-us/azure/aks/).

## Container groups

container group is a collection of containers that get scheduled on the same host machine. The containers in a container group share a lifecycle, resources, local network, and storage volumes. It's similar in concept to a pod in Kubernetes.

**Multi-container groups currently support only Linux containers. For Windows containers, Azure Container Instances only supports deployment of a single instance.**

## Deployment
use a Resource Manager template or a YAML file.
Resource Manager template is recommended when you need to deploy additional Azure service resources (for example, an Azure Files share) when you deploy the container instances.

Due to the YAML format's more concise nature, a YAML file is recommended when your deployment includes only container instances.

## Resource allocation
Azure Container Instances allocates resources such as CPUs, memory, and optionally GPUs (preview) to a container group by adding the resource requests of the instances in the group

## Networking

To enable external clients to reach a container within the group, you must expose the port on the IP address and from the container. Because containers within the group share a port namespace, port mapping isn't supported. 

## Storage

Supported volumes include:

- Azure file share
- Secret
- Empty directory
- Cloned git repo

## Common scenarios
Example usage could include:

- A container serving a web application and a container pulling the latest content from source control.
- An application container and a logging container. The logging container collects the logs and metrics output by the main application and writes them to long-term storage.
- An application container and a monitoring container. The monitoring container periodically makes a request to the application to ensure that it's running and responding correctly, and raises an alert if it's not.
- A front-end container and a back-end container. The front end might serve a web application, with the back end running a service to retrieve data.

# Exercise: Deploy a container instance by using the Azure CLI
(https://docs.microsoft.com/en-us/learn/modules/create-run-container-images-azure-container-instances/3-run-azure-container-instances-cloud-shell)

Create a DNS name to expose your container to the Interne
```
DNS_NAME_LABEL=aci-example-$RANDOM
```
Run the following az container create command to start a container instance.
```
az container create --resource-group az204-aci-rg 
    --name mycontainer 
    --image mcr.microsoft.com/azuredocs/aci-helloworld 
    --ports 80 
    --dns-name-label $DNS_NAME_LABEL --location <myLocation>
```
When the az container create command completes, run az container show to check its status.
```
az container show --resource-group az204-aci-rg 
    --name mycontainer 
    --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" 
    --out table
```

# Run containerized tasks with restart policies
With a configurable restart policy, you can specify that your containers are stopped when their processes have completed. Because container instances are billed by the second, you're charged only for the compute resources used while the container executing your task is running.

## Container restart policy
you can specify one of three restart policy settings:
- Always:  always restarted. This is the default setting applied when no restart policy is specified at container creation.
- Never: never restarted. The containers run at most once.
- OnFailure: are restarted only when the process executed in the container fails (when it terminates with a nonzero exit code). The containers are run at least once.

## Specify a restart policy
Specify the --restart-policy parameter when you call az container create.

```
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mycontainerimage \
    --restart-policy OnFailure
```

# Set environment variables in container instances
This is similar to the `--en`v command-line argument to `docker run`.

In the example below two variables are passed to the container when it is created. The example below is assuming you are running the CLI in a Bash shell or Cloud Shell, if you use the Windows Command Prompt, specify the variables with double-quotes, such as --environment-variables "NumWords"="5" "MinLength"="8".

```
az container create 
    --resource-group myResourceGroup 
    --name mycontainer2 
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest 
    --restart-policy OnFailure 
    --environment-variables 'NumWords'='5' 'MinLength'='8's
```

## Secure values
Environment variables with secure values aren't visible in your container's properties. Their values can be accessed only from within the container. For example, container properties viewed in the Azure portal or Azure CLI display only a secure variable's name, not its value.

Set a secure environment variable by specifying the secureValue property instead of the regular value for the variable's type. The two variables defined in the following YAML demonstrate the two variable types.

```
apiVersion: 2018-10-01
location: eastus
name: securetest
properties:
  containers:
  - name: mycontainer
    properties:
      environmentVariables:
        - name: 'NOTSECRET'
          value: 'my-exposed-value'
        - name: 'SECRET'
          secureValue: 'my-secret-value'
      image: nginx
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  osType: Linux
  restartPolicy: Always
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

You would run the following command to deploy the container group with YAML:

```
az container create --resource-group myResourceGroup 
    --file secure-env.yaml
```

# Mount an Azure file share in Azure Container Instances
By default, Azure Container Instances are stateless. If the container crashes or stops, all of its state is lost. 

Azure Files offers fully managed file shares in the cloud that are accessible via the industry standard Server Message Block (SMB) protocol. 

## Limitations

- You can only mount Azure Files shares to Linux containers.
- Azure file share volume mount requires the Linux container run as root.
- Azure File share volume mounts are limited to CIFS support.

## Deploy container and mount volume

The `--dns-name-label` value must be unique within the Azure region where you create the container instance.

```
az container create \
    --resource-group $ACI_PERS_RESOURCE_GROUP \
    --name hellofiles \
    --image mcr.microsoft.com/azuredocs/aci-hellofiles \
    --dns-name-label aci-demo \
    --ports 80 \
    --azure-file-volume-account-name $ACI_PERS_STORAGE_ACCOUNT_NAME \
    --azure-file-volume-account-key $STORAGE_KEY \
    --azure-file-volume-share-name $ACI_PERS_SHARE_NAME \
    --azure-file-volume-mount-path /aci/logs/
```

## Deploy container and mount volume - YAML
```
apiVersion: '2019-12-01'
location: eastus
name: file-share-demo
properties:
  containers:
  - name: hellofiles
    properties:
      environmentVariables: []
      image: mcr.microsoft.com/azuredocs/aci-hellofiles
      ports:
      - port: 80
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      - mountPath: /aci/logs/
        name: filesharevolume
  osType: Linux
  restartPolicy: Always
  ipAddress:
    type: Public
    ports:
      - port: 80
    dnsNameLabel: aci-demo
  volumes:
  - name: filesharevolume
    azureFile:
      sharename: acishare
      storageAccountName: <Storage account name>
      storageAccountKey: <Storage account key>
tags: {}
type: Microsoft.ContainerInstance/containerGroups
```

## Mount multiple volumes

To use a template or YAML file, provide the share details and define the volumes by populating the volumes array in the properties section of the template.

```
"volumes": [{
  "name": "myvolume1",
  "azureFile": {
    "shareName": "share1",
    "storageAccountName": "myStorageAccount",
    "storageAccountKey": "<storage-account-key>"
  }
},
{
  "name": "myvolume2",
  "azureFile": {
    "shareName": "share2",
    "storageAccountName": "myStorageAccount",
    "storageAccountKey": "<storage-account-key>"
  }
}]
```

Next, for each container in the container group in which you'd like to mount the volumes, populate the volumeMounts array in the properties section of the container definiti

```
"volumeMounts": [{
  "name": "myvolume1",
  "mountPath": "/mnt/share1/"
},
{
  "name": "myvolume2",
  "mountPath": "/mnt/share2/"
}]
```

# Knowledge check

Which of the methods below is recommended when deploying a multi-container group that includes only containers?  YAML file