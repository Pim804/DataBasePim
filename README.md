# OptiStock — Banco de Dados

Banco PostgreSQL utilizado pelo site e pela API do OptiStock para gerenciar peças, fornecedores, categorias, movimentações de estoque e usuários.

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
