# Modelo dimensional — Impacto da IA nos Estudantes

## Critério de classificação

- **Fato**: coluna numérica/contínua que representa uma medida do evento (pode ser agregada com AVG, MIN, MAX, etc.).
- **Dimensão**: coluna categórica ou de baixa cardinalidade que descreve o contexto e serve para filtrar/agrupar o fato.

Grão da tabela fato: **um registro por aluno** (`Student_ID`), já que o dataset traz uma linha única por estudante (não é uma série temporal por semana/mês).

---

## Tabela fato: `fato_desempenho_aluno`

| Coluna | Tipo conceitual | Justificativa |
|---|---|---|
| `Student_ID` (FK) | chave estrangeira | liga ao aluno em `dim_aluno` |
| `Major_Category` (FK) | chave estrangeira | liga ao curso em `dim_curso` |
| `Institutional_Policy` (FK) | chave estrangeira | liga à política em `dim_politica_institucional` |
| `Uso_IA_ID` (FK) | chave estrangeira | liga ao perfil de uso em `dim_uso_ia` |
| `Pre_Semester_GPA` | numérico | medida contínua, comparável antes/depois |
| `Post_Semester_GPA` | numérico | medida contínua, resultado do semestre |
| `Weekly_GenAI_Hours` | numérico | medida de intensidade de uso |
| `Traditional_Study_Hours` | numérico | medida de intensidade de estudo tradicional |
| `Tool_Diversity` | numérico (contagem) | quantidade de ferramentas usadas — é agregável (AVG por curso, por política etc.), por isso vira medida e não atributo descritivo |
| `Perceived_AI_Dependency` | numérico (escala 1–5) | é um score medido, não uma categoria nomeada — entra como medida, permitindo AVG de dependência percebida por grupo |
| `Anxiety_Level_During_Exams` | numérico (escala) | mesma lógica — score medido, não rótulo |
| `Skill_Retention_Score` | numérico | medida de resultado, direto de fato |

---

## Dimensões

### `dim_aluno`
| Coluna | Tipo | Observação |
|---|---|---|
| `Student_ID` | PK | identificador único do aluno |
| `Year_of_Study` | categórico | Freshman/Sophomore/Junior/Senior — atributo descritivo do aluno, não do evento; como o dataset tem uma linha por aluno, não justifica dimensão própria |

### `dim_curso`
| Coluna | Tipo | Observação |
|---|---|---|
| `id_curso` | PK | chave técnica |
| `Major_Category` | categórico | baixa cardinalidade, reutilizável em filtros |

### `dim_politica_institucional`
| Coluna | Tipo | Observação |
|---|---|---|
| `id_politica` | PK | chave técnica |
| `Institutional_Policy` | categórico | ex.: Allowed_With_Citation, Strict_Ban |

### `dim_uso_ia`
| Coluna | Tipo | Observação |
|---|---|---|
| `id_uso_ia` | PK | chave técnica |
| `Primary_Use_Case` | categórico | ex.: Copywriting/Drafting, Ideation |
| `Prompt_Engineering_Skill` | categórico | Beginner/Intermediate/Advanced |
| `Paid_Subscription` | booleano | atributo descritivo do perfil de uso |
| `Burnout_Risk_Level` | categórico (Low/Medium/High) | é um rótulo, não um número contínuo — mesmo sendo um "resultado", funciona melhor como dimensão porque permite fatiar as métricas do fato por nível de risco (ex.: GPA médio por nível de burnout) |

> `dim_uso_ia` é uma **dimensão de junção** (junk dimension): agrupa vários atributos categóricos de baixa cardinalidade que descrevem o comportamento de uso de IA, evitando criar uma dimensão minúscula pra cada coluna.

---

## Pontos que exigem decisão da dupla

1. **`Tool_Diversity` e `Perceived_AI_Dependency`**: tratei como medida (fato) porque são numéricos e agregáveis — mas se vocês quiserem usá-los mais como filtro categórico (ex.: "baixa/média/alta dependência"), dá pra transformar em faixas e mover pra uma dimensão. Decisão de modelagem, não tem certo/errado absoluto.
2. **`Burnout_Risk_Level`**: modelei como dimensão por ser categórico, mesmo sendo um resultado do estudo. Se preferirem tratá-lo como o "alvo" de uma análise de correlação, também pode ficar no fato como atributo qualitativo — mas aí perde a vantagem de ser usado como filtro fácil no Power BI.
3. **`Year_of_Study`**: deixei como atributo de `dim_aluno` em vez de dimensão própria, já que não existe outro fato que reutilize essa coluna fora do contexto do aluno.
