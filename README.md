# Mini OLX com Leilao (Projeto C14)

API REST de marketplace de anuncios com modulo de leilao integrado, desenvolvida em Python com foco em consistencia sob concorrencia.

---

## 1. Visao Geral

O projeto consiste em uma plataforma de marketplace onde usuarios podem cadastrar produtos para venda direta ou disponibiliza-los em formato de leilao com tempo determinado. No modulo de leilao, compradores podem realizar lances competitivos dentro de regras estritas de incremento e janelas temporais, com encerramento automatico e apuracao do vencedor.

A aplicacao e estritamente uma API REST (backend), sem interface grafica acoplada, projetada para consumo por clientes HTTP, documentada via OpenAPI/Swagger e validada por testes automatizados.

---

## 2. Objetivos Arquiteturais

O modulo de leilao introduz desafios tecnicos essenciais que orientam as decisoes de design de software:

- **Maquina de Estados:** Controle rigoroso dos estados do leilao (`agendado` -> `aberto` -> `encerrado` -> `pago` ou `cancelado`).
- **Validacao e Concorrencia:** Tratamento de condicoes de corrida e concorrencia real entre lances simultaneos por meio de transacoes de banco de dados com locks.
- **Isolamento de Dominio:** Nucleo de regras de negocio 100% puro em Python, sem dependencia de frameworks web ou ORM.
- **Processamento Assincrono:** Encerramento automatico via jobs agendados e publicacao de eventos desacoplados (`LanceRealizado`, `LeilaoEncerrado`).

---

## 3. Arquitetura

O sistema adota os principios da **Clean Architecture** combinados com **Event-Driven Architecture**:

```text
domain/
  Entidades e regras puras (Usuario, Anuncio, Leilao, Lance, Categoria)
  Sem dependencia de Flask ou SQLAlchemy

use_cases/
  Casos de uso da aplicacao (CriarAnuncio, IniciarLeilao, DarLance, EncerrarLeilao)

adapters/
  repositories/  Interfaces e implementacoes concretas de persistencia
  events/        Publisher e Subscriber de eventos de dominio

infra/
  flask_app/     Rotas, controllers, serializacao e autenticacao JWT
  db/            Models SQLAlchemy e migracoes
  jobs/          Rotinas agendadas para encerramento de leiloes
```

### Fluxo de Dependencias

O dominio reside no centro da arquitetura e nao possui dependencias externas:

```text
[Dominio do Leilao] <----- [Casos de Uso]
       ^                          ^
       |                          |
[Eventos e Jobs]          [Infraestrutura e Rotas]
```

---

## 4. Modelo de Dominio

### Entidades Principais

- **Usuario:** Gestao de conta, credenciais e papeis (comprador/vendedor).
- **Anuncio:** Cadastro de produtos, categorias e definicao do modelo de negociacao (venda direta ou leilao).
- **Leilao:** Vinculado a um anuncio, com preco inicial, incremento minimo, data/hora de inicio, data/hora de termino e status.
- **Lance:** Registro de oferta financeira vinculada a um usuario e leilao em determinado instante.
- **Categoria:** Classificacao e organizacao dos produtos.

### Maquina de Estados do Leilao

- `agendado`: Leilao criado, aguardando horario de abertura.
- `aberto`: Leilao em andamento, aceitando lances validos.
- `encerrado`: Periodo de lances finalizado; apura-se o maior lance valido.
- `pago`: Arrematante efetuou o pagamento do lote.
- `cancelado`: Leilao encerrado sem lances validos ou interrompido.

---

## 5. Stack Tecnologica

| Camada / Finalidade | Tecnologia |
| --- | --- |
| Linguagem | Python 3.11+ |
| Framework Web | Flask + Flask-RESTful |
| ORM e Migracoes | SQLAlchemy + Flask-Migrate |
| Autenticacao | Flask-JWT-Extended |
| Banco de Dados | PostgreSQL (Docker) |
| Filas e Jobs Assincronos | Celery + Redis (ou APScheduler) |
| Gerenciador de Dependencias | Poetry |
| Testes Automatizados | pytest + pytest-flask + pytest-cov |
| Documentacao da API | Swagger / OpenAPI (Flasgger) |
| Integracao Continua (CI/CD) | Jenkins (Jenkinsfile) |

---

## 6. Estrutura da Equipe e Responsabilidades

| Integrante | Modulo | Atribuicoes Principais |
| --- | --- | --- |
| **Teo** | Dominio do Leilao | Entidades de dominio, maquina de estados, regras de validacao de lances, concorrencia e testes unitarios de dominio. |
| **Pedro Vitor** | Catalogo e Usuarios | CRUD de anuncios e categorias, filtros de busca, autenticacao JWT e controle de permissoes. |
| **Pedro** | Eventos e Automacao | Barramento de eventos (Event Bus), jobs de encerramento automatico e historico de lances. |
| **Caio** | Infraestrutura e DevOps | Implementacao dos Repositories, models SQLAlchemy, rotas Flask, documentacao Swagger e pipeline de CI/CD. |

### Responsabilidades Transversais

- Revisao cruzada obrigatoria de Pull Requests.
- Criacao e manutencao de testes de integracao.
- Definicao de jobs individuais na pipeline de CI/CD.
- Documentacao de decisoes de arquitetura e utilizacao de IA.

---

## 7. Metodologia e Qualidade

- **Fluxo de Trabalho:** Sprints curtas (1 a 2 semanas) com quadro Kanban/Scrum.
- **Definition of Done (DoD):** Codigo testado com pytest, sem violacoes de contrato, aprovado em PR e documentado na especificacao OpenAPI.
- **Rastreabilidade:** Historias de usuario mapeadas diretamente para branches, commits, PRs e suites de testes.
