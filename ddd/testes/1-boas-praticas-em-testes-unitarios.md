# Testes Unitários com .NET


## Referências
- [Microsoft. Unit testing best practices with .NET Core and .NET Standard](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)
- [Steve Smith. Unit test controller logic in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/testing?view=aspnetcore-8.0)


## Testes Limpos: Princípios F.I.R.S.T.

O acrônimo F.I.R.S.T. representa os princípios definidos por Robert C. Martin, o Uncle Bob, no seu livro “Código Limpo, Habilidades práticas do Agile Software” (2009), a fim de orientar a criação de testes limpos.


### F para *Fast* - Rápidos

Os testes devem ser rápidos tanto na escrita quanto em sua execução.

O ideal é que consigamos executá-los frequentemente durante o desenvolvimento da aplicação. Isso possibilita um feedback rápido sobre a qualidade do código e ajuda a mitigar problemas de forma precoce.

> Testes que infringem o princípio "I" tendem a prejudicar o princípio "F", porque a configuração das dependências do teste atrapalham a rapidez de sua execução. E.g., rodar uma API Web para testar serviços.


### I para *Isolated/Independent* - Isolados/Independes

O ideal é que os testes sejam independentes uns dos outros e que não dependam de uma ordem de execução. Isso garante que cada teste valide uma única unidade de código, isolando-a de outras dependências.

A cadamada de testes, também, deve funcionar independentemente das outras camadas, como por ex. da API Web ou da base-de-dados. 

Para manter a indepedência do testes, é comum usar técnicas como a criação de dados ou classes fake, que replicam o código testado; o mock, que são simulacros do código testado; e o stub, que é o simulacro do comportamento testado.

> A Microsoft disponibiliza uma biblioteca para projetos de teste chamada **Moq**, para se criar simulacros das dependências no lugar de sua instanciação.

> Para garantir o isolamento de testes de repositórios, pode-se criar uma base de homologação ou usar dados in-memory. Para bases SQL, o EF oferece um recurso para simular uma base de dados.

Os testes são unitários quando apenas o código testado é efetivamente rodado, de modo que suas dependências não são instanciadas no código de teste. Ou seja, as dependências do código de unidade não fazem parte do teste, e isso garante sua indepedência.

> Testes que infringem o princípio "I" tendem a infringir os princípios "F" e "R", atrapalhando a testagem.

Quando o código das dependências também é rodado para verificar seu funcionamento, o teste não é mais unitário, e sim **integrado**. E.g., quando um teste é feito enquanto uma API está on-line, ou que realiza transações na base-de-dados.


### R para *Repeatable* - Repetíveis

A possibilidade de conseguir repetir os testes em qualquer ambiente ou máquina, sem depender de fatores externos ou estado compartilhado. Isso evita inconsistências nos resultados dos testes.


### S para *Self-Validating* - Autovalidável

Os testes devem ser autovalidáveis, ou seja, devem ser capazes de indicar automaticamente se passaram ou falharam. Isso permite que os desenvolvedores identifiquem rapidamente problemas no código.

Isso significa que os testes devem ser capazes de verificar o resultado de uma ação com o mais detalhes possível.

> A verificação de resultados pode ser feito pelo **padrão Result**.

> Para gerar resultados validáveis no .NET, é recomendado usar os tipos da biblioteca **FluentResults** como retorno na implementação dos métodos testados.


### T para *Timely* - A Tempo

Os testes devem ser escritos em tempo hábil.

Idealmente, os testes devem ser escritos antes da implementação do código real. Isso reforça a aplicação do Test-Driven Design.

> No **Test-Drive Design (TDD)**, o teste orienta a codificação, de forma que a implementação do código precisa passar no teste. Isso promove a qualidade do próprio código real, como na identificação de dependências, de exceções, de condições, etc.

Testes oportunos permitem que os desenvolvedores validem seu código à medida que o desenvolvem, facilitando a detecção precoce de erros.


## Padrões de Nomenclatura

### Nomenclatura das Classes

- Teste de Unidade: 
  - `[FileName]Test` (e.g. *UserServiceTest.cs*).

- Teste de Integração: 
  - `[FileName]IntegrationTest` (e.g. *UserServiceIntegrationTest.cs*).

### Nomenclatura dos Testes

O nome dos testes devem ser bem significativos sobre do que tratam.

Exemplos:

- `Add_EmptyString_ThrowsArgumentException`*[Operação_Cenário_Expectativa]*

- `WhenEmptyStringShouldThrowArgumentException`

- `QuandoStringVaziaDeveLancarArgumentException`

- `TestaAtualizacaoInformacaoDeterminadoBanco`


## Padrão Arrange-Act-Assert - AAA

O padrão Arrange-Act-Assert é uma forma de se organizar o código de um teste. O código segue o seguinte formato:
- Código para o arrange, que é o código de preparação, que antecede à ação e fornece aquilo que será utilizado no código de ação;
- Código para o act, que é efetivamente a invocação do código a ser testado;
- Código para o assert, que é o código para as verificações que incidem sobre o resultado da ação, e que representa as expectativas do desenvolvedor.

## Técnica do projeto _Automapper_

O cenário é uma classe que recebe os múltiplos casos de teste.

## Padrão Test-Data Builder

O problema que surge em testes é o código volumoso para o seu arranjo, que o torna demasiadamente complexo.

A criação dos elementos necessários ao teste podem, então, estar em uma classe própria Builder, que será invocado na classe Test.

Os arquivos Builder:
- São nomeados com o sufixo `Builder`.
- Podem estar em uma pasta própria no projeto de testes, com o nome *Builder*.

## Padrão Result

Os testes precisam ser autovalidáveis, conforme o princípio 'S' dos testes limpos.

O padrão Result tem como finalidade a validação do resultado da operação testada.

Isso significa que um resultado é guardado a partir do método testado para verificar sua validade.

## Padrão DTO

O uso de DTOs facilita a replicação de dados dos modelos.

## Definir saídas no console de testes

O `ITestOutputHelper` é um tipo do .NET com o qual é possível personalizar o conteúdo do **Outcomes** do **Test Explorer**.

Seu uso é via DI.

Veja-se o exemplo:

```csharp
public class MyTests()
{
    public ITestOutputHelper _testOutput { get; set; }
    public MyTests(ITestOutputHelper testOutput)
    {
        _testOutput = testOutput;
        _testOutput.WriteLine("Constructor succesfully executed.");
    }
}
```