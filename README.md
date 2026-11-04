# OptiStock — Banco de Dados

Banco PostgreSQL utilizado pelo site e pela API do OptiStock para gerenciar peças, fornecedores, categorias, movimentações de estoque e usuários.

## Arquivos

- `datapim.sql`: estrutura completa do banco, organizada em blocos comentados.
- `README.md`: documentação e instruções de instalação.

## Requisitos

- PostgreSQL 18, versão utilizada para gerar o SQL.
- pgAdmin ou cliente `psql`.
- Um banco vazio e um usuário com permissão para criar os objetos.

## Instalação pelo pgAdmin

1. Crie um banco vazio, por exemplo `datapim_novo`.
2. Selecione esse banco e abra o **Query Tool**.
3. Abra ou cole o conteúdo completo de `datapim.sql`.
4. Execute todo o script, incluindo `BEGIN` e `COMMIT`.
5. Atualize a árvore **Schemas > public > Tables** para visualizar as tabelas.

O script cria os objetos dentro do banco selecionado; ele não cria o banco nem configura usuários do PostgreSQL. Não execute no `datapim` já configurado: as tabelas existentes causarão conflitos.

Se ocorrer um erro, execute `ROLLBACK;`, corrija a causa e tente novamente em um banco vazio.

## Instalação pelo terminal

Com o banco vazio já criado, execute na pasta `database`:

```bash
psql -h localhost -p 5432 -U SEU_USUARIO -d datapim_novo -W -v ON_ERROR_STOP=1 -f datapim.sql
```

Substitua `SEU_USUARIO` e o nome do banco conforme seu ambiente. A autenticação precisa estar liberada para esse usuário e banco.

## Tabelas

| Tabela | Finalidade |
|---|---|
| `categorias` | Categorias das peças |
| `fornecedores` | Cadastro dos fornecedores |
| `pecas` | Código, preço, saldo, estoque mínimo e localização |
| `movimentacoes_estoque` | Histórico de entradas e saídas |
| `comunicados` | Comunicados internos |
| `usuarios` | Contas de acesso e dados de autenticação |
| `perfis` | Perfis de acesso |
| `usuarios_perfis` | Vínculos entre usuários e perfis |
| `permissoes_usuarios` | Permissões associadas a usuários |
| `permissoes_perfis` | Permissões associadas a perfis |
| `logins_usuarios` | Identificadores de provedores de login externo |
| `tokens_usuarios` | Tokens de autenticação gerenciados pelo Identity |
| `__EFMigrationsHistory` | Controle técnico das migrations aplicadas |

## Relacionamentos principais

- Cada peça pertence a uma categoria e pode ter um fornecedor.
- Cada movimentação referencia uma peça e o usuário responsável.
- Cada comunicado referencia seu autor.
- Usuários e perfis possuem relacionamento de muitos para muitos.

## Regras do banco

- Quantidade em estoque, estoque mínimo e preço não podem ser negativos.
- Movimentações exigem quantidade positiva e tipo `Entry` ou `Exit`.
- Os saldos anterior e posterior devem ser coerentes com a movimentação.
- O histórico de movimentações não pode ser alterado nem excluído.
- O código da peça e o nome da categoria devem ser únicos.
- A atualização de uma peça registra automaticamente a data de alteração.

A atualização do saldo é realizada pela aplicação em transação. Os triggers não atualizam o saldo automaticamente.

## Views e funções

| Objeto | Finalidade |
|---|---|
| `visao_geral_estoque` | Peças com categoria, fornecedor, situação e valor em estoque |
| `movimentacoes_diarias` | Entradas e saídas agrupadas por dia UTC |
| `fn_resumo_estoque()` | Indicadores gerais do estoque |
| `fn_totais_movimentacoes(inicio, fim)` | Totais no período, incluindo o início e excluindo o fim |
| `fn_atualizar_data_peca()` | Função do trigger de data de atualização |
| `fn_proteger_historico_movimentacoes()` | Função do trigger de proteção do histórico |

## Consultas de exemplo

```sql
SELECT * FROM public.categorias ORDER BY nome;
SELECT * FROM public.pecas ORDER BY nome;
SELECT * FROM public.visao_geral_estoque;
SELECT * FROM public.movimentacoes_diarias;
SELECT * FROM public.fn_resumo_estoque();

SELECT * FROM public.fn_totais_movimentacoes(
    '2026-11-01 00:00:00+00'::timestamptz,
    '2026-12-01 00:00:00+00'::timestamptz
);
```

## Integração com o site

O site utiliza Entity Framework Core e Npgsql. O SQL inclui as quatro migrations correspondentes à estrutura exportada, permitindo que essa versão do site reconheça o esquema instalado.

Configure `ConnectionStrings:OptiStock` para apontar ao banco instalado. No ambiente local preparado neste projeto, as configurações ficam no arquivo privado `src/OptiStock.Web/appsettings.Development.local.json`.

O script não inclui usuários, senhas ou dados de negócio. A conta administrativa e os perfis iniciais são criados pelo site, conforme a configuração `BootstrapAdmin`.
