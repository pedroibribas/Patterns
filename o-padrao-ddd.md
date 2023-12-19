# Pastas

## Introdução

Organizar o projeto em pastas pode melhorar a **manutenção** do código. Isso torna o código **modular**, e aí partes do código ficam isoladas, o que facilitar a modificação sem afetar outras partes do projeto. Também facilita a colaboração entre a equipe, permitindo que cada desenvolvedor atue em uma parte específica do projeto.

Como, por exemplo, separar em uma pasta específica classes com a lógica de negócios -- **Models**; com interface do usuário -- **Commands**; classes utilitárias -- **Utils**; classes de teste -- **Tests**; classes que se comunicam com a API -- **Services**; etc.

```
|-- Commands
|-- Models
|-- Services
|-- Tests
|-- Utils
```

Contudo, é importante evitar criar uma hierarquia de pastas muito complexa, senão o projeto acaba ficando difícil de entender e manter. É necessário um equilíbrio entre a organização do projeto e a simplicidade para manter a eficiência e a qualidade do código. A hierarquia muito complexa pode ser confusa para novos membros da equipe, levar a erros e dificultar a colaboração.

## Diretrizes
Segue algumas diretrizes de organização do projeto em pastas:
- Manter a estrutura de pastas o mais simples possível. Uma boa maneira de fazer isso é agrupar arquivos semelhantes em pastas relacionadas -- em vez de criar pastas para cada arquivo individual.
- Evitar pastas desnecessárias. Cada pasta no projeto precisa de um propósito claro, e todos os arquivos dentro dela se relacionam a esse propósito.
- Sempre considere a **escalabilidade** do projeto ao organizar pastas. A estrutura de pastas deve lidar com o crescimento futuro do projeto e permitir a fácil adição e remoção de arquivos conforme necessário.
- Embora a organização do projeto em pastas possa ser útil, o mais importante é manter um código limpo, legível e bem documentado.

## sample
```shell
Solution
    |-- Presentation
        |-- Controllers
        |-- ViewModels
        |-- Views
    |-- Domain
        |-- Interfaces
            |-- Services
            |-- Repositories
        |-- Dtos
        |-- Validations
        |-- Services
        |-- Entities
        |-- Enums
    |-- Data
        |-- Context
        |-- Repositories
        |-- Migrations
        |-- CrossCutting
    |-- Tests
```

## var 1: API Web
```shell
Solution
    |-- Api
        |-- Controllers
        |-- Models
    ...
```

## var 2: removendo responsabilidades do Domain
```shell
...
    |-- Domain
        |-- Interfaces
        |-- Dtos
        |-- Validations
    |-- Entities
        |-- Enums
    |-- Application
        |-- Services
```

## conceitos
- Solution: solução .NET; ela reúne um conjunto de projetos.
- Domain (ou Core?): esfera de conhecimento e atividade ao redor do qual a aplicação se desenvolve. Ela apresenta: contratos, entidades, serviços, objetos de valor e fábricas. Ela acaba que não conhece nenhuma outra.
- Presentation: aplicação web completa, com API Web e UI.
- Entities (entidades): regras de negócio.
- Interfaces: contratos.
- Dtos: classes p/ transferência de dados usadas por controladores.
- Data (ou Infra, ou Infrastructure): classes responsáveis por implementar os contratos do domínio relativos aos repositórios.