# Azure workshop secound part
Services, Deployment, CI

## Azure Container Registry

### Create a private container registry 

```bash
resource=workshop
registry=workshop$RANDOM
az login
az group create --name $resource --location westeurope
az acr create --resource-group $resource --name $registry --sku Basic
az acr login --name $registry

git clone https://github.com/tamascsaba/hello-workshop.git
az acr build -t hello-workshop -r $registry .
az acr repository list --name $registry --output table

#Alternativ push images with docker
az acr update -n $registry --admin-enabled true
docker pull mcr.microsoft.com/hello-world
docker tag mcr.microsoft.com/hello-world workshopRegistry.azurecr.io/hello-world
docker push $registry.azurecr.io/hello-world
     
```

## Azure App Service

### PHP Quickstart
```bash
# Replace the following URL with a public GitHub repo URL
gitrepo=https://github.com/Azure-Samples/php-docs-hello-world
webappname=mywebapp$RANDOM

# Create a resource group.
az group create --location westeurope --name $resource

# Create an App Service plan
az appservice plan create --name $webappname --resource-group $resource --sku FREE

# Create a web app.
az webapp create --name $webappname --resource-group $resource --plan $webappname

# Deploy sample code
az webapp deployment source config --name $webappname --resource-group $resource --repo-url $gitrepo --branch master --manual-integration

# Copy the result
echo http://$webappname.azurewebsites.net
```

### Python and ACR
```bash
workshopapp=workshopapp$RANDOM

az acr update -n $registry --admin-enabled true
az acr credential show --resource-group $resource --name $registry

docker login $registry.azurecr.io --username <registry-username>
docker build -t hello-workshop .
docker tag hello-workshop $registry.azurecr.io/hello-workshop:latest
docker push $registry.azurecr.io/hello-workshop:latest
docker run -it -p 8080:8000 $registry.azurecr.io/hello-workshop

az acr repository list -n $registry

az appservice plan create --name hello-workshop --resource-group $resource --is-linux
az webapp create --resource-group $resource --plan hello-workshop --name $workshopapp --deployment-container-image-name $registry.azurecr.io/hello-workshop:latest

az webapp config appsettings set --resource-group $resource --name $workshopapp --settings WEBSITES_PORT=8080
az webapp identity assign --resource-group $resource --name $workshopapp --query principalId --output tsv
az account show --query id --output tsv

az role assignment create --assignee <principal-id> --scope /subscriptions/<subscription-id>/resourceGroups/$resource/providers/Microsoft.ContainerRegistry/registries/$registry --role "AcrPull"

az webapp config container set --name $workshopapp --resource-group $resource --docker-custom-image-name $registry.azurecr.io/appsvc-tutorial-custom-image:latest --docker-registry-server-url https://$registry.azurecr.io

az webapp restart --name $workshopapp --resource-group $resource

echo http://$workshopapp.azurewebsites.net

```
Based on: https://docs.microsoft.com/hu-hu/azure/app-service/tutorial-custom-container?pivots=container-linux

Clean Up: az group delete --name $resource

