# Plano de Implementação — Mesário DigiTech

Sistema de votação escolar digital com painel de mesário, terminais de votação e apuração de resultados.

---

## Modelo de Dados (banco compartilhado: `votacao_digitech`)

> Ambos os módulos (mesario-digitech e votacao-digitech) compartilham o mesmo banco de dados.
> O schema é definido no `schema.sql` do módulo votacao-digitech.

| Tabela            | Colunas                                                                                                 |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| `alunos`          | `id`, `matricula`, `nome`, `email`, `turma`, `tipo`, `senha_hash`                                       |
| `chapas`          | `id`, `numero`, `nome`, `descricao`                                                                     |
| `terminais`       | `id`, `numero`, `nome`                                                                                  |
| `sessoes_votacao` | `id`, `eleitor_id`, `terminal_id`, `liberada`, `votou`, `data_liberacao`, `data_expiracao`, `data_voto` |
| `votos`           | `id`, `eleitor_id`, `chapa_id`, `data_voto`                                                             |
| `eleicoes`        | `id`, `titulo`, `data_inicio`, `data_fim`, `ativa`, `data_criacao`                                      |
| `audit_log`       | `id`, `acao`, `usuario_id`, `ip`, `detalhes`, `data_registro`                                           |
| `rate_limit`      | `id`, `ip`, `data_tentativa`                                                                            |

---

## Tarefas em Ordem de Dependência

### Fase 1 — Infraestrutura e Banco de Dados

- [ ] **1.1 — Criar script SQL de criação do banco e tabelas**
  - Arquivo: `database/schema.sql`
  - Usar banco compartilhado `votacao_digitech` (schema definido no módulo votacao-digitech)
  - Criar tabelas: `alunos`, `chapas`, `terminais`, `sessoes_votacao`, `votos`, `eleicoes`, `audit_log`, `rate_limit`
  - Definir chaves primárias, estrangeiras, índices e constraints (`UNIQUE` em `matricula`, `UNIQUE(eleitor_id)` em `votos` para impedir voto duplo)

- [ ] **1.2 — Criar script SQL de dados iniciais (seed)**
  - Arquivo: `database/seed.sql`
  - Inserir terminais padrão
  - Inserir chapas para teste
  - Inserir alunos de exemplo

---

### Fase 2 — Classes de Domínio

- [ ] **2.1 — Criar classe `Aluno`**
  - Arquivo: `classes/Aluno.php`
  - Responsabilidade: CRUD de alunos

  ```php
  class Aluno {
      public function __construct(PDO $pdo)
      public function listarTodos(): array
      public function buscarPorId(int $id): ?array
      public function buscarPorMatricula(string $matricula): ?array
      public function criar(string $matricula, string $nome, string $turma): bool
      public function atualizar(int $id, string $nome, string $turma): bool
      public function excluir(int $id): bool
      public function importarCSV(string $caminhoArquivo): int  // retorna qtd importada
  }
  ```

- [ ] **2.2 — Criar classe `Chapa`**
  - Arquivo: `classes/Chapa.php`
  - Responsabilidade: CRUD de chapas/candidatos

  ```php
  class Chapa {
      public function __construct(PDO $pdo)
      public function listarTodas(): array
      public function buscarPorId(int $id): ?array
      public function criar(int $numero, string $nome, ?string $foto): bool
      public function atualizar(int $id, int $numero, string $nome, ?string $foto): bool
      public function excluir(int $id): bool
  }
  ```

- [ ] **2.3 — Criar classe `Terminal`**
  - Arquivo: `classes/Terminal.php`
  - Responsabilidade: CRUD e status dos terminais

  ```php
  class Terminal {
      public function __construct(PDO $pdo)
      public function listarTodos(): array
      public function buscarPorId(int $id): ?array
      public function criar(int $numero, ?string $nome): bool
      public function atualizar(int $id, int $numero, ?string $nome): bool
      public function excluir(int $id): bool
      public function obterStatus(int $id): string  // 'livre', 'ocupado', 'votando'
  }
  ```

- [ ] **2.4 — Criar classe `Apuracao`**
  - Arquivo: `classes/Apuracao.php`
  - Responsabilidade: contagem de votos e geração de resultados
  ```php
  class Apuracao {
      public function __construct(PDO $pdo)
      public function contarVotosPorChapa(): array       // [{chapa_id, nome, total_votos}]
      public function totalVotosRegistrados(): int
      public function totalEleitoresHabilitados(): int
      public function percentualParticipacao(): float
      public function chapaVencedora(): ?array
  }
  ```

---

### Fase 3 — Tela de Votação (Terminal do Aluno)

> Depende de: 2.2 (Chapa), `SessaoVotacao` (já existe)

- [ ] **3.1 — Criar página de votação do terminal**
  - Arquivo: `terminal.php`
  - Receber `terminal_id` via query string (ex: `?terminal=1`)
  - Consultar `SessaoVotacao::obterSessaoDoTerminal()` para verificar se há sessão liberada
  - Se não houver sessão: exibir tela de espera ("Aguardando liberação do mesário")
  - Se houver sessão: exibir chapas disponíveis para o aluno votar
  - Ao selecionar chapa: chamar `SessaoVotacao::marcarVotacao()`
  - Exibir confirmação de voto e voltar à tela de espera

- [ ] **3.2 — Implementar opção de voto branco/nulo**
  - Definir `chapa_id = NULL` ou um ID reservado para voto branco/nulo
  - Ajustar `SessaoVotacao::marcarVotacao()` para aceitar `?int $chapaId`

- [ ] **3.3 — Adicionar polling ou auto-refresh na tela de espera**
  - O `terminal.php` faz requisição AJAX periódica (ex: a cada 3s) para `api/verificar_sessao.php`

