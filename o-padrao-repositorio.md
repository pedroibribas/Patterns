# O Padrão Repository

> Baseado em [The Repository Pattern](https://learn.microsoft.com/en-us/previous-versions/msp-n-p/ff649690(v=pandp.10)), da Microsoft.

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
> Ver [Unit of Work](https://martinfowler.com/eaaCatalog/unitOfWork.html), em MartinFowler.com.

Um dos problemas com a persistência de dados é que as transações negociais podem gerar múltiplas requisições que tornam o acesso à base muito lento, ou gerar requisições inconsistentes.

O padrão Unit Of Work serve para encapsular todas as operações que possam afetar a base de dados que se relacionam entre si e que deveriam ser consistentes. Assim, a Unit of Work entende por si o que precisa ser feito para alterar a base a partir do trabalho feito na lógica negocial.


## Padrão Data Mapper
> Ver [Data Mapper](https://martinfowler.com/eaaCatalog/dataMapper.html), em MartinFowler.com.

O problema que surge ao lidar com a persistência de dados é que os objetos em-memória e a base de dados diferem nos mecanismos de estruturação de dados; cada mecanismo gera um esquema diferente que não coincidem. Esse problema exige uma lógica complexa para processar a transferência de dados dos objetos em-memória e da base de dados.

Em outras palavras, a 'tipagem' entre a lógica de negócio e a base de dados são diferentes, e as consultas apropriadas à base ou a exposição dos dados recuperados às entidades negociais precisam ser traduzidas em representações específicas.

O padrão Data Mapper representa uma camada de software em que reúne mappers que movem dados entre objetos e uma base de dados enquanto os mantém independentes entre si e de si mesmo. Isso significa que a lógica de negócio não conhece o esquema da base de dados, sua linguagem, e até mesmo se a base de dados existe; e a base de dados não conhece o esquema dos objetos em-memória. Aliás, a camada de domínio não conhece o Data Mapper -- visto lidar com mappers.


<br>

---

# .NET Core e MongoDB Driver com Padrão Repository

## Referências
- [Create a web API with ASP.NET Core and MongoDB](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mongo-app?view=aspnetcore-8.0&tabs=visual-studio), em Microsoft.com;
- [Build Your First .NET Core Application with MongoDB Atlas; MongoDB.com](https://www.mongodb.com/developer/languages/csharp/build-first-dotnet-core-application-mongodb-atlas/#building-a-poco-class-for-the-mongodb-document-model), em MongoDb.com;
- [Create a RESTful API with .NET Core and MongoDB; MongoDB.com.](https://www.mongodb.com/developer/languages/csharp/create-restful-api-dotnet-core-mongodb/), em MongoDb.com.
- [Re-use](https://mongodb.github.io/mongo-csharp-driver/2.14/reference/driver/connecting/#re-use); em MongoDb.github.io.

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

  public IMongoCollection Collection<T>(string collectionName)
  {
    get
    {
      return _db.GetCollection<T>(collectionName);
    }
  }
}
```

### Template do Repositório
- O objeto `T` representa a entidade de negócio.
```csharp
namespace Domain.Interfaces.Repositories.Base
{
  public interface IBaseRepository<T>
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
public class BaseRepository<T> : IBaseRepository<T> where T : class
{
  protected readonly IMongoCollection<T> _collection = null;

  public BaseRepository(IOptions<MongoDbSettings> settings)
  {
    MongoDbConext context = new(settings);
    
    string collectionName = "";

    _collection = context.GetCollection<T>(collectionName);
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
