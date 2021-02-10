# Azure workshop secound part
Services, Deployment, CI

## Azure Container Registry

### Create a private container registry 


```bash
az login
az group create --name workshopTwo --location westeurope
az acr create --resource-group workshopTwo --name workshopRegistry --sku Basic
az acr login --name workshopRegistry
az acr repository list --name workshopRegistry --output table

#Push images to 
docker pull mcr.microsoft.com/hello-world
docker tag mcr.microsoft.com/hello-world workshopRegistry.azurecr.io/hello-world:v1
docker push workshopRegistry.azurecr.io/hello-world:v1
     
```

## Azure App Service

## Azure Functions

## Azure Pipelines
