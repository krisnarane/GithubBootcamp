# Library App

## Descrição

Library App é uma aplicação modular desenvolvida para gerenciar operações de biblioteca, como empréstimos de livros, gestão de usuários (patrons) e controle de inventário. Construída em .NET, segue o padrão de arquitetura limpa (Clean Architecture), garantindo escalabilidade, organização e facilidade de manutenção.

## Estrutura do Projeto

- `AccelerateDevGHCopilot.sln` - Arquivo de solução principal do projeto.
- `src/`
  - `Library.ApplicationCore/`
    - `Entities/` - Entidades centrais do domínio (ex: Livro, Usuário, Empréstimo).
    - `Enums/` - Enumerações utilizadas em toda a aplicação.
    - `Interfaces/` - Interfaces para abstração das operações principais.
    - `Services/` - Implementação das regras de negócio e serviços de domínio.
    - `Library.ApplicationCore.csproj` - Projeto do núcleo da aplicação.
  - `Library.Console/`
    - `appSettings.json` - Arquivo de configuração da aplicação console.
    - `CommonActions.cs` - Enumerações de ações reutilizáveis no console.
    - `ConsoleApp.cs` - Lógica principal da interface de console.
    - `ConsoleState.cs` - Gerencia os estados da aplicação console.
    - `Program.cs` - Ponto de entrada da aplicação console.
    - `Json/` - Dados e utilitários em formato JSON (livros, usuários, empréstimos, etc).
    - `Library.Console.csproj` - Projeto da aplicação console.
  - `Library.Infrastructure/`
    - `Data/` - Implementações de acesso a dados (repositórios, leitura/gravação de JSON).
    - `Library.Infrastructure.csproj` - Projeto da camada de infraestrutura.
- `tests/`
  - `UnitTests/`
    - `LoanFactory.cs` - Fábrica para criação de dados de teste de empréstimos.
    - `PatronFactory.cs` - Fábrica para criação de dados de teste de usuários.
    - `ApplicationCore/` - Testes unitários do núcleo da aplicação.
    - `UnitTests.csproj` - Projeto de testes unitários.

## Principais Classes e Interfaces

- **Entidades**
  - `Book` - Representa um livro na biblioteca.
  - `Patron` - Representa um usuário da biblioteca.
  - `Loan` - Representa uma transação de empréstimo.
- **Interfaces**
  - `IBookRepository` - Interface para operações de dados de livros.
  - `IPatronRepository` - Interface para operações de dados de usuários.
  - `ILoanService` - Interface para gerenciamento de empréstimos.
- **Serviços**
  - `LoanService` - Implementa a lógica de negócios relacionada a empréstimos.
  - `NotificationService` - Gerencia notificações de empréstimos em atraso.

