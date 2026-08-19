# Divisão de Tarefas e Responsabilidades (Atualizado)

Baseado na arquitetura em camadas (Clean Architecture) e no modelo focado em API REST (sem frontend) estabelecido no `Doc Inicial.md`, a divisão de tarefas original do arquivo `Sem título.md` foi atualizada. O sistema deixa de ter um frontend ("Full Stack") e o foco passa a ser o backend, com separação por módulos da Clean Architecture.

A equipe é composta por 4 integrantes: **Téo**, **Caio**, **Pedro Vitor** e **Pedro**.

---

### Divisão por Módulos (Clean Architecture)

| Integrante | Módulo | Responsabilidades Principais |
| --- | --- | --- |
| **Téo** | **Domínio do leilão** | Entidades, máquina de estados, validação de lance, concorrência, testes unitários de domínio. |
| **Pedro Vitor** | **Catálogo e usuários** | CRUD de anúncios, categorias, busca/filtro, autenticação (JWT) e permissões. |
| **Pedro** | **Eventos e automação** | Event bus interno, job de encerramento automático, notificações. |
| **Caio** | **Infraestrutura** | Repository pattern, persistência, rotas Flask, documentação Swagger, pipeline de CI/CD. |

---

### Visão Detalhada das Entregas (Backend API)

Neste modelo, cada pessoa atua em sua camada e se integra com as demais, garantindo o desacoplamento.

**Téo — Domínio do Leilão (Core da Regra de Negócio)**
```text
Domínio e Regras (100% isolado)
├── Entidades: Leilao, Lance (pura lógica Python)
├── Casos de Uso: IniciarLeilao, DarLance
├── Lógica:
│   ├── Máquina de estados (agendado → aberto → encerrado → pago/cancelado)
│   ├── Validação de lances (incremento mínimo, dentro da janela de tempo)
│   └── Tratamento de concorrência genuína via transação/lock
└── Testes: Testes unitários puros, sem dependência de banco ou Flask.
```

**Pedro Vitor — Catálogo e Usuários (Sustentação do Domínio)**
```text
Catálogo e Gestão de Usuários
├── Domínio/Use Cases: Usuario, Anuncio, Categoria
├── Infra:
│   ├── Rotas e controllers para criação de anúncios (venda direta ou leilão)
│   ├── Buscas e filtros
│   └── Integração com Flask-JWT-Extended para autenticação
└── Telas: NENHUMA (Consumível via Postman/Swagger).
```

**Pedro — Eventos e Automação (Assincronismo)**
```text
Motor Assíncrono e Event-Driven
├── Eventos:
│   ├── Publisher/Subscriber de eventos (ex: LanceRealizado, LeilaoEncerrado)
├── Jobs:
│   ├── Encerramento automático via job agendado ao atingir data/hora fim
│   └── Configuração do Celery + Redis ou APScheduler
└── Histórico: Geração de histórico de lances a partir de eventos
```

**Caio — Infraestrutura e DevOps (Base Técnica)**
```text
Infraestrutura e Integração
├── Banco de Dados:
│   ├── Models SQLAlchemy e Flask-Migrate
│   ├── Implementação dos Repositories concretos (ex: SQLAlchemyLeilaoRepository)
├── API e Documentação:
│   ├── Estrutura base do Flask + Flask-RESTful
│   ├── Documentação OpenAPI/Swagger (Flasgger)
└── DevOps:
    ├── Pipeline de CI/CD (Jenkins / Jenkinsfile)
    └── Configurações globais (Poetry, Docker)
```

---

### Como o trabalho se conecta na prática

O sistema cresce respeitando a regra de dependência da Clean Architecture, em vez de "Vertical Slices". O Domínio é o núcleo.

```text
[Domínio do Leilão (Téo)] ◄───── [Casos de Uso (Todos)]
       ▲                                 ▲
       │                                 │
[Eventos/Jobs (Pedro)]         [Infra/Rotas (Caio / Pedro Vitor)]
```
*(As setas indicam o sentido das dependências. A infraestrutura e os eventos conhecem o domínio, mas o domínio não conhece ninguém).*

---

### Responsabilidades Secundárias (Transversais)

* **Compartilhado entre todos:** Testes de integração (pytest), README, histórias de usuário, seção de Uso de IA.
* **Revisão de Código:** Revisão cruzada de pull requests obrigatória.
* **CI/CD:** Cada integrante deve comitar **pelo menos 1 job próprio** na pipeline de CI/CD, conforme exigido pelo edital.
* **Ordem de Desenvolvimento:** O Domínio (Téo) deve ser construído primeiro, já que as demais áreas dependem dele. Isso evita bloqueios.
