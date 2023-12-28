# O Padrão Service

## Referências
*[Em construção...]*

## O Serviço HttpClient
- [Microsoft. HttpClient Class](https://learn.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=net-7.0)
- [Microsoft. Guidelines for using HttpClient](https://learn.microsoft.com/pt-br/dotnet/fundamentals/networking/http/httpclient-guidelines)
- [Microsoft. Use IHttpClientFactory to implement resilient HTTP requests](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)
- [Microsoft. IHttpClientFactory with .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/httpclient-factory)

A documentação da Microsoft nos apresenta algumas diretrizes importantes na utilização da classe `HttpClient`. De acordo com a Microsoft, o ideal é a criação de **uma única instância de HttpClient** e que esta seja reutilizada durante a vida útil da aplicação para evitarmos problemas como o de esgotamento de soquetes. Como solução ela nos apresenta o uso da interface `IHttpClientFactory` para tornar as instâncias de HttpClient mais gerenciáveis e lidar com cenários de longa execução.

Veja-se o exemplo de implementação:
```csharp
// # Serviço MyAppAPIClientFactory.cs:
using System.Net.Http.Headers;
public class MyAppAPIClientFactory : IHttpClientFactory
{
    private readonly string uri = "http://localhost:5057";
    public HttpClient CreateClient(string name)
    {
        HttpClient _client = new();
        _client.DefaultRequestHeaders.Accept.Clear();
        _client.DefaultRequestHeaders.Accept.Add(
            new MediaTypeWithQualityHeaderValue("application/json"));
        _client.BaseAddress = new Uri(uri);
        return _client;
    }
}
// # Injeção do HttpClient instanciado na classe que precisa executar requisições:
public class HttpClientMessage
{
    private readonly HttpClient _client;
    public HttpClientMessage(HttpClient client)
    {
        _client = client;
    }
    // ...
}
```

## Testes unitários com xUnit

### Service

Um arquivo deve encapsular os testes do serviço. E.g., `MessageServiceTest.cs`.

### API desconectada

### HttpClient

A fim de manter o arquivo de testes independente da API Web, a conexão deve ser simulada no arquivo de testes.

Dado que é custoso instanciar o HttpClient, o recomendado é simular uma instância de HttpClient. Para isso, é possível usar o **Moq**.

*Exemplo: mock básico de HttpClient como dependência via It.IsAny<TValue>()*
```csharp
var userServiceMock = new Mock<PetService>(
  MockBehavior.Default,
  It.IsAny<HttpClient>()); // mock da dependência HttpClient
```


Exemplo:

```csharp
// Mock do handler da chamada.
var messageHandlerMock = new Mock<HttpMessageHandler>();
// Configurações do handler.
messageHandlerMock
  .Protected()
  .Setup<Task<HttpResponseMessage>>("SendAsync",
                                    ItExpr.IsAny<HttpRequestMessage>(),
                                    ItExpr.IsAny<CancellationToken>())
  .ReturnsAsync(new HttpResponseMessage
  {
      StatusCode = HttpStatusCode.OK,
      Content = new StringContent(@"
        [
          { ""id"": ""1"", ""name"": ""Mark"" },
          { ""id"": ""2"", ""name"": ""Luke"" }
        ]")
  });

// Mock do cliente Http, incluindo tratamento da requisição.
var httpClientMock = new Mock<HttpClient>(MockBehavior.Default, messageHandlerMock.Object);
// Configuração da chamada.
httpClientMock.Object.BaseAddress = new Uri("http://localhost:5057");

// Agora basta instanciar o serviço a ser testado.
UsersService usersService = new(httpClientMock.Object);

// Testes...
```
