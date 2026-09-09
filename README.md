# Mesário DigiTech — REPOSITÓRIO ARQUIVADO

> **Este repositório foi arquivado em 09/09/2026.** O desenvolvimento continua em um único repositório:
>
> ## ➜ https://github.com/prof-ronildo-unicatolica/votacao-digitech

## Por que foi arquivado

O "mesário" não é um sistema separado: é um **perfil de usuário** e um **conjunto de telas** do sistema de votação. Manter dois repositórios com o mesmo banco de dados gerava duplicação de documentação, migrações conflitantes e deploy duplo. A justificativa completa está em [`docs/DECISOES.md`](https://github.com/prof-ronildo-unicatolica/votacao-digitech/blob/develop/docs/DECISOES.md) (decisão D1) do repositório principal.

## Onde está cada coisa agora

| O que este repositório planejava              | Onde vive no `votacao-digitech`                                  |
| --------------------------------------------- | ---------------------------------------------------------------- |
| Painel do mesário (`index.php`)               | Rota `/mesario` · histórias **H6** e **H9** em `docs/HISTORIAS_USUARIO.md` |
| Terminal de votação (`terminal.php`)          | Rota `/urna/{terminal}` · histórias **H7** e **H8**              |
| Painel administrativo (`admin/`)              | Rota `/admin` · histórias **H1** a **H5**, **H10**, **H11**      |
| Classes `Aluno`, `Chapa`, `Terminal`, `Apuracao` | Models Eloquent em `app/Models/` e `app/Support/Apuracao.php` |
| `database/schema.sql` e `seed.sql`            | `database/migrations/` e `database/seeders/`                     |
| `api/verificar_sessao.php`                    | `GET /api/terminais/{id}/status` (já funcional)                  |
| Classe `Auth`, CSRF, validação                | Recursos nativos do Laravel (ver `docs/ROADMAP.md`)              |
| Fluxo de Git (deste README)                   | `docs/GIT_FLOW.md`                                               |
| Plano de implementação                        | `docs/ROADMAP.md` + `docs/HISTORIAS_USUARIO.md`                  |

## Para a equipe

1. Clone o repositório principal e siga `docs/INSTALACAO_XAMPP.md` ou `docs/INSTALACAO_LAMP.md`.
2. Os cartões do Trello vêm de `docs/HISTORIAS_USUARIO.md`.
3. Não abra issues nem PRs aqui: o repositório é somente leitura.

O histórico de commits deste repositório permanece disponível para consulta.
