# Objectives

- Explain the features and benefits Azure Container Registry offers.
- Describe how to use ACR Tasks to automate builds and deployments.
- Explain the elements in a Dockerfile.
- Build and run an image in the ACR by using Azure CLI.

## Discover the Azure Container Registry

Use the Azure Container Registry (ACR) service with your existing container development and deployment pipelines, or use Azure Container Registry Tasks to build container images in Azure

# Use cases
various deployment targets:

- Scalable orchestration systems that manage containerized applications across clusters of hosts, including Kubernetes, DC/OS, and Docker Swarm.
- Azure services that support building and running applications at scale, including Azure Kubernetes Service (AKS), App Service, Batch, Service Fabric, and others.

Configure ACR Tasks to automatically rebuild application images when their base images are updated, or automate image builds when your team commits code to a Git repository.

## Azure Container Registry service tiers
- Basic: Basic registries have the same programmatic capabilities as Standard and Premium However, the included storage and image throughput are most appropriate for lower usage scenarios.
- Standard: same capabilities as Basic, with increased included storage and image throughput. Standard registries should satisfy the needs of most production scenarios.
- Premium: In addition to higher image throughput, Premium adds features such as geo-replication for managing a single registry across multiple regions, content trust for image tag signing, private link with private endpoints to restrict access to the registry.

## Supported images and artifacts

Azure container registries can include both Windows and Linux images. Docker.
Helm Charts: (https://docs.microsoft.com/en-us/azure/container-registry/container-registry-helm-repos)
Open Container Initiative (OCI) Image Format Specification: https://github.com/opencontainers/image-spec/blob/main/spec.md

## Automated image builds
Use Azure Container Registry Tasks (ACR Tasks) (https://docs.microsoft.com/en-us/azure/container-registry/container-registry-tasks-overview) to streamline building, testing, pushing, and deploying images in Azure.

# Explore storage capabilities

(https://docs.microsoft.com/en-us/learn/modules/publish-container-image-to-azure-container-registry/3-azure-container-registry-storage)

# Build and manage containers with tasks

ACR Tasks is a suite of features within Azure Container Registry. It provides cloud-based container image building for platforms including Linux, Windows, and Azure Resource Manager, and can automate OS and framework patching for your Docker containers. ACR Tasks enables automated builds triggered by source code updates, updates to a container's base image, or timers.


## Task scenarios
Quick task - Build and push a single container image to a container registry on-demand, in Azure, without needing a local Docker Engine installation. Think docker build, docker push in the cloud.

Automatically triggered tasks - Enable one or more triggers to build an image:
  Trigger on source code update
  Trigger on base image update
  Trigger on a schedule

Multi-step task - Extend the single image build-and-push capability of ACR Tasks with multi-step, multi-container-based workflows.


## Quick task
With quick tasks, you can verify your automated build definitions and catch potential problems prior to committing your code.

Using the familiar `docker build` format, the `az acr build` command in the Azure CLI takes a context (the set of files to build), sends it to ACR Tasks and, by default, pushes the built image to its registry upon completion.

## Trigger task on source code update

Trigger a container image build or multi-step task when code is committed, or a pull request is made or updated,

configure a build task with the Azure CLI command `az acr task ` create by specifying a Git repository and optionally a branch and Dockerfile.

## Trigger on base image update
When the updated base image is pushed to your registry, or a base image is updated in a public repo such as in Docker Hub, ACR Tasks can automatically build any application images based on it.

## Schedule a task
Scheduling a task is useful for running container workloads on a defined schedule, or running maintenance operations or tests on images pushed regularly to your registry.

## Multi-step tasks

Multi-step tasks, defined in a YAML file specify individual build and push operations for container images or other artifacts. 
For example, you can create a multi-step task that automates the following:

- Build a web application image
- Run the web application container
- Build a web application test image
- Run the web application test container, which performs tests against the running application container
- If the tests pass, build a Helm chart archive package
- Perform a helm upgrade using the new Helm chart archive package

## Explore elements of a Dockerfile

The following aspects of the image are defined:

- The base or parent image we use to create the new image
- Commands to update the base OS and install additional software
- Build artifacts to include, such as a developed application
- Services to expose, such a storage and network configuration
- Command to run when the container is launched

notice the last step, step 8. The ENTRYPOINT in the file indicates which process will execute once we run a container from an imag

```
# Step 1: Specify the parent image for the new image
FROM ubuntu:18.04

# Step 2: Update OS packages and install additional software
RUN apt -y update &&  apt install -y wget nginx software-properties-common apt-transport-https \
	&& wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb \
	&& dpkg -i packages-microsoft-prod.deb \
	&& add-apt-repository universe \
	&& apt -y update \
	&& apt install -y dotnet-sdk-3.0

# Step 3: Configure Nginx environment
CMD service nginx start

# Step 4: Configure Nginx environment
COPY ./default /etc/nginx/sites-available/default

# STEP 5: Configure work directory
WORKDIR /app

# STEP 6: Copy website code to container
COPY ./website/. .

# STEP 7: Configure network requirements
EXPOSE 80:8080

# STEP 8: Define the entry point of the process that runs in the container
ENTRYPOINT ["dotnet", "website.dll"]
```

# Exercise: Build and run a container image by using Azure Container Registry Tasks
(https://docs.microsoft.com/en-us/learn/modules/publish-container-image-to-azure-container-registry/6-build-run-image-azure-container-registry)

Create a basic container registry. 
```
az acr create --resource-group az204-acr-rg \
    --name <myContainerRegistry> --sku Basic
```

create the Dockerfile.
```
echo FROM mcr.microsoft.com/hello-world > Dockerfile
```

Run the az acr build command, which builds the image and, after the image is successfully built, pushes it to your registry
```
az acr build --image sample/hello-world:v1  \
    --registry <myContainerRegistry> \
    --file Dockerfile .
```


Use the az acr repository list command to list the repositories in your registry
```
az acr repository list --name <myContainerRegistry> --output table
```

Use the az acr repository show-tags command to list the tags on the sample/hello-world repository. Replace <myContainerRegistry> with the name you used earlier.
```
az acr repository show-tags --name <myContainerRegistry> \
    --repository sample/hello-world --output table
```

## Run the image in the ACR

Run the sample/hello-world:v1 container image from your container registry by using the `az acr run` command.
```
az acr run --registry <myContainerRegistry> \
    --cmd '$Registry/sample/hello-world:v1' /dev/null
```

# Knowledge check
Which of the following Azure Container Registry support geo-replication to manage a single registry across multiple regions? Premium