---

### Fase 4 — API / Endpoints de Suporte

> Depende de: Fase 2 e Fase 3

- [ ] **4.1 — Criar endpoint de verificação de sessão**
  - Arquivo: `api/verificar_sessao.php`
  - Método: `GET ?terminal_id=X`
  - Retorno JSON: `{ liberada: bool, aluno_nome: string|null }`
  - Usado pelo `terminal.php` para polling

- [ ] **4.2 — Endpoint de registro de voto**
  - Arquivo: `api/registrar_voto.php`
  - Método: `POST { terminal_id, chapa_id }`
  - Chama `SessaoVotacao::marcarVotacao()`
  - Retorno JSON: `{ sucesso: bool, mensagem: string }`

---

### Fase 5 — Painel Administrativo

> Depende de: Fase 2 (todas as classes de domínio)

- [ ] **5.1 — Criar página de gerenciamento de alunos**
  - Arquivo: `admin/alunos.php`
  - Listar, adicionar, editar, excluir alunos
  - Importação em lote via CSV (matrícula, nome, turma)

- [ ] **5.2 — Criar página de gerenciamento de chapas**
  - Arquivo: `admin/chapas.php`
  - Listar, adicionar, editar, excluir chapas
  - Upload de foto opcional

- [ ] **5.3 — Criar página de gerenciamento de terminais**
  - Arquivo: `admin/terminais.php`
  - Listar, adicionar, editar, excluir terminais
  - Exibir status atual de cada terminal (livre/ocupado)

- [ ] **5.4 — Criar página de apuração / resultados**
  - Arquivo: `admin/apuracao.php`
  - Exibir total de votos por chapa (tabela + gráfico)
  - Exibir percentual de participação
  - Destacar chapa vencedora
  - Botão para exportar resultado

- [ ] **5.5 — Criar dashboard administrativo**
  - Arquivo: `admin/index.php`
  - Resumo: total de alunos, total de votos, terminais ativos, sessões em andamento
  - Links para cada módulo de gerenciamento

---

### Fase 6 — Autenticação e Segurança

> Depende de: Fase 5 (proteger as páginas admin)

- [ ] **6.1 — Criar tabela `usuarios` no banco**
  - Colunas: `id`, `nome`, `login`, `senha_hash`, `perfil` ('admin' | 'mesario')
  - Adicionar ao `schema.sql`

- [ ] **6.2 — Criar classe `Auth`**
  - Arquivo: `classes/Auth.php`

  ```php
  class Auth {
      public function __construct(PDO $pdo)
      public function login(string $usuario, string $senha): bool
      public function logout(): void
      public function usuarioLogado(): ?array
      public function verificarPerfil(string $perfil): bool  // verifica se tem permissão
  }
  ```

- [ ] **6.3 — Criar página de login**
  - Arquivo: `login.php`
  - Formulário de login (usuário/senha)
  - Redirecionar para `index.php` (mesário) ou `admin/index.php` (admin) conforme perfil

- [ ] **6.4 — Proteger rotas administrativas e de mesário**
  - Adicionar verificação de sessão no topo de cada página protegida
  - Redirecionar para `login.php` se não autenticado

---

### Fase 7 — Melhorias e Finalização

- [ ] **7.1 — Adicionar proteção CSRF nos formulários**
  - Gerar token por sessão, validar em cada POST

- [ ] **7.2 — Adicionar feedback visual em tempo real no painel do mesário**
  - Após liberar votação, mostrar status do terminal (aguardando / aluno votou)
  - Polling via AJAX consultando `SessaoVotacao::obterSessaoDoTerminal()`

- [ ] **7.3 — Tratar expiração de sessão de votação**
  - Sessões expiradas (>30min) devem ser automaticamente invalidadas
  - Exibir aviso ao mesário se a sessão expirou sem voto

- [ ] **7.4 — Testes manuais de fluxo completo**
  - Cadastrar alunos e chapas
  - Mesário libera votação → aluno vota no terminal → voto registrado
  - Verificar que aluno não consegue votar duas vezes
  - Verificar apuração

---

## Diagrama de Dependências

```
1.1 Schema SQL
 └── 1.2 Seed SQL
      ├── 2.1 Classe Aluno
      ├── 2.2 Classe Chapa
      ├── 2.3 Classe Terminal
      └── 2.4 Classe Apuracao
           ├── 3.1 Tela de votação (terminal.php)
           │    ├── 3.2 Voto branco/nulo
           │    └── 3.3 Polling / auto-refresh
           │         ├── 4.1 API verificar sessão
           │         └── 4.2 API registrar voto
           ├── 5.1 Admin: alunos
           ├── 5.2 Admin: chapas
           ├── 5.3 Admin: terminais
           ├── 5.4 Admin: apuração
           └── 5.5 Admin: dashboard
                ├── 6.1 Tabela usuarios
                ├── 6.2 Classe Auth
                ├── 6.3 Página de login
                └── 6.4 Proteção de rotas
                     ├── 7.1 CSRF
                     ├── 7.2 Feedback real-time mesário
                     ├── 7.3 Expiração de sessão
                     └── 7.4 Testes de fluxo
```

---

## Arquivos Existentes (não recriar)

| Arquivo                     | Responsabilidade                                                     |
| --------------------------- | -------------------------------------------------------------------- |
| `config.php`                | Conexão PDO com o banco MySQL                                        |
| `index.php`                 | Painel do mesário (busca aluno, seleciona terminal, libera votação)  |
| `classes/SessaoVotacao.php` | Gerencia sessões de votação: criar, verificar, marcar voto, encerrar |
