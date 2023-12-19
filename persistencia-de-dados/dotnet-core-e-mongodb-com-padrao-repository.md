# .NET Core e MongoDB com Padrão Repository

## Referências
- [Microsoft. Create a web API with ASP.NET Core and MongoDB](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mongo-app?view=aspnetcore-8.0&tabs=visual-studio)
- [Microsoft. Use NoSQL databases as a persistence infrastructure](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/nosql-database-persistence-infrastructure)
- [Nic Raboy. Build Your First .NET Core Application with MongoDB Atlas](https://www.mongodb.com/developer/languages/csharp/build-first-dotnet-core-application-mongodb-atlas/#building-a-poco-class-for-the-mongodb-document-model)
- [Nic Raboy. Create a RESTful API with .NET Core and MongoDB](https://www.mongodb.com/developer/languages/csharp/create-restful-api-dotnet-core-mongodb/)
- [MongoDb Reference. Re-use](https://mongodb.github.io/mongo-csharp-driver/2.14/reference/driver/connecting/#re-use)
- [MongoDb Reference. Mapping Classes](https://mongodb.github.io/mongo-csharp-driver/2.14/reference/bson/mapping/)

## Document-Oriented Database

Quando se usa uma base de dados NoSQL na camada de infraestrutura, normalmente não se usa um ORM, como o EF Core, mas sim a API fornecida pelo motor NoSQL, como é o caso do MongoDB e outros.

Mesmo assim, ao usar uma base NoSQL como uma document-oriented, a forma de se implementar modelos no padrão Aggregate é em parte semelhante, como na identificação dos aggregate-roots, entidades-filhas e classes value-object; porém, com mais flexibilidade do que ao usar EF Core, visto não precisar de um mapeamento relacional.

Isso não quer dizer que os modelos POCO podem ser adaptados facilmente para suportar diferentes infraestruturas de persistência, por exemplo no caso de reaproveitá-los para uma base relacional, visto que cada estilo de persistência impõe diversas limitações próprias e trade-offs. O modelo deve ser implementado a partir de um entendimento de como os dados serão usados em cada base particularmente.

Na base orientada a documento, um aggregate é implementado como um único documento, de modo que os aggregates se parecem com documentos serializados.



## Modelamento
- [Microsoft. Data modeling in Azure Cosmos DB](https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/modeling-data)
- [Microsoft. Use NoSQL databases as a persistence infrastructure](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/nosql-database-persistence-infrastructure)
```csharp
namespace BookStoreApi.Models;
public class Locations
{
    [BsonId]
    [BsonRepresentation(BsonType.ObjectId)]
    public string Id { get; set; }
    public int LocationId { get; set; }
    public string Code { get; set; }
    [BsonRepresentation(BsonType.ObjectId)]
    public string Parent_Id { get; set; }
    public string Description { get; set; }
    public double Latitude { get; set; }
    public double Longitude { get; set; }
    public GeoJsonPoint<GeoJson2DGeographicCoordinates> Location { get; private set; }

    public void SetLocation(double lon, double lat) => SetPosition(lon, lat);

    private void SetPosition(double lon, double lat)
    {
        Latitude = lat;
        Longitude = lon;
        Location = new GeoJsonPoint<GeoJson2DGeographicCoordinates>(
            new GeoJson2DGeographicCoordinates(lon, lat));
    }
}
```

## Implementação
1. Objeto que guarda e disponibiliza as configurações de acesso à base.
2. Contexto para acesso à coleção mongo.
3. Abstração do repositório.
    - Opcionalmente, abstração `IRepository`.
4. Implementação do repositório.
    - Opcionalmente, implementação do repositório-base.
5. No container de DI da aplicação:
    - Instanciação do objeto de configuração com dados mapeados.
    - Registro do contexto.
    - Registro do repositório (e repositório-base).

## Configurações de acesso à base

### Mapeando configurações do `appsettings.json`
As configurações de acesso são definidas no `appsettings.json`:
```json
{
  "MongoDatabase": {
    "ConnectionString": "mongodb://localhost:27017",
    "DatabaseName": "MinhaBaseMongoDb"
  }
}
```
As configurações são mapeadas e guardadas em um objeto de opções.
```csharp
namespace Infrastructure;
public class MongoDbSettings
{
    public string ConnectionString { get; set; } = null!;
    public string DatabaseName { get; set; } = null!;
}
```
O objeto é registrado no container de injeção de dependência da aplicação.
```csharp
builder.Services.Configure<MongoDbSettings>(
  builder.Configuration.GetSection("MongoDatabase"));
```

## Unit Of Work
- [Microsoft. Tempo de vida, configuração e inicialização do DbContext](https://learn.microsoft.com/pt-br/ef/core/dbcontext-configuration/#dbcontext-in-dependency-injection-for-aspnet-core)
- [Connection Strings](https://learn.microsoft.com/en-us/ef/core/miscellaneous/connection-strings)
- Uma Unit of Work serve como contexto de acesso à base, e expõe o acesso para o repositório de sua coleção específica.
- O `MongoClient` lê a instância do servidor para rodar operações na base.
- O método `IMongoDatabase.GetCollection<TDocument>(collection)` permite acessar dados de uma coleção específica, viabilizando operações CRUD nela. Ele retorna a representação da coleção `MongoCollection`, a qual recebe as operações CRUD.
- O `TDocument` representa o tipo de objeto CLR na coleção.
```csharp
using DotNetDDD.WebApi.Models;
using Microsoft.Extensions.Options;
using MongoDB.Driver;

namespace Infrastructure.Context;

public class MongoDbContext
{
    private readonly IMongoDatabase _db = null!;

    public MongoDbContext(IOptions<MongoDbSettings> settings)
    {
        MongoClient mongoClient = new(settings.Value.ConnectionString);

        if (mongoClient != null)
            _db = mongoClient.GetDatabase(settings.Value.DatabaseName);
    }

    public IMongoCollection<T> GetCollection<T>(string collectionName)
    {
        return _db.GetCollection<T>(collectionName);
    }
}
```
- O contexto é registrado no container de DI.
> "It is recommended to store a MongoClient instance in a global place, either as a static variable or in an IoC container with a singleton lifetime" - [_Re-use_](https://mongodb.github.io/mongo-csharp-driver/2.14/reference/driver/connecting/#re-use).
```csharp
// TODO
```

### Abstrações (Domínio)
- Template de repositório relativo a objeto genérico que representa um aggregate-root.
```csharp
namespace Domain.Interfaces.Repositories.Base;
public interface IRepository<T> where T : class // Ideia: where T : IAggregateRoot
{
  Task<List<T>> GetAsync();
  Task<T?> GetAsync(string id);
  Task CreateAsync(T newEntity);
  Task UpdateAsync(string id, T updatedEntity);
  Task RemoveAsync(string id);
}
```
- Abstrações dos repositórios específicos, que herdam do `IRepository`.
```csharp
namespace Domain.Interfaces.Repositories;
public interface IBookRepository : IRepository<Book>
{
  Task<Book> GetByGenre(string genreName);
}
```


### Repositório-Base
```csharp
namespace Infrastructure.Repositories.Base;
public class BaseRepository<T> : IRepository<T> where T : class
{
  protected readonly IMongoCollection<T> _collection = null;

  public BaseRepository(IOptions<MongoDbSettings> settings)
  {
    MongoDbConext context = new(settings);
    _collection = context.GetCollection<T>(CollectionName());
  }

  protected virtual string CollectionName();

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
```csharp
builder.Services.AddSingleton<IRepository, BaseRepository>();
```

### Repositório
```csharp
namespace Infrastructure.Repositories;
public class BookRepository : BaseRepository<Book>, IBookRepository
{
  public BookRepository(MongoDbContext context) : base(context)
  {
  }

  protected override string CollectionName => "Books";
  
  public async Task<Book> GetByGenre(string genreName)
  {
    return await _collection.Find(x => x.Genre == genreName)
      .ToListAsync();
  }
}
```
```csharp
builder.Services.AddSingleton<IBookRepository, BookRepository>();
```
