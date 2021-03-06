---
title: Azure Container Instances tutorial - Prepare Azure Container Registry | Microsoft Docs
description: Azure Container Instances tutorial - Prepare Azure Container Registry
services: container-instances
documentationcenter: ''
author: neilpeterson
manager: timlt
editor: ''
tags: acs, azure-container-service
keywords: Docker, Containers, Micro-services, Kubernetes, DC/OS, Azure

ms.assetid: 
ms.service: container-instances
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/19/2017
ms.author: seanmck
---

# Deploy and use Azure Container Registry

This is part two of a three-part tutorial. In the [previous step](./container-instances-tutorial-prepare-app.md), a container image was created for a simple web application written in [Node.js](http://nodejs.org). In this tutorial, this image is pushed to an Azure Container Registry. If you have not created the container image, return to [Tutorial 1 – Create container image](./container-instances-tutorial-prepare-app.md). 

The Azure Container Registry is an Azure-based, private registry, for Docker container images. This tutorial walks through deploying an Azure Container Registry instance, and pushing a container image to it. Steps completed include:

> [!div class="checklist"]
> * Deploying an Azure Container Registry instance
> * Tagging container image for Azure Container Registry
> * Uploading image to Azure Container Registry

In subsequent tutorials, you deploy the container from your private registry to Azure Container Instances.

## Before you begin

This tutorial requires that you are running the Azure CLI version 2.0.4 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI 2.0]( /cli/azure/install-azure-cli).

## Deploy Azure Container Registry

When deploying an Azure Container Registry, you first need a resource group. An Azure resource group is a logical collection into which Azure resources are deployed and managed.

Create a resource group with the [az group create](/cli/azure/group#create) command. In this example, a resource group named *myResourceGroup* is created in the *eastus* region.

```azurecli
az group create --name myResourceGroup --location eastus
```

Create an Azure Container registry with the [az acr create](/cli/azure/acr#create) command. The name of a Container Registry **must be unique**. In the following example, we use the name *mycontainerregistry082*.

```azurecli
az acr create --resource-group myResourceGroup --name mycontainerregistry082 --sku Basic --admin-enabled true
```

Throughout the rest of this tutorial, we use `<acrname>` as a placeholder for the container registry name that you chose.

## Get Azure Container Registry information

Once the container registry is created, you can query its login server and password. The following code returns these values. Note each value for login server and password, as they are referenced throughout this tutorial.

Container registry login server (update with your registry name):

```azurecli
az acr show --name <acrName> --query loginServer
```

Throughout the rest of this tutorial, we use `<acrLoginServer>` as a placeholder for the container registry login server value.

Container registry password:

```azurecli
az acr credential show --name <acrName> --query "passwords[0].value"
```

Throughout the rest of this tutorial, we use `<acrPassword>` as a placeholder for the container registry password value.

## Login to the container registry

You must login to your container registry instance before pushing images to it. Use the [docker login](https://docs.docker.com/engine/reference/commandline/login/) command to complete the operation. When running docker login, you need to provide the registry login server name and credentials.

```bash
docker login --username=<acrName> --password=<acrPassword> <acrLoginServer>
```

The command returns a 'Login Succeeded’ message once completed.

## Tag container image

To deploy a container image from a private registry, the image needs to be tagged with the `loginServer` name of the registry.

To see a list of current images, use the `docker images` command.

```bash
docker images
```

Output:

```bash
REPOSITORY                   TAG                 IMAGE ID            CREATED              SIZE
aci-tutorial-app             latest              5c745774dfa9        39 seconds ago       68.1 MB
```

Tag the *aci-tutorial-app* image with the loginServer of the container registry. Also, add `:v1` to the end of the image name. This tag indicates the image version number.

```bash
docker tag aci-tutorial-app <acrLoginServer>/aci-tutorial-app:v1
```

Once tagged, run `docker images` to verify the operation.

```bash
docker images
```

Output:

```bash
REPOSITORY                                                TAG                 IMAGE ID            CREATED             SIZE
aci-tutorial-app                                          latest              5c745774dfa9        39 seconds ago      68.1 MB
mycontainerregistry082.azurecr.io/aci-tutorial-app        v1                  a9dace4e1a17        7 minutes ago       68.1 MB
```

## Push image to Azure Container Registry

Push the *aci-tutorial-app* image to the registry.

Using the following example, replace the container registry loginServer name with the loginServer from your environment.

```bash
docker push <acrLoginServer>/aci-tutorial-app:v1
```

## List images in Azure Container Registry

To return a list of images that have been pushed to your Azure Container registry, user the [az acr repository list](/cli/azure/acr/repository#list) command. Update the command with the container registry name.

```azurecli
az acr repository list --name <acrName> --username <acrName> --password <acrPassword> --output table
```

Output:

```azurecli
Result
----------------
aci-tutorial-app
```

And then to see the tags for a specific image, use the [az acr repository show-tags](/cli/azure/acr/repository#show-tags) command.

```azurecli
az acr repository show-tags --name <acrName> --username <acrName> --password <acrPassword> --repository aci-tutorial-app --output table
```

Output:

```azurecli
Result
--------
v1
```

## Next steps

In this tutorial, an Azure Container Registry was prepared for use with Azure Container Instances, and the container image was pushed. The following steps were completed:

> [!div class="checklist"]
> * Deploying an Azure Container Registry instance
> * Tagging container image for Azure Container Registry
> * Uploading image to Azure Container Registry

Advance to the next tutorial to learn about deploying the container to Azure using Azure Container Instances.

> [!div class="nextstepaction"]
> [Deploy containers to Azure Container Instances](./container-instances-tutorial-deploy-app.md)
