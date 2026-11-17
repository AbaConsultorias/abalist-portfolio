# Sistema desktop de materiais e orçamentos

Aplicação desktop para centralizar o cadastro de materiais, a consulta de preços e a montagem de listas e orçamentos. A solução também oferece gestão de tarefas e registro de jornada.

## Minha contribuição

Defini a arquitetura do sistema e conduzi a implementação com apoio do Codex. Estruturei a solução em camadas para separar a interface desktop, a API, as regras de negócio e a persistência. Usei o Codex como assistente de implementação, orientando como as funcionalidades deveriam se encaixar nessa estrutura.

## Funcionalidades

- Consultar materiais e preços para compor listas e orçamentos.
- Organizar tarefas e acompanhar atividades vinculadas a empreendimentos.
- Registrar marcações de ponto e consultar o calendário de jornada.

## Arquitetura

A interface desktop foi construída com C# e Avalonia. A API ASP.NET Core expõe as operações da aplicação. As regras de negócio ficam na camada de domínio, e a infraestrutura usa Entity Framework Core e SQLite para persistência. O projeto inclui testes automatizados com xUnit.

A aplicação conta com um atualizador desktop. O fluxo de distribuição usa pacotes assinados, e o ambiente de homologação usa contas e dados fictícios.