# Diagrama UML - Mermaid
```mermaid
classDiagram
    %% Entidades
    class Author {
        +Id: int
        +Name: string
        +Biography: string
        +Books: List~Book~
    }

    class Book {
        +Id: int
        +Title: string
        +ISBN: string
        +AuthorId: int
        +Author: Author
        +BookItems: List~BookItem~
    }

    class BookItem {
        +Id: int
        +Barcode: string
        +IsAvailable: bool
        +BookId: int
        +Book: Book
        +Loans: List~Loan~
    }

    class Patron {
        +Id: int
        +Name: string
        +Email: string
        +MembershipExpiration: DateTime
        +Loans: List~Loan~
    }

    class Loan {
        +Id: int
        +LoanDate: DateTime
        +DueDate: DateTime
        +ReturnDate: DateTime?
        +Status: LoanStatus
        +BookItemId: int
        +BookItem: BookItem
        +PatronId: int
        +Patron: Patron
        +Extensions: List~LoanExtension~
    }

    %% Enums
    class LoanStatus {
        <<enumeration>>
        Active
        Returned
        Overdue
        Lost
    }

    class LoanExtensionStatus {
        <<enumeration>>
        Approved
        Denied
        Pending
    }

    class MembershipRenewalStatus {
        <<enumeration>>
        Success
        Failed
        Pending
    }

    %% Interfaces de Serviço
    class ILoanService {
        <<interface>>
        +BorrowBook(patronId: int, bookItemId: int): Loan
        +ReturnBook(loanId: int): LoanReturnStatus
        +ExtendLoan(loanId: int): LoanExtensionStatus
        +GetActiveLoans(patronId: int): List~Loan~
    }

    class IPatronService {
        <<interface>>
        +RegisterPatron(patron: Patron): Patron
        +RenewMembership(patronId: int): MembershipRenewalStatus
        +GetPatronDetails(patronId: int): Patron
    }

    %% Interfaces de Repositório
    class ILoanRepository {
        <<interface>>
        +Add(loan: Loan): void
        +GetById(id: int): Loan?
        +GetActiveLoans(patronId: int): List~Loan~
        +Update(loan: Loan): void
    }

    class IPatronRepository {
        <<interface>>
        +Add(patron: Patron): void
        +GetById(id: int): Patron?
        +Update(patron: Patron): void
    }

    %% Implementações
    class LoanService {
        -_loanRepository: ILoanRepository
        +BorrowBook(patronId: int, bookItemId: int): Loan
        +ReturnBook(loanId: int): LoanReturnStatus
        +ExtendLoan(loanId: int): LoanExtensionStatus
        +GetActiveLoans(patronId: int): List~Loan~
    }

    class PatronService {
        -_patronRepository: IPatronRepository
        +RegisterPatron(patron: Patron): Patron
        +RenewMembership(patronId: int): MembershipRenewalStatus
        +GetPatronDetails(patronId: int): Patron
    }

    class JsonLoanRepository {
        -_jsonData: JsonData
        +Add(loan: Loan): void
        +GetById(id: int): Loan?
        +GetActiveLoans(patronId: int): List~Loan~
        +Update(loan: Loan): void
    }

    class JsonPatronRepository {
        -_jsonData: JsonData
        +Add(patron: Patron): void
        +GetById(id: int): Patron?
        +Update(patron: Patron): void
    }

    %% Relacionamentos
    Author "1" -- "*" Book: writes
    Book "1" -- "*" BookItem: has copies
    BookItem "1" -- "*" Loan: is borrowed in
    Patron "1" -- "*" Loan: has
    Loan "1" -- "*" LoanExtension: has

    %% Implementações de interfaces
    ILoanService <|.. LoanService: implements
    IPatronService <|.. PatronService: implements
    ILoanRepository <|.. JsonLoanRepository: implements
    IPatronRepository <|.. JsonPatronRepository: implements

    %% Dependências
    LoanService --> ILoanRepository: depends on
    PatronService --> IPatronRepository: depends on
    JsonLoanRepository --> JsonData: uses
    JsonPatronRepository --> JsonData: uses
```

## Como Usar

1. Clone o repositório:

   ```bash
   git clone https://github.com/krisnarane/GithubBootcamp
   ```

2. Abra o arquivo de solução `AccelerateDevGHCopilot.sln` no Visual Studio.

3. Compile a solução para restaurar as dependências e gerar os binários.

4. Execute a aplicação console:

   ```bash
   dotnet run --project src/Library.Console/Library.Console.csproj
   ```

5. Execute os testes unitários:

   ```bash
   dotnet test tests/UnitTests/UnitTests.csproj
   ```

## Funcionalidades

- Busca e cadastro de usuários (patrons)
- Consulta e empréstimo de livros
- Controle de devoluções e renovações
- Visualização do histórico de empréstimos
- Notificações para empréstimos em atraso
- Persistência dos dados em arquivos JSON

## Licença

Este projeto está licenciado sob a Licença MIT. Veja o arquivo `LICENSE` para mais detalhes.
