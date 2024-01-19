# Testes Integrados de Repositórios com .NET

## Teste de Conexão

### MongoDb

Eu idealizei um teste de conexão que verifica o retorno de um comando `ping`.

O teste ficou assim:

```csharp
[Fact]
public void VerifyMongoDbConnection_ReturnsOk()
{
    MongoDbContext context = new();

    string result = context.VerifyConnection().ToString();

    Assert.Contains("ok", result);
}
```

O teste invoca a função `VerifyConnection`, da classe `MongoDbContext`:

```csharp
public BsonDocument VerifyConnection()
{
    try
    {
        return _db.RunCommand((Command<BsonDocument>)"{ ping: 1 }"); // Retorna {{"ok":"1.0"}}.
    }
    catch (Exception)
    {
        throw;
    }
}
```

### SQL

## Repositório

### Dependency Injection

Instanciar o repositório testado diretamente vincula os testes ao código de implementação do repositório,
e assim as modificações feitas neste dificultam a manutenção do código de testes.

Por isso o repositório testado precisa ser instanciado a partir do padrão Dependency Injection.
Dessa maneira, o código de testes fica vinculado à abstração do repositório.

Veja-se um exemplo de DI do repositório. Nele é usada a biblioteca `Microsoft.Extensions.DependencyInjection`:

```csharp
using Microsoft.Extensions.DependencyInjection;

public class BooksRepositoryTests
{
    private readonly IBooksRepository _repository = null!;

    public BooksRepositoryTests()
    {
        ServiceCollection services = new();
        services.AddTransient<MongoDbContext>()
                .AddTransient<IBooksRepository, BooksRepository>();
        ServiceProvider provider = services.BuildServiceProvider();
        IBooksRepository? service = provider.GetService<IBooksRepository>();
        if (service != null)
        {
            _repository = service;
        }
    }

    // ...
}
```

### Testes

Durante o desenvolvimento dos testes de integração, talvez alguns testes utilizem a mesma base de dados ou serviço compartilhado com outros desenvolvedores ou aplicação. Isso pode interferir de maneira negativa na execução desses testes, gerando inconsistência nos dados. Para resolver a situação, podemos usar algumas abordagens, como usar uma base de homologação, bancos de dados em memória, entre outros. Dentre as opções temos a utilização de objetos mock e os stubs.