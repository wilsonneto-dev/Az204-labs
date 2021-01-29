### Criar recursos

Criar o grupo:

`az group create --name az204lab2 --location eastus2`

Configurar defaults:

`az configure --defaults group=az204lab2 location=eastus2`

Criar o storage:

`az storage account create --name functionstorage --location eastus2 --resource-group az204lab2 --sku Standard_LRS`

Criar a Function App:

` az functionapp create --consumption-plan-location eastus2 --name funclogic --os-type Linux --runtime dotnet --storage-account functionstorage`

#### Criar a HTTP function

Criar a HTTP Trigger function:

`func init --worker-runtime dotnet --force`

Colocar a key do storage no local.settings.json

`az storage account keys list --account-name functionstorage`

Criar a http trigger:

`func function create --name Echo --template "HTTP Trigger"`

A function será 

```
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using System.Threading.Tasks;
using Microsoft.Azure.WebJobs.Extensions.Http;

namespace lb02functs
{
    public static class Echo
    {
        [FunctionName("Echo")]
        public static async Task<IActionResult> Run(
            [HttpTrigger("post")] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("Received a request");

            return new OkObjectResult(req.Body);
        }
    }
}
```

Rodar para testar:

`func start --build`

#### Criar function por schedule/tempo

Cria a function local:
`func function create --name "Recurring" --template "TimerTrigger"`

A function ficará:
```
using System;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace lb02functs
{
    public static class Recurring
    {
        [FunctionName("Recurring")]
        public static void Run([TimerTrigger("*/30 * * * * *")]TimerInfo myTimer, ILogger log)
        {
            log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
        }
    }
}
```

Rodar a function para testar:

`func start --build`

#### Criando a function que se integra as outros serviços

Criar um container na conta de storage:

`az storage container create --name content --public-access off --account-name functionstorage`

Subir um arquivo no container criado:

`az storage blob upload --account-name functionstorage --container-name content --file "{{PATH}}\settings.json" --name settings.json`

Ver se o arquivo subiu:

`az storage blob list --account-name functionstorage --container-name content --output table`


### Criar uma function HTTP que terá acesso a um arquivo do blob blob:

Criar a function:

`func function create --template "HTTP Trigger" --name "GetSettingsInfo"`

Reescrever a function para criada para:

```
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;

namespace lb02functs
{
  public static class GetSettingsInfo
  {
    [FunctionName("GetSettingsInfo")]
    public static IActionResult Run(
        [HttpTrigger("GET")] HttpRequest request,
        [Blob("content/local.settings.json")] string json
    ) => new OkObjectResult(json);
  }
}
```

Testar:

`func start --build`


#### Deploy

Fazer deploy na functionapp criada na azure:

`func azure functionapp publish funclogic`

