# Azure workshop secound part
Services, Deployment, CI

## Azure Container Registry

### Create a private container registry 

```bash
az login
az group create --name workshopTwo --location westeurope
az acr create --resource-group workshopTwo --name workshopRegistry --sku Basic
az acr login --name workshopRegistry

git clone https://github.com/tamascsaba/hello-workshop.git
az acr build -t hello-workshop -r workshopRegistry .
az acr repository list --name workshopRegistry --output table

#Push images with docker
az acr update -n workshopRegistry --admin-enabled true
docker pull mcr.microsoft.com/hello-world
docker tag mcr.microsoft.com/hello-world workshopRegistry.azurecr.io/hello-world:v1
docker push workshopRegistry.azurecr.io/hello-world:v1
     
```

## Azure App Service

### PHP Quickstart
```bash
# Replace the following URL with a public GitHub repo URL
gitrepo=https://github.com/Azure-Samples/php-docs-hello-world
webappname=mywebapp$RANDOM

# Create a resource group.
az group create --location westeurope --name myResourceGroup

# Create an App Service plan
az appservice plan create --name $webappname --resource-group myResourceGroup --sku FREE

# Create a web app.
az webapp create --name $webappname --resource-group myResourceGroup --plan $webappname

# Deploy sample code
az webapp deployment source config --name $webappname --resource-group myResourceGroup --repo-url $gitrepo --branch master --manual-integration

# Copy the result
echo http://$webappname.azurewebsites.net
```

## Python and ACR
```bash
git clone https://github.com/Azure-Samples/docker-django-webapp-linux.git
cd docker-django-webapp-linux

docker build --tag appsvc-tutorial-custom-image .
docker run -it -p 8000:8000 appsvc-tutorial-custom-image
az group create --name AppSvc-DockerTutorial-rg --location westeurope
az acr create --name <registry-name> --resource-group AppSvc-DockerTutorial-rg --sku Basic --admin-enabled true
az acr credential show --resource-group AppSvc-DockerTutorial-rg --name <registry-name>

docker login <registry-name>.azurecr.io --username <registry-username>
docker tag appsvc-tutorial-custom-image <registry-name>.azurecr.io/appsvc-tutorial-custom-image:latest
docker push <registry-name>.azurecr.io/appsvc-tutorial-custom-image:latest

az acr repository list -n <registry-name>

az appservice plan create --name AppSvc-DockerTutorial-plan --resource-group AppSvc-DockerTutorial-rg --is-linux
az webapp create --resource-group AppSvc-DockerTutorial-rg --plan AppSvc-DockerTutorial-plan --name <app-name> --deployment-container-image-name <registry-name>.azurecr.io/appsvc-tutorial-custom-image:latest

az webapp config appsettings set --resource-group AppSvc-DockerTutorial-rg --name <app-name> --settings WEBSITES_PORT=8000
az webapp identity assign --resource-group AppSvc-DockerTutorial-rg --name <app-name> --query principalId --output tsv
az account show --query id --output tsv

az role assignment create --assignee <principal-id> --scope /subscriptions/<subscription-id>/resourceGroups/AppSvc-DockerTutorial-rg/providers/Microsoft.ContainerRegistry/registries/<registry-name> --role "AcrPull"

az webapp config container set --name <app-name> --resource-group AppSvc-DockerTutorial-rg --docker-custom-image-name <registry-name>.azurecr.io/appsvc-tutorial-custom-image:latest --docker-registry-server-url https://<registry-name>.azurecr.io

az webapp restart --name <app_name> --resource-group AppSvc-DockerTutorial-rg

```
Based on: https://docs.microsoft.com/hu-hu/azure/app-service/tutorial-custom-container?pivots=container-linux

