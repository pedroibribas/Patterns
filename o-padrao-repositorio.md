# O Padrão Repository

> Baseado em [The Repository Pattern](https://learn.microsoft.com/en-us/previous-versions/msp-n-p/ff649690(v=pandp.10)).

## Introdução

**Problemas.** É comum que a lógica de negócio - a business logic - acesse dados de um armazenamento de dados, como bases-de-dados; e acessar dados diretamente da lógica de negócio pode causar problemas como:
- Código duplicado, o que pode prejudicar a manutenção do projeto;
- Aumento da probabilidade de erros de programação;
- 'Tipagem' fraca dos dados do negócio;
- Dificuldade ao centralizar políticas relacionadas aos dados, como caching;
- Complicações para testar com facilidade a lógica de negócio isolada de dependências externas.

**Objetivos.** O padrão Repositório pode ser usado quando se quer alcançar objetivos como:
- Aprimorar a manutenabilidade e legibilidade do código ao separar a lógica de negócio da lógica de dados ou do serviço de acesso;
- Acessar a fonte de dados a partir de vários pontos do projeto, e usar regras e lógicas de acesso de tratamento centralizado e consistentes;
- Fazer uso de entidades de negócio que sejam fortemente 'tipadas' para poder identificar problemas em tempo de compilação em vez de execução;
- Implementar e centralizar uma estratégia de chaching para a fonte de dados;
- Associar um comportamento a certo dado. For example, you want to calculate fields or enforce complex relationships or business rules between the data elements within an entity;
- Maximizar a quantidade de código que possa alcançada por testes automatizados e viabilizar o teste unitário ao isolando a camada de dados;
- Aplicar um modelo de domínio para simplificar lógicas de negócio complexas.

## Conceito
O repositório é a camada que assume a lógica de interação com a fonte de dados da aplicação, isto é, a lógica que mapeia a entidade negocial para dados da fonte e vice-versa, a que recupera dados da fonte e a que persiste mudanças da entidade na fonte de dados.

As vantagens que se pode obter com o repositório são:
- Permite centralizar a lógica de acesso a dados;
- Provê um ponto próprio para testes unitários;
- Provê uma arquitetura flexível para se adaptar conforme o design do resto da aplicação evolui;
- Viabiliza que a lógica de negócio seja agnóstica ao tipo de dados que constitui a fonte dos dados ou qual é a sua linguagem, bastando que a camada de regra de negócio apenas use a interface do repositório específico.

**Dinâmica.** _Em construção_.

## Padrão Unit of Work
Em situações complexas, a lógica negocial do cliente pode usar do padrão **Unit Of Work**, pelo qual é possível encapsular várias operações relacionadas que deveriam ser consistentes entre si ou que têm dependências relacionadas, e que submete itens ao repositório para que este proceda com seus comandos sobre a fonte de dados.

## Padrão Data Mapper

> Referências:
> - [Data Mapper](https://martinfowler.com/eaaCatalog/dataMapper.html), em MartinFowler.com.

O problema que surge ao lidar com a persistência de dados é que os objetos em-memória e a base de dados diferem nos mecanismos de estruturação de dados; cada mecanismo gera um esquema diferente que não combinam, cujo tráfego de dados pode causar prejuízos. Esse problema exige uma lógica complexa para processar a transferência de dados dos objetos em-memória e da base de dados.

O padrão Data Mapper representa uma camada de software em que reúne mappers que movem dados entre objetos e uma base de dados enquanto os mantém independentes entre si e de si mesmo. Isso significa que a lógica de negócio não precisa conhecer 

O repositório atua como um mediador entre diferentes domínios, como é o caso da fonte de dados e a lógica de negócio que age sobre a entidade. Contudo, a 'tipagem' entre esses domínios são diferentes, e as consultas apropriadas à fonte ou a exposição dos dados recuperados às entidades negociais precisam ser tratadas, isto é, traduzidas entre representações específicas, o que pode ser realizado pelo padrão de Data Mapper.

<br>

---

# .NET Core e MongoDB Driver (Padrão Repository)

## Referências
- [Create a web API with ASP.NET Core and MongoDB; Microsoft](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mongo-app?view=aspnetcore-8.0&tabs=visual-studio)
- [Build Your First .NET Core Application with MongoDB Atlas; MongoDB.com](https://www.mongodb.com/developer/languages/csharp/build-first-dotnet-core-application-mongodb-atlas/#building-a-poco-class-for-the-mongodb-document-model)
- [Create a RESTful API with .NET Core and MongoDB; MongoDB.com.](https://www.mongodb.com/developer/languages/csharp/create-restful-api-dotnet-core-mongodb/)

```csharp
// API

var builder = WebApplication.CreateBuilder(args);
builder.Services.Configure<MongoDbSettings>(
  builder.Configuration.GetSection("MongoDatabase"));

// Interfaces

namespace Domain.Interfaces.Repositories.Base
{
  public interface IBaseRepository<T>
  {
  }
}

namespace Domain.Interfaces.Repositories
{
  public interface IPersonRepository : IBaseRepository<Person>
  {
    Task<Domain.Entities.Person> GetByCpf(string cpf);
  }
}

// Settings

namespace Infrastructure.Configuration
{
  public class MongoDbSettings
  {
    public string? ConnectionString { get; set; }
    public string? Database { get; set; }
    public string? CollectionName { get; set; }
  }
}

// Unit of Work para acesso à data source

namespace Infrastructure.Context
{
  public class PersonContext
  {
    private readonly IMongoDatabase _database = null;

    public PersonContext(IOptions<MongoDbSettings> settings)
    {
      var client = new MongoClient(settings.Value.ConnectionString);
      if (client != null)
        _database = client.GetDatabase(settings.Value.Database);
    }

    public IMongoCollection<Person> People(string name)
    {
      get { return _database.GetCollection<Person>("Person"); }
    }
  }
}

// Repositórios

namespace Infrastructure.Repositories.Base
{
  public class BaseRepository<T> : IBaseRepository<T> where T : class
  {
    private readonly MongoDbContext _context = null;

    protected readonly FilterDefinition<T> FilterBuilder;
    protected readonly UpdateOptions UpdateOptions;

    public BaseRepository()
    {
      _context = new MongoDbContext(MongoDbSettings);

      FilterBuilder = Builders<T>.Filter;
      UpdateOptions = new UpdateOptions { isUpsert = true; };
    }
  }
}

namespace Infrastructure.Repositories.Person
{
  public class PersonRepository : BaseRepository<Domain.Entities.Person>, IPersonRepository
  {
    public PersonRepository(IMongoDbContext context) : base(context) { }
    public Task<Domain.Entities.Person> GetByCpf(string cpf)
    {
      FilterDefinition<Domain.Entities.Person> filter = FilterBuilder.Eq("Cpf", cpf);
      return _db.Find(filter).FirstOrDefault();
    }
  }
}

```
