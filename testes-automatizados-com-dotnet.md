# Testes automatizados com .NET

## Referências
- [Microsoft. Unit testing best practices with .NET Core and .NET Standard](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices)

## Introdução
_[Em construção...]_

## Padrões

- [Microsoft. Unit testing C# in .NET Core using dotnet test and xUnit](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test)

### Camada independente
A cadamada de testes deve funcionar independentemente das outras camadas, como por ex. da API Web.

### Nomenclaturas
- `QuandoStringVaziaDeveLancarArgumentException`
- `WhenEmptyStringShouldThrowArgumentException`
- [Operação_Cenário_Expectativa] `Add_EmptyString_ThrowsArgumentException`

_[Em construção...]_

### Padrão Arrange-Act-Assert - AAA

O padrão Arrange-Act-Assert é uma forma de se organizar o código de um teste. O código segue o seguinte formato:
- Código para o arrange, que é o código de preparação, que antecede à ação e fornece aquilo que será utilizado no código de ação;
- Código para o act, que é efetivamente a invocação do código a ser testado;
- Código para o assert, que é o código para as verificações que incidem sobre o resultado da ação, e que representa as expectativas do desenvolvedor.

Exemplos:

_Teste de igualdade no padrão AAA com xUnit:_
```csharp
[Fact]
public void Add_EmptyString_ReturnsZero()
{
    // Arrange
    var stringCalculator = new StringCalculator();

    // Act
    var actual = stringCalculator.Add("");

    // Assert
    Assert.Equal(0, actual);
}
```

_Teste de exceção no padrão AAA com xUnit:_
```csharp
[Fact]
public void Add_NullString_ThrowsArgumentNullException()
{
    // Arrange
    var stringCalculator = new StringCalculator();
    var number = null;

    // Act + Assert
    Assert.Equal<ArgumentNullException>(() => stringCalculator.Add(number));
}
```

### Técnica do projeto _Automapper_
O cenário é uma classe que recebe os múltiplos casos de teste.

### Padrão Test-Driven Design - TDD
O teste orienta a codificação, de forma que a implementação do código precisa passar no teste.

