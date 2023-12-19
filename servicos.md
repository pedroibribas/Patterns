A documentação da Microsoft nos apresenta algumas diretrizes importantes na utilização da classe HttpClient. De acordo com a Microsoft, o ideal é a criação de uma única instância de HttpClient e que esta seja reutilizada durante a vida útil da aplicação para evitarmos problemas como o de esgotamento de soquetes. E como solução ela nos apresenta o uso da interface IHttpClientFactory para tornar as instâncias de HttpClient mais gerenciáveis e lidar com cenários de longa execução.

Para saber mais detalhes destas recomendações da Microsoft para o HttpClient, acesse o link abaixo:

https://learn.microsoft.com/pt-br/dotnet/api/system.net.http.httpclient?view=net-7.0

https://learn.microsoft.com/pt-br/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests