# O Padrão Repository

## Referências
- [Microsoft. The Repository Pattern](https://learn.microsoft.com/en-us/previous-versions/msp-n-p/ff649690(v=pandp.10))
- [Microsoft. Design the infrastructure persistence layer](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)
- [Edward Hieatt e Rob Mee. Repository pattern](https://martinfowler.com/eaaCatalog/repository.html)


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

O padrão Repository é um padrão DDD pensado para manter responsabilidades de persistência fora do modelo de domínio. No padrão Repository, as abstrações de persistência - as interfaces - são mantidas dentro do domínio, e as implementações dessas abstrações são definidas em outro local da aplicação, na forma de adaptadores próprios para persistência. A implementação do repositório são classes que encapsulam a lógica de acesso a fontes de dados.

As vantagens que se pode obter com o repositório são:
- Permite centralizar a lógica de acesso a dados;
- Provê um ponto próprio para testes unitários;
- Provê uma arquitetura flexível para se adaptar conforme o design do resto da aplicação evolui;
- Viabiliza que a lógica de negócio seja agnóstica ao tipo de dados que constitui a fonte dos dados ou qual é a sua linguagem, bastando que a camada de regra de negócio apenas use a interface do repositório específico.

_[Sobre lidar com dois tipos de dados diferentes...]_

## Padrão Aggregate
- [Microsoft. Design the infrastructure persistence layer](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)
- [Martin Fowler. DDD Aggregate](https://martinfowler.com/bliki/DDD_Aggregate.html)

O aggregate é um padrão DDD que representa um conceito dominial, representado por um conjunto de objetos dominiais que podem ser tratados como uma única unidade, de modo que o aggregate-root é o objeto exposto à aplicação. E.g., o aggregate-root _Pedido_ agrega os value-objects _Endereço_ e _ItemDoPedido_.

O repositório deve corresponder a um aggregate-root, numa relação de um-para-um, especialmente para manter a consistência transacional, i.e., a atualização de dados.

Em suma, um repositório permite popular dados em-memória que vieram da base na forma de entidades dominiais, e assim as entidades em memória podem ser mudadas e então re-persistidas na base via transações. A partir de um comando de mudança, os dados são atualizados em memória para depois os dados serem atualizados na base via transação. Para manter a consistência transacional entre todos os objetos de um aggregate-root, um único repositório deve corresponder a um único aggregate-root.

E, também, apenas aggregates-roots devem ter repositórios.

Para assegurar essa regra no C#, convém que os repositórios implementem um tipo de repositório genérico relacionado a um único agregado, como o seguinte: `interface IRepository<T> where T : IAggregateRoot`.

## Padrão Unit of Work
- [Martin Fowler. Unit of Work](https://martinfowler.com/eaaCatalog/unitOfWork.html)

Um dos problemas com a persistência de dados é que múltiplas requisições relacionadas não precisam, cada uma, gerar uma transação própria. Se cada requisição gerar uma transação, o acesso aos dados pode ficar lento ou gerar inconsistências. É o caso, por exemplo, do registro de um usuário, em que as operações de inserção, atualização e deleção podem ser tratadas em uma transação única.

O padrão Unit Of Work serve para encapsular todas as operações que possam afetar a base de dados que se relacionam entre si e que deveriam ser consistentes. Assim, a Unit of Work entende por si o que precisa ser feito para alterar a base a partir do trabalho feito na lógica negocial.


## Padrão Data Mapper
- [Martin Fowler. Data Mapper](https://martinfowler.com/eaaCatalog/dataMapper.html)

O problema que surge ao lidar com a persistência de dados é que os objetos em-memória e a base de dados diferem nos mecanismos de estruturação de dados; cada mecanismo gera um esquema diferente que não coincidem. Esse problema exige uma lógica complexa para processar a transferência de dados dos objetos em-memória e da base de dados.

Em outras palavras, a 'tipagem' entre a lógica de negócio e a base de dados são diferentes, e as consultas apropriadas à base ou a exposição dos dados recuperados às entidades negociais precisam ser traduzidas em representações específicas.

O padrão Data Mapper representa uma camada de software em que reúne mappers que movem dados entre objetos e uma base de dados enquanto os mantém independentes entre si e de si mesmo. Isso significa que a lógica de negócio não conhece o esquema da base de dados, sua linguagem, e até mesmo se a base de dados existe; e a base de dados não conhece o esquema dos objetos em-memória. Aliás, a camada de domínio não conhece o Data Mapper -- visto lidar com mappers.


<br>

---

# .NET Core e MongoDB Driver com Padrão Repository

## Referências
- [Microsoft. Create a web API with ASP.NET Core and MongoDB](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mongo-app?view=aspnetcore-8.0&tabs=visual-studio)
- [Microsoft. Use NoSQL databases as a persistence infrastructure](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/nosql-database-persistence-infrastructure)
- [Nic Raboy. Build Your First .NET Core Application with MongoDB Atlas](https://www.mongodb.com/developer/languages/csharp/build-first-dotnet-core-application-mongodb-atlas/#building-a-poco-class-for-the-mongodb-document-model)
- [Nic Raboy. Create a RESTful API with .NET Core and MongoDB](https://www.mongodb.com/developer/languages/csharp/create-restful-api-dotnet-core-mongodb/)
- [MongoDb Reference. Re-use](https://mongodb.github.io/mongo-csharp-driver/2.14/reference/driver/connecting/#re-use)
- [MongoDb Reference. Mapping Classes](https://mongodb.github.io/mongo-csharp-driver/2.14/reference/bson/mapping/)

## Document-oriented Database

Quando se usa uma base de dados NoSQL na camada de infraestrutura, normalmente não se usa um ORM, como o EF Core, mas sim a API fornecida pelo motor NoSQL, como é o caso do MongoDB. 

Ainda assim, ao usar uma base de dados NoSQL como uma document-oriented, a forma de se desenhar os modelos conforme o padrão Aggregate é em parte semelhante, considerando a identificação dos aggregate-roots, entidades-filhas e classes value-object.

[_Continuar_]

## Código



### Objeto de configuração de acesso à base
- É necessário um objeto que mapeia e guarda as configurações de acesso à base de dados para a aplicação.
```csharp
namespace BookStoreApi.Models;
public class BookStoreDatabaseSettings
{
    public string ConnectionString { get; set; } = null!;
    public string DatabaseName { get; set; } = null!;
}
```
- Esse objeto é, então, registrado no container de injeção de dependência da aplicação.
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.Configure<BookStoreDatabaseSettings>(
  /* Apontar local onde as configurações de acesso estão definidas.
   * E.g. builder.Configuration.GetSection("BookStoreDatabase"), se em appsettings.json. */);
```

### Unit Of Work
- Uma Unit of Work serve como contexto de acesso à base, e expõe o acesso para o repositório de sua coleção específica.
- O `MongoClient` lê a instância do servidor para rodar operações na base.
- O método `IMongoDatabase.GetCollection<TDocument>(collection)` permite acessar dados de uma coleção específica, viabilizando operações CRUD nela. Ele retorna a representação da coleção `MongoCollection`, a qual recebe as operações CRUD.
- O `TDocument` representa o tipo de objeto CLR na coleção.
```csharp
namespace Infrastructure.Context;
public class MongoDbContext
{
  private readonly IMongoDatabase _db = null;

  public MongoDbContext(IOptions<MongoDbSettings> settings)
  {
    MongoClient mongoClient = new(settings.Value.ConnectionString);
    if (mongoClient != null)
      _db = mongoClient.GetDatabase(settings.Value.DatabaseName);
  }

  public IMongoCollection GetCollection<T>(string collectionName)
  {
    return _db.GetCollection<T>(collectionName);
  }
}
```
- DI
> It is recommended to store a MongoClient instance in a global place, either as a static variable or in an IoC container with a singleton lifetime - [_Re-use_](https://mongodb.github.io/mongo-csharp-driver/2.14/reference/driver/connecting/#re-use).

### Template do Repositório
- O objeto `T` representa a entidade de negócio.
```csharp
namespace Domain.Interfaces.Repositories.Base
{
  public interface IRepository<T> where T : IAggregateRoot
  {
    Task<List<T>> GetAsync();
    Task<T?> GetAsync(string id);
    Task CreateAsync(T newEntity);
    Task UpdateAsync(string id, T updatedEntity);
    Task RemoveAsync(string id);
  }
}
```
- O repositório-base implementa as operações comuns a todos repositórios específicos, para centralizar essa lógica em um único ponto e evitar repetição, conforme padrão Template.
```csharp
namespace Infrastructure.Repositories.Base;
public class BaseRepository<T> : IRepository<T> where T : IAggregateRoot
{
  protected readonly IMongoCollection<T> _collection = null;

  public BaseRepository(IOptions<MongoDbSettings> settings)
  {
    MongoDbConext context = new(settings);
    _collection = context.GetCollection<T>(
      typeof(T).Name);
  }

  protected async Task<List<T>> GetAsync() =>
    await _collection.Find(_ => true).ToListAsync();

  protected async Task<T?> GetAsync(string id) =>
    await _collection.Find(x => x.Id == id).FirstOrDefaultAsync();

  protected async Task CreateAsync(T newEntity) =>
    await _collection.InsertOneAsync(newEntity);

  protected async Task UpdateAsync(string id, T updatedEntity) =>
    await _collection.ReplaceOneAsync(x => x.Id == id, updatedEntity);

  protected async Task RemoveAsync(string id) =>
    await _collection.DeleteOneAsync(x => x.Id == id);
}
```

- DI: _TODO_
```csharp
builder.Services.AddSingleton<IBookRepository, BookRepository>();
```


### Repositório

- O repositório fica exposto para a lógica de negócio via sua interface, visto que aquela ignora a lógica de interação com a fonte dos dados.
- O repositório implementa o repositório-base, definindo a específica entidade negocial que representa.
```csharp
namespace Domain.Interfaces.Repositories;
public interface IBookRepository : IBaseRepository<Book>
{
  Task<Book> GetByGenre(string genreName);
}
```

- O contexto é instanciado no repositório via DI. Portanto, o contexto também deve estar registrado no container de DI.
```csharp
namespace Infrastructure.Repositories.Book;
public class BookRepository : BaseRepository<Book>, IBookRepository
{
  public BookRepository(MongoDbContext context) : base(context)
  {
  }
  
  public async Task<Book> GetByGenre(string genreName)
  {
    FilterDefinition<Book> filter = Builders<Book>.Filter.Eq("Genre", genreName);
    return await _collection.Find(filter).ToListAsync();
  }
}
```

- O repositório é registrado no container de DI.
```csharp
builder.Services.AddSingleton<IBookRepository, BookRepository>();
```
