## Lab: Building a web application on Azure platform as a service offerings

Logar no Azure CLI:

`az login`

---

### Exercise 1: Build a back-end API by using Azure Storage and the Web Apps feature of Azure App Service

#### Storage Account:

Criar um grupo: 

`az group create --name ManagedPlatform --location eastus2`

Criar o storage account:

`az storage account create --name imgstor --location eastus2 --resource-group ManagedPlatform --sku Standard_LRS`

Pegar chaves de acesso:

`az storage account keys list --resource-group ManagedPlatform --account-name imgstor`

A connection string será a key listada preenchida abaixo:

`DefaultEndpointsProtocol=https;AccountName=imgstor;AccountKey={{KEY}};EndpointSuffix=core.windows.net`

Criar o container "images"

`az storage container create --name images --public-access blob --resource-group ManagedPlatform --account-name imgstor`

Subir o arquivo "grilledcheese.jpg"

`az storage blob upload --name grilledcheese.jpg --container images --file {{PATH}}\grilledcheese.jpg --account-name imgstor --account-key {{STORAGE KEY}}`

Pegar o endereço do storage:

`az storage account show --name imgstor --resource-group ManagedPlatform --query primaryEndpoints.blob`

Para ver a imagem basta pegar o endereço retornado e montar:

{{storage blob public address}}/images/grilledcheese.jpg

---

#### Criar o Web App da API:

Selecionar grupo e região padrão:

`az configure --defaults group=ManagedPlatform location=eastus2`

Criar o service plan:

`az appservice plan create --name ManagedPlan --sku S1`

Ver os runtimes disponíveis:

`az webapp list-runtimes`

Criar o webapp:

`az webapp create --name imgapi --plan ManagedPlan --% --runtime "DOTNETCORE|3.1"`

* Na linha acima o '--%' é apenas para o power shell não usar o "|" (pipe) como separador de instrução

---

#### Configurar o Web App:

Verificar os settings que ele já tem:

`az webapp config appsettings list --name imgapi`

* fazer o passo de pegar a connection string descrito na parte da criação e configuração do storage

Criar o app setting da connection string com o storage:

`az webapp config appsettings set --name imgapi --settings "StorageConnectionString=DefaultEndpointsProtocol=https;AccountName=imgstor;AccountKey={{key}};EndpointSuffix=core.windows.net"`

Verificar se esta configurado ok o app settings da connection com o storage:

`az webapp config appsettings list --name imgapi`

Para ver o endereço do webapp, use:

`az webapp show --name imgapi --query "hostNames"`

---

#### Deploy do webapp:

Verificar webapps que estão no grupo:

`az webapp list --resource-group ManagedPlatform --output table`

Buscar pelo webapp:

`az webapp list --resource-group ManagedPlatform --query "[?starts_with(name, 'imgapi')]"`

Pegar o nome do webapp:

`az webapp list --resource-group ManagedPlatform --query "[?starts_with(name, 'imgapi')].{Name:name}" --output tsv`

Deploy por zip:

`az webapp deployment source config-zip --resource-group ManagedPlatform --src .\api.zip --name imgapi`

--- 

### Exercise 2: Build a front-end web application by using Azure Web Apps

#### Criar o webapp de front

Criando o webapp:

`az webapp create --name imgweb --plan ManagedPlan --% --runtime "DOTNETCORE|3.1"`

Configurar o app setting:

`az webapp config appsettings set --name imgweb --settings "ApiUrl=https://imgapi.azurewebsites.net/"`

Verificar se o appsetting está ok:

`az webapp config appsettings list --name imgweb`

Fazer deploy do webapp de front:

`az webapp deployment source config-zip --name imgweb --src .\web.zip`

Pegar a url do site:

`az webapp show --name imgweb --query "hostNames"`

---

### Apagar recursos

Para apagar os recursos criados:

`az group delete --name ManagedPlatform --no-wait --yes`

