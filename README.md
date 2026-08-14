# Data Warehouse — Impacto da IA nos Estudantes

Modelagem dimensional de um dataset sobre como o uso de IA generativa afeta o desempenho de estudantes: notas antes e depois do semestre, horas de uso de IA, horas de estudo tradicional, retenção de habilidade e risco de burnout.

> **Status:** o modelo dimensional está desenhado e documentado. Os scripts SQL e o dashboard ainda não foram implementados — os arquivos em `sql/` e `powerbi/` são placeholders.

## O modelo

Grão da fato: **um registro por aluno** (`Student_ID`). O dataset traz uma linha única por estudante, não uma série temporal — então o grão não é por semana nem por semestre.

```
fato_desempenho_aluno
├── dim_aluno                    Student_ID, Year_of_Study
├── dim_curso                    Major_Category
├── dim_politica_institucional   Institutional_Policy
└── dim_uso_ia                   junk dimension (ver abaixo)
```

Medidas na fato: `Pre_Semester_GPA`, `Post_Semester_GPA`, `Weekly_GenAI_Hours`, `Traditional_Study_Hours`, `Tool_Diversity`, `Perceived_AI_Dependency`, `Anxiety_Level_During_Exams`, `Skill_Retention_Score`.

### Por que `dim_uso_ia` é uma junk dimension

Os atributos que descrevem o comportamento de uso de IA — `Primary_Use_Case`, `Prompt_Engineering_Skill`, `Paid_Subscription`, `Burnout_Risk_Level` — são todos categóricos e de baixa cardinalidade. Criar uma dimensão separada para cada um produziria quatro tabelas minúsculas ligadas à mesma fato. Agrupá-los em uma dimensão de junção mantém o esquema legível e permite fatiar as métricas por qualquer combinação desses atributos.

### Critério de classificação

- **Fato** — coluna numérica/contínua que mede o evento e é agregável (AVG, MIN, MAX).
- **Dimensão** — coluna categórica ou de baixa cardinalidade que descreve o contexto e serve para filtrar/agrupar.

O caso limítrofe é `Burnout_Risk_Level`: é um resultado do estudo, mas é um rótulo (Low/Medium/High), não um contínuo. Modelado como dimensão porque a pergunta útil é "GPA médio por nível de burnout" — o que exige que ele seja filtro, não medida.

A justificativa coluna a coluna, incluindo as decisões ainda em aberto, está em **[`docs/arquitetura.md`](docs/arquitetura.md)**.

## Estrutura

```
sql/
├── 001_staging.sql      carga bruta          (a implementar)
├── 002_clean.sql        limpeza/tipagem      (a implementar)
├── 003_dimensions.sql   dimensões           (a implementar)
└── 004_fact.sql         tabela fato          (a implementar)
docs/arquitetura.md      modelo dimensional e justificativas
powerbi/dashboard.pbix   dashboard            (a implementar)
```
