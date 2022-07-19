## **Visão Geral**
O plugin **`dotnet-notification-plugin`** adiciona em uma Stack a capacidade de provisionar o uso da [**Amazon Simple Notification Service (SNS)**](https://aws.amazon.com/pt/sns/), publicando mensagens em tópicos do serviço.

### **Pré-requisitos**
Para utilizar este plugin é preciso ter instalado na sua máquina os itens abaixo:  

- Uma Stack **DotNET** criada pelo [**STK CLI**](https://stackspot.com/);  
- .NET 5 ou 6 
- O template base de `dotnet-api-template` ou o `dotnet-worker-template` já aplicado. 

### **Inputs configurados automaticamente**  
O input abaixo é usado para configurar o plugin:  

| **Campo** | **Valor** | **Descrição** |
| :--- | :--- | :--- |
| Region | Padrão: "sa-east-1" | Região da AWS que será utilizada para configurar o SNS. |

> É possível sobrescrever a configuração padrão adicionando a seção **`Sns`** no seu `appsettings.json`. Confira o exemplo abaixo:  

```json
  "Sns": {
      "Region": "sa-east-1"
  }
```

- É possivel adicionar nesta seção o parâmetro **`topicArn`** para fazer a comunicação com o seu tópico. Confira o exemplo abaixo:  

```json
  "Sns": {
      "Region": "sa-east-1",
      "TopicArn": "arn:aws:sns:sa-east-1:000000000000:mytopic",
  }
```

- A configuração abaixo será feita no **`IServiceCollection`**, através do `services.AddNotificationSns()`, no `Startup` da aplicação ou `Program`. Ela terá **`IConfiguration`** e **`IWebHostEnvironment`** como parametros de entrada. 

```csharp
//using StackSpot.Notification.SNS;

services.AddNotificationSns(configuration);
```

> O plugin oferece opções para customizar o seu serviço utilizando os parâmetros e sobrecargas do método **`AddNotificationSns`** para **`ServiceLifetime`** e **`AWSOptions`**.


#### **Exemplos de aplicação do plugin**

Confira abaixo alguns exemplos de aplicação do **`dotnet-notification-plugin`**:  

> A  classe da mensagem que será enviada para o serviço de notificação deverá ser herdada da classe **`SnsMessage`**.

- **Publish**  
Publica uma mensagem no serviço de notificação. Por padrão, ele utiliza o `Arn` do tópico configurado no `appsetings.json`. Se houver sucesso, ele retornará o `MessageId`. Confira o exemplo abaixo:  

```csharp
// using StackSpot.Notification.SNS.Common;
// using StackSpot.Notification.SNS;

public class Message : SnsMessage
{
    public Message(string text)
    {
        _text = text;
    }

    private readonly string _text;

    public string text
    {
        get { return _text; }
    }
}

[ApiController]
[Route("[controller]")]
public class SampleController : ControllerBase
{
    private readonly INotification<SnsMessage> _sns;

    public SampleController(INotification<SnsMessage> sns)
    {
        _sns = sns;
    }

    [HttpPost]
    public async Task<IActionResult> Post([FromBody] string text)
    {        
        var message = new Message(text);

        var result = await _sns.Publish(test);

        return Ok(result.Content);
    }
}
```
Além disso, ao plubicar uma mensagem é possível utilizar a sobrecarga do método **`Publish`** e informar para qual `Tópico` será enviada a mensagem através do parâmetro `topicArn`.

> A interface **`INotification`** também disponibiliza os métodos `ListTopics` e `ListSubscriptionsByTopic`, que listam os **Tópicos** e **Inscritos**, respectivamente.

#### **Execução em ambiente local**    

Para fazer o desenvolvimento local do plugin, crie um contêiner com a imagem do [**LocalStack**](https://github.com/localstack/localstack). 

Para que funcione localmente, preencha a variável de ambiente **`LOCALSTACK_CUSTOM_SERVICE_URL`** com o valor da URL do serviço. O valor padrão do LocalStack é **http://localhost:4566**.

Confira abaixo um exemplo de arquivo **`docker-compose`** com a criação do contêiner: 

```
version: '2.1'

services:
  localstack:
    image: localstack/localstack
    ports:
      - "4566:4566"
    environment:
      - SERVICES=sns
      - AWS_DEFAULT_OUTPUT=json
      - DEFAULT_REGION=sa-east-1
```

Depois de criar o contêiner, crie um tópico para fazer os testes com o componente. 

É recomendado que você tenha instalado em sua estação o [**AWS CLI**](https://aws.amazon.com/pt/cli/).  

Confira abaixo um exemplo de comando para criar uma fila:

```
aws  sns create-topic --endpoint-url=http://localhost:4566 --region=sa-east-1 --name [NOME DO SEU TÓPICO]
```
