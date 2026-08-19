Dividir o projeto por **domínios** (ou *vertical slices*) é uma excelente estratégia para o ambiente acadêmico. Dessa forma, todos os integrantes atuam como desenvolvedores *Full Stack* em suas respectivas áreas, precisando entender como o banco de dados, a regra de negócio e a interface se conectam de ponta a ponta.

Aqui está uma proposta de divisão focada nas funcionalidades (domínios) do sistema de leilão:

### Divisão por Domínios (Vertical Slices)

| Pessoa          | Domínio de Responsabilidade           | O que desenvolve de ponta a ponta (Front, Back e Banco)                                                                            |
| --------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **Téo**         | **Identidade e Usuários**             | Autenticação, cadastro de contas, perfis de usuário, permissões (admin vs. cliente) e segurança das rotas.                         |
| **Caio**        | **Catálogo e Gestão de Leilões**      | Criação dos leilões, cadastro de lotes/produtos, regras de agendamento, status do leilão (rascunho, publicado) e listagem/vitrine. |
| **Pedro Vitor** | **Motor de Lances e Tempo Real**      | A sala de leilão ao vivo, validação e registro de lances, incremento mínimo, **WebSocket**, cronômetro e prorrogação de tempo.     |
| **Pedro**       | **Pós-Leilão, Histórico e Auditoria** | Encerramento do leilão, apuração do vencedor, geração do histórico de lances, logs de auditoria e painel de resultados.            |
|                 |                                       |                                                                                                                                    |

---

### Visão Detalhada das Entregas (Full Stack)

Neste modelo, cada pessoa cria a tabela no banco, escreve a API e desenha a tela correspondente à sua funcionalidade.

**Pessoa 1 — Domínio de Identidade**

```text
Identidade e Usuários
├── Banco de Dados: Tabela de Usuários e Tokens
├── Backend: 
│   ├── API de Login e Cadastro
│   ├── Criptografia de senhas
│   └── Middlewares de Autenticação
└── Frontend:
    ├── Telas de Login e Cadastro
    └── Tela de "Meu Perfil"

```

**Pessoa 2 — Domínio de Catálogo**

```text
Catálogo e Leilões
├── Banco de Dados: Tabelas de Leilões e Produtos/Lotes
├── Backend:
│   ├── CRUD de Leilões e Produtos
│   └── Regras de transição de status (Agendado -> Ativo)
└── Frontend:
    ├── Painel de criação de Leilões (Admin)
    └── Página inicial (Vitrine de leilões ativos/futuros)

```

**Pessoa 3 — Domínio de Lances (Core do Leilão)**

```text
Motor de Lances
├── Banco de Dados: Tabela de Lances (Bids)
├── Backend:
│   ├── API de validação de lances
│   ├── Controle de concorrência e Lock
│   └── Servidor WebSocket (Emissão de eventos)
└── Frontend:
    ├── A "Sala do Leilão"
    ├── Atualização do cronômetro na tela
    └── Botão de lance dinâmico

```

**Pessoa 4 — Domínio de Conclusão e Histórico**

```text
Pós-Leilão e Auditoria
├── Banco de Dados: Tabelas de Vencedores, Logs e Auditoria
├── Backend:
│   ├── Algoritmo de encerramento e definição de vencedor
│   ├── API de histórico e relatórios
│   └── Registro de logs de segurança
└── Frontend:
    ├── Tela de "Meus Arremates" (Visão do cliente)
    ├── Histórico completo do lote
    └── Painel de auditoria (Visão Admin)

```

---

### Como o trabalho se conecta na prática

Como todos estão construindo pedaços verticais, o sistema cresce como um quebra-cabeça. As APIs precisam conversar entre si.

```text
 [Identidade] ──────► (Fornece autenticação para todos) 
      │
      ▼
 [Catálogo] ────────► (Fornece os produtos para a sala ao vivo)
      │
      ▼
  [Lances] ─────────► (Processa a ação principal do usuário logado no produto)
      │
      ▼
[Pós-Leilão] ───────► (Lê os lances finalizados e o catálogo para gerar o resultado)

```

### Responsabilidades Secundárias (Transversais)

Para garantir que o projeto não vire um "Frankenstein" (onde a tela de um membro não combina com a do outro, ou os códigos têm padrões diferentes), você pode manter **papéis de governança transversais**.

Todos programam em todas as camadas, mas cada um "guarda" um aspecto da qualidade geral:

* **Pessoa 1 (Guardião de UI/UX):** Garante que o CSS, os botões e os componentes visuais de todas as telas sigam o mesmo padrão e identidade visual.
* **Pessoa 2 (Guardião do Repositório):** Responsável por revisar os Pull Requests, gerenciar os *branches* no Git e garantir que não haja conflitos de código ao juntar as partes.
* **Pessoa 3 (Guardião da Arquitetura e Integração):** Define como será a estrutura de pastas do projeto, os padrões de nomenclatura e garante que os contratos das APIs estejam consistentes.
* **Pessoa 4 (Guardião do Banco e Qualidade):** Responsável por revisar a modelagem geral do banco (para evitar dados duplicados entre os domínios) e liderar os testes da aplicação integrada.