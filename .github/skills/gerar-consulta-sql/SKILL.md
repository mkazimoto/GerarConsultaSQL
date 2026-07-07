---
name: gerar-consulta-sql
description: "Use when: gerando consultas SQL para o ERP RM da TOTVS; consultando tabelas do banco RM via MCP"
---

# Gerar Consulta SQL – ERP RM TOTVS

## Quando Usar
- Criar ou ajustar consultas T-SQL para o banco de dados do ERP RM da TOTVS
- Explorar tabelas e relacionamentos do schema RM via MCP
- Gerar relatórios a partir de módulos

---

## Procedimento

### 1. Entender o Requisito
Antes de escrever qualquer SQL, confirme:
- **O que** precisa ser retornado (campos, entidade de negócio)
- **Filtros** necessários: coligada, competência, status, período
- **Granularidade**: por funcionário, por lançamento, por documento, etc.

Se o pedido for ambíguo, pergunte o módulo ou a entidade de negócio antes de prosseguir.

---

### 2. Explorar o Schema via MCP

Use a ferramenta MCP `totvs-rm-database-mcp-server` para descobrir tabelas e colunas. Exemplos de uso:

**Inspecionar colunas de uma tabela específica**:
```
search_objects: object_type=column, schema=dbo, table=FLAN
```

**Buscar por nome de coluna**:
```
search_objects: object_type=column, pattern=CODCOLIGADA
```

> Dica: A primeira letra do nome da tabela indica o módulo:
> | Letra | Módulo |
> |-------|--------|
> | `0` | TOTVS Gestão de Custos |
> | `A` | TOTVS Automação de Ponto |
> | `B` | TOTVS Avaliação e Pesquisa |
> | `C` | TOTVS Gestão Contábil |
> | `D` | TOTVS Gestão Fiscal |
> | `E` | Ensino Básico |
> | `F` | TOTVS Gestão Financeira |
> | `G` | TOTVS Inteligência de Negócios |
> | `H` | TOTVS Aprovações e Atendimento |
> | `I` | TOTVS Gestão Patrimonial |
> | `K` | TOTVS Planejamento e Controle da Produção |
> | `L` | TOTVS Gestão Bibliotecária |
> | `M` | TOTVS Construção e Projetos |
> | `N` | TOTVS Manutenção |
> | `O` | TOTVS Saúde Hospitais e Clínicas |
> | `P` | TOTVS Folha de Pagamento |
> | `R` | TOTVS Segurança e Saúde Ocupacional |
> | `S` | TOTVS Educacional |
> | `T` | TOTVS Gestão de Estoque, Compras e Faturamento |
> | `U` | Ensino Superior |
> | `V` | TOTVS Gestão de Pessoas |
> | `W` | TOTVS Gestão de Conteúdos |
> | `X` | TOTVS Incorporação |
> | `Y` | TOTVS Controle de Acesso |

---

### 3. Construir a Consulta

Siga **obrigatoriamente** todas as regras abaixo ao gerar o SQL:

#### Regras T-SQL
- **Nunca** use `SELECT *` — liste sempre as colunas explicitamente
- **Qualifique** toda coluna com o alias da tabela: `FLAN.CODCOLIGADA`, não apenas `CODCOLIGADA`
- Use **INNER JOIN** em vez de subconsultas correlacionadas sempre que possível
- Inclua **(NOLOCK)** em todas as cláusulas FROM/JOIN de leitura
- Formate o SQL com indentação consistente (cláusulas em linhas separadas)
- Adicione **comentários** explicando cada bloco lógico da consulta

#### Regras de Negócio ERP RM
- **Sempre** filtre por `CODCOLIGADA` nas tabelas que possuem essa coluna
- Prefira filtros via JOIN em vez de subselects quando buscar em tabelas de domínio
- Use `GCOLIGADA` para obter a razão social / nome da coligada quando necessário

#### Template Base
```sql
-- ============================================================
-- Objetivo: [descreva o objetivo da consulta]
-- Módulo:   [Financeiro / RH / Contábil / ...]
-- Tabelas:  [TABELA1, TABELA2, ...]
-- ============================================================

SELECT
    T1.COLUNA1,
    T1.COLUNA2,
    T2.COLUNA3
FROM TABELA1 T1 (NOLOCK)
    INNER JOIN TABELA2 T2 (NOLOCK)
        ON T2.CODCOLIGADA = T1.CODCOLIGADA
       AND T2.FK_COLUNA   = T1.PK_COLUNA
WHERE
    T1.CODCOLIGADA = :CODCOLIGADA   -- filtro de coligada (obrigatório)
    -- adicionar filtros adicionais aqui
ORDER BY
    T1.COLUNA1
```

---

### 4. Validar com Execute SQL

Após gerar a consulta, teste-a usando a ferramenta MCP:

```
execute_sql: SELECT TOP 10 <consulta gerada>
```

- Verifique se retornou linhas esperadas
- Confirme que colunas e tipos batem com o esperado
- Se retornar erro, ajuste o SQL e repita

---

### 5. Entregar o Resultado

Entregue o SQL formatado em um bloco de código `sql`, com:
1. Cabeçalho de comentários (objetivo, módulo, tabelas)
2. Colunas explicitamente listadas
3. Comentários em blocos lógicos (WHERE, JOIN, etc.)
4. Parâmetros destacados (ex.: `:CODCOLIGADA`, `:DATAINICIO`)

---

## Exemplos de Prompts para Ativar esta Skill

- `/gerar-consulta-sql Listar lançamentos financeiros em aberto e a receber da coligada 1`
- `/gerar-consulta-sql Listar de funcionários ativos agrupados por faixa etária`
- `/gerar-consulta-sql Listar os top 10 os clientes inadimplentes de vendas de imóveis`
- `/gerar-consulta-sql Listar os top 10 professores com mais faltas no módulo de ensino superior`
- `/gerar-consulta-sql Listar os top 10 alunos com as maiores notas no módulo de ensino básico`