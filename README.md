# ABA APP | Suíte desktop de gestão operacional

**C# · .NET 10 · Avalonia · ASP.NET Core · EF Core · SQLite · Google Calendar API**

O ABA APP reúne em um aplicativo desktop os fluxos de materiais e orçamentos, gestão de tarefas, controle de jornada, agenda corporativa, comunicação e acesso a documentos.

## Minha contribuição

Defini a arquitetura da suíte e conduzi a implementação com apoio do Codex. Organizei a solução em módulos com responsabilidades próprias e camadas para interface, contratos da aplicação, regras de negócio e persistência. Durante a implementação, orientei como cada fluxo se encaixaria nessa estrutura.

## Módulos

| Área | O que a equipe consegue fazer |
| --- | --- |
| **ABALIST · Materiais e orçamentos** | Consultar o catálogo e as ofertas de fornecedores, acompanhar o histórico de preços e montar listas e kits de materiais para orçamentos. |
| **ABATASK · Tarefas e empreendimentos** | Organizar tarefas por responsável e empreendimento, acompanhar entregáveis e lembretes, e visualizar prazos e agendamentos em painel ou calendário. |
| **ABAPONTOS · Jornada e RH** | Registrar marcações, consultar jornada e saldo, solicitar correções e justificativas, acompanhar férias e apoiar a conferência e o fechamento mensal. |
| **Calendário · Google Calendar API** | Consultar a agenda conectada e criar, editar ou excluir compromissos pelo desktop. A conexão da conta é administrada no aplicativo. |
| **ABACALL · Comunicação interna** | Entrar em salas de chamada, trocar de sala e participar de reuniões com compartilhamento de tela. A sala inicial pode ser definida por setor ou cargo. |
| **ABASERVIDOR · Documentos** | Navegar pela biblioteca de arquivos da equipe e baixar documentos conforme as permissões. Operações de escrita dependem da configuração do ambiente. |
| **ABA Financeiro · Em homologação** | Montar cronogramas de pagamento por empreendimento e acompanhar o resultado mensal da empresa, com valores previstos e realizados em separado. |

## Arquitetura da solução

~~~mermaid
flowchart LR
    Desktop["Desktop · Avalonia"] -->|"HTTP"| API["API · ASP.NET Core"]
    API --> Application["Contratos e serviços da aplicação"]
    Application --> Domain["Entidades e regras de negócio"]
    API --> Infrastructure["Implementações de infraestrutura"]
    Infrastructure --> Domain
    Infrastructure --> SQLite[("SQLite · EF Core")]
    Infrastructure --> Files["Biblioteca de documentos"]
    API --> Calendar["Integração OAuth 2.0"]
    Calendar --> Google["Google Calendar API"]
~~~

O desktop apresenta as áreas de trabalho e consome a API. O domínio concentra regras como a validação de preço e o cálculo de jornada. A infraestrutura implementa a persistência e o acesso à biblioteca de documentos. A integração com o Google é intermediada pelo servidor, que conduz a autorização e renova os tokens para consultar e alterar eventos.

## Decisões técnicas na prática

### Materiais · Preço atual com histórico

Material, oferta e histórico têm responsabilidades separadas. Um mesmo item pode ter ofertas de vários fornecedores. Cada oferta guarda seu preço vigente, enquanto o histórico registra o valor anterior, o novo valor, a data e o responsável. O domínio rejeita preços negativos e o serviço grava a oferta e a alteração de histórico na mesma operação de persistência.

**Exemplo fictício.** Uma oferta passa de R$ 20,00 para R$ 24,50. A consulta exibe R$ 24,50 como preço vigente, e o histórico preserva os dois valores. Reenviar R$ 24,50 mantém o histórico como está.

### Pontos · Jornada registrada e saldo creditado

O cálculo separa o tempo trabalhado a partir das marcações do saldo reconhecido pelas regras de aprovação. A aprovação de uma correção de ponto e a aprovação das horas extras são decisões independentes.

**Exemplo fictício.** Em uma jornada prevista de 8 horas, a tela pode mostrar 8h30 trabalhadas e 0h de saldo creditado enquanto os 30 minutos extras estiverem sem aprovação.

### Tarefas · Prazo de entrega e horário de execução

Cada tarefa pode ter um prazo de entrega e um intervalo de execução próprios. O painel agrupa o trabalho que está em execução, o que está atrasado e o que vem depois. O calendário interno usa as mesmas tarefas autorizadas e filtradas pela equipe.

**Exemplo fictício.** Uma visita agendada para terça-feira pode gerar um relatório com entrega na sexta. As duas datas aparecem na tarefa e orientam ações diferentes.

### Calendário · Compromissos do Google no desktop

O módulo Calendário acessa eventos da agenda conectada via Google Calendar API. O administrador inicia a conexão OAuth 2.0 e a API gerencia a troca e a renovação de tokens. Com a agenda conectada, a interface oferece consulta por período e edição de compromissos. As datas de execução e entrega do ABATASK são administradas no calendário interno de tarefas.

## Qualidade e distribuição

A solução inclui testes automatizados de domínio e integração, dados fictícios para homologação e um atualizador desktop que verifica a assinatura dos pacotes. Os cenários acima ilustram o comportamento das regras e dos fluxos implementados.
