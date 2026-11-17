# ABALIST | Histórico de preços em um catálogo de materiais

O ABALIST é um aplicativo desktop para consultar materiais e preços e montar listas e orçamentos. Este estudo de caso acompanha uma operação do catálogo: alterar o preço de um fornecedor e preservar a evolução daquele valor.

## Minha contribuição

Defini a arquitetura em camadas e conduzi a implementação com apoio do Codex. Neste fluxo, a interface Avalonia envia a solicitação à API, o domínio valida o preço e o serviço coordena a gravação. Essa divisão mantém a regra de preço no domínio e permite exercitá-la sem iniciar a interface ou o banco.

## Decisão de arquitetura

O catálogo separa **material**, **oferta do fornecedor** e **histórico de preço**. O material identifica o item. A oferta guarda o código usado pelo fornecedor e o preço unitário vigente. O histórico registra o valor anterior, o novo valor, a origem, a data e o responsável pela alteração.

Com esse modelo, um mesmo material pode ter ofertas de vários fornecedores. O catálogo consulta o preço vigente diretamente na oferta, enquanto a consulta de histórico recupera as alterações por material ou fornecedor. Em uma alteração efetiva, o serviço grava o novo preço e a entrada de histórico em uma única chamada de persistência do Entity Framework Core.

## Da tela ao banco

~~~mermaid
sequenceDiagram
    participant Desktop as Desktop Avalonia
    participant API as API ASP.NET Core
    participant Service as Serviço de catálogo
    participant Domain as Domínio do catálogo
    participant DB as SQLite

    Desktop->>API: Solicita alteração do preço
    API->>Service: Encaminha novo preço e usuário
    Service->>DB: Carrega oferta
    DB-->>Service: Preço vigente
    alt Mesmo preço
        Service-->>API: Encerra sem gravar
    else Preço diferente
        Service->>Domain: Valida e aplica novo valor
        alt Preço válido
            Domain-->>Service: Oferta atualizada
            Service->>DB: Salva oferta e histórico via EF Core
            DB-->>Service: Gravação concluída
            Service-->>API: Conclui operação
        else Preço negativo
            Domain-->>Service: Rejeita alteração
        end
    end
~~~

## Exemplos com dados fictícios

| Ação | Preço vigente após a operação | Histórico |
| --- | --- | --- |
| Cadastrar uma oferta a R$ 20,00 | R$ 20,00 | Registra o preço inicial |
| Alterar a oferta de R$ 20,00 para R$ 24,50 | R$ 24,50 | Registra os dois valores e quem alterou |
| Enviar R$ 24,50 novamente | R$ 24,50 | Mantém as entradas existentes |
| Enviar um preço negativo | R$ 24,50 | A validação do domínio impede a gravação |

Os exemplos descrevem o comportamento verificado por leitura da implementação.
