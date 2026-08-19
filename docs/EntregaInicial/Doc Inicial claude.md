# Mini OLX com Leilão — Resumo do Projeto (C14 - NP2)

## 1. Visão geral

**Tema:** marketplace de anúncios (estilo OLX) com módulo de **leilão** para determinados itens — usuários podem cadastrar produtos para venda direta ou para leilão, dar lances em tempo limitado, e o sistema encerra o leilão automaticamente definindo o vencedor.

**Tipo de aplicação:** API REST (Web), sem frontend — consumível via Swagger/Postman/testes automatizados.

**Objetivo arquitetural:** o tema serve como contexto mínimo para ancorar histórias de usuário e regras de negócio, mas o foco do projeto está em **arquitetura em camadas, desacoplamento e consistência sob concorrência** — não em complexidade de regra de negócio pela regra em si.

---

## 2. Por que este tema

O leilão introduz uma regra de negócio que **força** boas decisões arquiteturais, evitando que o projeto vire um CRUD raso:

- Máquina de estados real (agendado → aberto → encerrado → pago)
- Validação de lance (maior que o atual + incremento mínimo, dentro da janela de tempo)
- Concorrência genuína: dois lances quase simultâneos exigem tratamento real (lock/transação)
- Encerramento automático via job agendado

O restante do sistema (cadastro de usuário, anúncio, categoria, busca) sustenta o volume de funcionalidades e histórias de usuário sem exigir complexidade extra em todo módulo.

---

## 3. Arquitetura

**Padrão:** Clean Architecture / Arquitetura em Camadas + Event-Driven

```
domain/          → Entidades e regras puras (Usuario, Anuncio, Leilao, Lance)
                   Sem dependência de Flask ou SQLAlchemy — 100% testável isoladamente

use_cases/       → Casos de uso (CriarAnuncio, IniciarLeilao, DarLance, EncerrarLeilao)

adapters/
  repositories/  → Interfaces + implementações concretas (SQLAlchemyLeilaoRepository)
  events/        → Publisher/Subscriber de eventos (LanceRealizado, LeilaoEncerrado)

infra/
  flask_app/     → Rotas, controllers, serialização, autenticação (JWT)
  db/            → Models SQLAlchemy, migrations (Flask-Migrate)
  jobs/          → Job agendado de encerramento automático (Celery ou APScheduler)
```

**Decisões-chave a defender:**

- Domínio isolado permite trocar a camada de persistência sem alterar regra de negócio
- Cada transição de estado do leilão publica um evento, consumido por handlers desacoplados (ex.: notificação, atualização de histórico)
- Concorrência em lances tratada via transação de banco com lock (evita condição de corrida)

---

## 4. Stack tecnológica

|Camada|Tecnologia|
|---|---|
|Backend|Flask + Flask-RESTful|
|ORM / Migrations|SQLAlchemy + Flask-Migrate|
|Autenticação|Flask-JWT-Extended|
|Banco de dados|PostgreSQL (via Docker)|
|Jobs assíncronos|Celery + Redis (ou APScheduler, alternativa mais simples)|
|Gerenciador de dependências|Poetry|
|Testes|pytest + pytest-flask + pytest-cov|
|CI/CD|Jenkins (Jenkinsfile) (GitHub Actions não permitido)|
|Documentação da API|Swagger / OpenAPI (flasgger ou Flask-Smorest)|

---

## 5. Modelo de domínio (entidades principais)

- **Usuario** — comprador/vendedor, autenticação, papel
- **Anuncio** — produto, categoria, tipo (venda direta ou leilão), status
- **Leilao** — associado a um anúncio, preço inicial, incremento mínimo, data/hora de início e fim, status
- **Lance** — valor, usuário, timestamp, associado a um leilão
- **Categoria** — classificação de anúncios

**Máquina de estados do Leilão:**

`agendado → aberto → encerrado → pago (ou cancelado)`

Regras de transição:

- Só aceita lance quando `aberto`
- Encerramento automático ao atingir data/hora fim (job agendado)
- Encerrado sem lances → status `cancelado`
- Encerrado com lances → vencedor definido pelo maior lance válido

---

## 6. Divisão de tarefas (equipe de 4 integrantes)

| Integrante | Módulo              | Responsabilidades                                                                            |
| ---------- | ------------------- | -------------------------------------------------------------------------------------------- |
| **Téo**    | Domínio do leilão   | Entidades, máquina de estados, validação de lance, concorrência, testes unitários de domínio |
| **B**      | Catálogo e usuários | CRUD de anúncios, categorias, busca/filtro, autenticação e permissões                        |
| **C**      | Eventos e automação | Event bus interno, job de encerramento automático, notificações                              |
| **Caio**   | Infraestrutura      | Repository pattern, persistência, rotas Flask, documentação Swagger, pipeline de CI/CD       |

**Compartilhado entre todos:** testes de integração, README, histórias de usuário, seção de Uso de IA, revisão cruzada de pull requests.

Cada integrante deve comitar **pelo menos 1 job próprio** na pipeline de CI/CD, conforme exigido pelo edital.

**Ordem sugerida de desenvolvimento:** domínio (A) primeiro, já que B, C e D dependem dele — evita bloqueios e dá ritmo natural às sprints.

---

## 7. Metodologia sugerida

- **Abordagem:** Scrum simplificado ou Kanban, com sprints de 1-2 semanas
- **Ferramentas:** GitHub Projects ou Trello para board, reuniões curtas de alinhamento
- **Definição de Pronto (DoD):** código revisado via PR, testes passando no CI, endpoint documentado no Swagger
- **Métricas simples:** nº de issues fechadas por sprint, lead time por história

---

## 8. Exemplos de histórias de usuário

1. **Como vendedor**, eu quero cadastrar um anúncio em modo leilão para vender meu produto ao maior lance.
2. **Como comprador**, eu quero dar um lance em um leilão aberto para tentar arrematar o produto.
    - _Given_ leilão aberto com lance atual R$100, _When_ dou lance de R$120, _Then_ lance é aceito e evento `LanceRealizado` é publicado.
3. **Como comprador**, eu quero ser impedido de dar um lance abaixo do mínimo, para manter a integridade do leilão.
4. **Como sistema**, o leilão deve encerrar automaticamente ao atingir o horário definido, definindo o vencedor.
5. **Como comprador**, eu quero consultar o histórico de lances de um leilão para acompanhar a disputa.

_(lista completa com Given/When/Then, prioridade e status deve ser expandida no documento de entrega)_

---

## 9. Cobertura dos requisitos da NP2

|Requisito|Como é atendido|
|---|---|
|Testes automatizados|Testes de domínio (sem banco) + testes de integração via pytest|
|CI/CD sem GitHub Actions|Jenkins (Jenkinsfile), 1 job por integrante|
|Revisão de código|PRs obrigatórios entre módulos dependentes (A→C, D→todos)|
|Histórias de usuário|Mínimo 5, com rastreabilidade história → issue → PR → teste|
|Metodologia|Scrum/Kanban documentado com evidências (board, métricas)|
|Uso de IA|Seção dedicada no README com prompts reais e exemplos|
|Refactoring|Domínio isolado facilita refactors documentáveis via commits/PRs|

---

_Documento vivo — deve ser atualizado conforme o projeto evolui e novas decisões arquiteturais forem tomadas._