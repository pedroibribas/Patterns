# O Padrão Repositório (Repository)

> Baseado em [The Repository Pattern](https://learn.microsoft.com/en-us/previous-versions/msp-n-p/ff649690(v=pandp.10))

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

**O Repositório.**
- **Conceito:** Um repositório assume a lógica de interação com a fonte de dados da aplicação, isto é, a lógica que recupera os dados da fonte de dados - _query_ -, mapeia os dados obtidos a uma entidade do negócio - _mapping_ - e persiste mudanças da entidade na fonte de dados - _persistence_.
- **Vantagens:** O repositório permite centralizar a lógica de acesso a dados; provê um ponto próprio para testes unitários; e provê uma arquitetura flexível para se adaptar conforme o design do resto da aplicação evolui; A lógica de negócio deveria ser agnóstica ao tipo de dados que constitui a camada da fonte dos dados, que poderia ser uma base-de-dados, uma lista SharePoint ou um serviço Web, ou qual linguagem é usada para recuperar os dados, se SQL, ou CAML, bastando que a camada de regra de negócio apenas use a interface do repositório específico.
- **Fluxos:** As formas pelas quais o repositório atua em relação ao cliente são as seguintes:
1) Query: **Business Layers** > **Repository** > query > **Data Source Layer** > set of entities => **Repository** > business entity > **Business Layers**
2) Persistence: **Business Layers** > new/changed entities > **Repository** > **Data Source Layer**
-**Unit of Work:** Em situações complexas, a lógica negocial do cliente pode usar do padrão **Unit Of Work**, pelo qual é possível encapsular várias operações relacionadas que deveriam ser consistentes entre si ou que têm dependências relacionadas, e que submete itens ao repositório para que este proceda com seus comandos sobre a fonte de dados.
- **Data Mapper:** O repositório atua como um mediador entre diferentes domínios, como é o caso da fonte de dados e a lógica de negócio que age sobre a entidade. Contudo, a 'tipagem' entre esses domínios são diferentes, e as consultas apropriadas à fonte ou a exposição dos dados recuperados às entidades negociais precisam ser tratadas, isto é, traduzidas entre representações específicas, o que pode ser realizado pelo padrão de Data Mapper.
