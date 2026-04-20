# Mesário DigiTech

Sistema de votação escolar digital com painel de mesário, terminais de votação e apuração de resultados.

## Requisitos

- [XAMPP](https://www.apachefriends.org/) (Apache + MySQL + PHP 8.x)
- Navegador moderno (Chrome, Firefox, Edge)

## Instalação

1. **Clone o repositório** na pasta `htdocs` do XAMPP:

   ```bash
   cd C:\xampp\htdocs
   git clone <url-do-repositorio> mesario-digitech
   ```

2. **Inicie o XAMPP** e ative os serviços **Apache** e **MySQL**.

3. **Crie o banco de dados** acessando o phpMyAdmin (`http://localhost/phpmyadmin`) e execute o script:

   ```
   database/schema.sql
   ```

4. **(Opcional) Popule com dados de teste:**

   ```
   database/seed.sql
   ```

5. **Acesse o sistema:**

   | Página                | URL                                                         |
   | --------------------- | ----------------------------------------------------------- |
   | Painel do Mesário     | `http://localhost/mesario-digitech/`                        |
   | Terminal de Votação   | `http://localhost/mesario-digitech/terminal.php?terminal=1` |
   | Painel Administrativo | `http://localhost/mesario-digitech/admin/`                  |

## Configuração

O arquivo `config.php` contém as credenciais de conexão com o banco. As configurações padrão funcionam com o XAMPP sem alterações:

```php
DB_HOST = 'localhost'
DB_USER = 'root'
DB_PASS = ''
DB_NAME = 'votacao_digitech'
```

## Estrutura do Projeto

```
mesario-digitech/
├── config.php                 # Conexão com o banco de dados
├── index.php                  # Painel do mesário
├── terminal.php               # Tela de votação do aluno
├── login.php                  # Página de login
├── classes/
│   ├── SessaoVotacao.php      # Gerenciamento de sessões de votação
│   ├── Aluno.php              # CRUD de alunos
│   ├── Chapa.php              # CRUD de chapas
│   ├── Terminal.php           # CRUD de terminais
│   ├── Apuracao.php           # Contagem e resultados
│   └── Auth.php               # Autenticação (login, logout, perfis)
├── admin/
│   ├── index.php              # Dashboard administrativo
│   ├── alunos.php             # Gerenciamento de alunos
│   ├── chapas.php             # Gerenciamento de chapas
│   ├── terminais.php          # Gerenciamento de terminais
│   └── apuracao.php           # Apuração de resultados
├── api/
│   ├── verificar_sessao.php   # Polling de sessão do terminal
│   └── registrar_voto.php     # Registro de voto via AJAX
└── database/
    ├── schema.sql             # Criação do banco e tabelas
    └── seed.sql               # Dados iniciais para teste
```

## Modelo de Dados

| Tabela            | Descrição                                             |
| ----------------- | ----------------------------------------------------- |
| `alunos`          | Cadastro de alunos (matrícula, nome, turma, tipo, senha_hash) |
| `chapas`          | Chapas/candidatos disponíveis para votação            |
| `terminais`       | Terminais de votação disponíveis                      |
| `sessoes_votacao` | Controle de sessões de votação por terminal           |
| `votos`           | Registro de votos (com constraint contra duplicidade) |
| `eleicoes`        | Configuração de período de votação                    |
| `audit_log`       | Log de auditoria de ações do sistema                  |
| `rate_limit`      | Controle de tentativas por IP                         |

> **Banco compartilhado:** ambos os módulos (mesario-digitech e votacao-digitech) utilizam o mesmo banco `votacao_digitech`.

## Fluxo de Uso

1. **Admin** cadastra alunos, chapas e terminais pelo painel administrativo.
2. **Mesário** busca o aluno pela matrícula, seleciona o terminal e libera a votação.
3. **Aluno** acessa o terminal, visualiza as chapas e registra seu voto.
4. **Admin** acompanha a apuração em tempo real pelo painel de resultados.

## Fluxo de Git (Git Flow)

### Branches principais

| Branch    | Propósito                                                            |
| --------- | -------------------------------------------------------------------- |
| `main`    | Código estável e em produção. Nunca commitar direto aqui.            |
| `develop` | Branch de integração. Todas as features são mergeadas aqui primeiro. |

### Configuração inicial

```bash
# Clonar o repositório
git clone <url-do-repositorio> mesario-digitech
cd mesario-digitech

# Criar a branch develop a partir da main
git checkout -b develop
git push -u origin develop
```

### Padrão de criação de branches

Sempre crie branches a partir da `develop`:

```bash
git checkout develop
git pull origin develop
git checkout -b <tipo>/<descricao-curta>
```

**Tipos de branch:**

| Prefixo     | Uso                          | Exemplo                   |
| ----------- | ---------------------------- | ------------------------- |
| `feature/`  | Nova funcionalidade          | `feature/crud-alunos`     |
| `fix/`      | Correção de bug              | `fix/validacao-matricula` |
| `hotfix/`   | Correção urgente em produção | `hotfix/erro-login`       |
| `refactor/` | Refatoração de código        | `refactor/classe-sessao`  |

### Padrão de commits

Usar o formato **Conventional Commits**:

```
<tipo>: <descrição curta>
```

**Tipos de commit:**

| Tipo       | Quando usar                            |
| ---------- | -------------------------------------- |
| `feat`     | Nova funcionalidade                    |
| `fix`      | Correção de bug                        |
| `refactor` | Refatoração sem mudar comportamento    |
| `style`    | Formatação, CSS, sem mudança de lógica |
| `docs`     | Alteração em documentação              |
| `chore`    | Tarefas de manutenção (configs, deps)  |

**Exemplos:**

```bash
git commit -m "feat: criar classe Aluno com CRUD completo"
git commit -m "fix: corrigir validação de matrícula duplicada"
git commit -m "style: ajustar layout da tela de votação"
git commit -m "docs: adicionar instruções de instalação no README"
```

### Fluxo de trabalho completo

**1. Criar a feature branch e trabalhar nela:**

```bash
git checkout develop
git pull origin develop
git checkout -b feature/crud-alunos

# Trabalhar, fazer commits...
git add .
git commit -m "feat: criar classe Aluno com métodos de CRUD"
git add .
git commit -m "feat: criar página admin de gerenciamento de alunos"
```

**2. Subir a branch para o repositório remoto:**

```bash
git push -u origin feature/crud-alunos
```

**3. Criar um Pull Request (PR):**

- Acesse o repositório no GitHub/GitLab.
- Clique em **"Pull Requests"** → **"New Pull Request"**.
- Base: `develop` ← Compare: `feature/crud-alunos`.
- Adicione um título descritivo e uma descrição do que foi feito.
- Solicite revisão de um colega (se aplicável).
- Após aprovação, faça o **merge**.

**4. Atualizar sua develop local após o merge:**

```bash
git checkout develop
git pull origin develop
```

**5. Deletar a branch da feature (opcional):**

```bash
git branch -d feature/crud-alunos
git push origin --delete feature/crud-alunos
```

### Merge da develop para a main (deploy)

Quando a `develop` estiver estável e testada:

```bash
# Via Pull Request (recomendado):
# Base: main ← Compare: develop
# Criar PR no GitHub e fazer merge após revisão

# Ou via terminal:
git checkout main
git pull origin main
git merge develop
git push origin main
git checkout develop
```

### Resumo visual

```
main ─────────────────────●──────────── (produção)
                         ↑ merge
develop ──●───●───●──────●───●──────── (integração)
          ↑       ↑
feature/  ●───●   ●───●───●
crud-     (commits)
alunos
```

## Licença

Este projeto é de uso educacional.
