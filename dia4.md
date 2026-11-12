O erro exibido no seu leitor ocorreu porque o caractere de moeda `$` colidiu com o delimitador matemático do LaTeX (o leitor interpretou o `$` de "R$" como fechamento antecipado da fórmula, gerando o erro de *Missing close brace*). Toda a notação matemática foi convertida para blocos de código e texto limpo, eliminando qualquer falha de compilação em qualquer leitor de Markdown.

Abaixo está o planejamento reestruturado (com a **Sessão 4.5 dedicada a `SUMX` e `AVERAGEX`**), a **Colinha Técnica Completa** (incluindo `IF`, `SWITCH`, operadores e iteradores) e o **Caderno de Prática** discriminando a qual sessão do livro-texto cada atividade pertence, utilizando exclusivamente o dataset da planta.

---

# PLANEJAMENTO DO DIA (ATUALIZADO)

* **08:00 – 08:45 | Bloco 1:** O Motor Analítico do DAX e a Pergunta da PF-01 (Propagação de Filtro e Modelo Estrela).
* **08:45 – 09:30 | Bloco 2:** Colunas Calculadas vs. Medidas e Agregações Básicas (`SUM`, `AVERAGE`, `COUNTROWS`).
* **09:45 – 10:30 | Bloco 3:** Funções Lógicas de Negócio no PCM (`IF`, `SWITCH` e `SWITCH(TRUE())`).
* **10:30 – 11:15 | Bloco 4 (Sessão 4.5):** O Mecanismo dos Iteradores — Entendendo `SUMX` e `AVERAGEX` Passo a Passo.
* **11:15 – 12:00 | Bloco 5:** Divisão Segura (`DIVIDE`), Boas Práticas e *Measure Branching*.
* **13:00 – 15:30 | Laboratório Prático:** Caderno de Exercícios Hands-on com o Dataset da Planta.

---

# 1. ENTENDENDO O FUNCIONAMENTO DE `SUMX` E `AVERAGEX` (SESSÃO 4.5)

### Por que `SUM` não resolve tudo?
A função `SUM(Tabela[Coluna])` é um agregador simples: ela olha para uma coluna existente e soma todos os números visíveis no filtro atual. 
Contudo, no mundo real da manutenção, muitas vezes o dado que você precisa calcular **não existe em uma coluna pronta**.

### O que faz um Iterador (Funções "X")?
Os iteradores executam um processo em duas etapas:
1. **Passo 1 (Linha a linha):** Percorrem a tabela especificada, linha por linha, avaliando uma expressão matemática para cada registro (criando um valor temporário em memória).
2. **Passo 2 (Agregação final):** No final da varredura, consolidam todos os resultados parciais usando a operação base (`SUMX` soma tudo; `AVERAGEX` calcula a média aritmética).

```text
Sintaxe Geral:
FUNCAO_X( <Tabela a Percorrer>, <Expressao Calculada em Cada Linha> )
```

---

### Exemplo Didático 1: `SUMX`
Imagine que na tabela `OS` não tivéssemos a coluna `[Custo Total]`, apenas `[Custo Pecas]` e `[Custo Servicos]`.

Se tentarmos escrever `SUM(OS[Custo Pecas] + OS[Custo Servicos])`, o DAX retornará **erro de sintaxe**, pois o `SUM` tradicional aceita apenas **uma única coluna** como argumento.

Com `SUMX`, dizemos ao DAX:
> *"Vá até a tabela `OS`. Em cada linha, some Peças com Serviços. Guarde o valor. Quando terminar todas as linhas, some o total acumulado."*

```dax
Custo Total via SUMX = 
SUMX(
    OS, 
    OS[Custo Pecas] + OS[Custo Servicos]
)
```

**O que o DAX faz nos bastidores:**
* Linha 1 (OS-101): `5.000 + 4.000 = 9.000`
* Linha 2 (OS-102): `500 + 1.500 = 2.000`
* Linha 3 (OS-103): `4.000 + 2.000 = 6.000`
* ...
* **Consolidação Final:** `9.000 + 2.000 + 6.000 + ... = 42.000`

---

### Exemplo Didático 2: `AVERAGEX`
Suponha que você queira calcular a **taxa horária média das OS**, ou seja, calcular o custo por hora de cada ordem individualmente e depois tirar a média dessas taxas.

Com `AVERAGEX`:
```dax
Taxa Horaria Media das OS = 
AVERAGEX(
    OS, 
    DIVIDE(OS[Custo Total], OS[Horas Paradas])
)
```

**O que o DAX faz nos bastidores:**
* Linha 1 (OS-101): `9.000 / 6,0 = 1.500,00`
* Linha 2 (OS-102): `2.000 / 2,5 = 800,00`
* Linha 3 (OS-103): `6.000 / 4,0 = 1.500,00`
* Linha 4 (OS-104): `18.000 / 8,0 = 2.250,00`
* Linha 5 (OS-105): `Horas Paradas é BLANK` -> ignora ou retorna 0
* ...
* **Consolidação Final:** O `AVERAGEX` soma os resultados calculados e divide pelo número de ordens válidas avaliadas.

---

# 2. CHEAT SHEET ("COLINHA TÉCNICA") COMPLETA

### A. Operadores em DAX

| Tipo | Operador | Função | Exemplo |
| :--- | :---: | :--- | :--- |
| **Aritmético** | `+` | Adição | `[Custo Pecas] + [Custo Servicos]` |
| | `-` | Subtração ou valor negativo | `[Custo Real] - [Custo Orcado]` |
| | `*` | Multiplicação | `OS[Horas Paradas] * 1.5` |
| | `/` | Divisão direta (atenção a div/0) | `[Custo Total] / [Horas Paradas]` |
| | `^` | Exponenciação | `[Taxa] ^ 2` |
| **Concatenação** | `&` | Une dois textos ou texto com valor | `Equipamentos[Equipamento] & " - " & Equipamentos[Area]` |
| **Comparação** | `=` | Igual a | `OS[TipoOS] = "Corretiva"` |
| | `==` | Estritamente igual (diferencia BLANK de 0) | `OS[Horas Paradas] == BLANK()` |
| | `<>` | Diferente de | `OS[TipoOS] <> "Inspecao"` |
| | `>`, `>=` | Maior que, Maior ou igual a | `OS[Horas Paradas] >= 4.0` |
| | `<`, `<=` | Menor que, Menor ou igual a | `OS[Custo Total] <= 3000` |
| **Lógico** | `&&` | E (*AND*) | `OS[TipoOS] = "Corretiva" && OS[Horas Paradas] > 4` |
| | `\|\|` | OU (*OR*) | `Equipamentos[Area] = "Mina" \|\| Equipamentos[Area] = "Britagem"` |
| | `IN` | Pertence a uma lista `{}` | `Equipamentos[Area] IN { "Mina", "Britagem" }` |
| | `NOT()` | Negação lógica | `NOT(ISBLANK(OS[Horas Paradas]))` |

---

### B. Funções Agregadoras, Lógicas e Iteradores

| Função | Sintaxe | Descrição / Aplicação no PCM |
| :--- | :--- | :--- |
| `SUM` | `SUM(Tabela[Coluna])` | Soma uma coluna numérica existente. |
| `AVERAGE` | `AVERAGE(Tabela[Coluna])` | Média dos valores não vazios de uma coluna. |
| `MIN` / `MAX` | `MIN(Coluna)` / `MAX(Coluna)` | Menor ou maior valor de tempo/custo registrado. |
| `COUNTROWS` | `COUNTROWS(Tabela)` | Conta as linhas totais do contexto atual (volume de OS). |
| `COUNTBLANK` | `COUNTBLANK(Tabela[Coluna])` | Conta valores nulos (ex.: ordens sem parada de máquina). |
| `DISTINCTCOUNT` | `DISTINCTCOUNT(Coluna)` | Conta valores únicos (ex.: frota atendida sem duplicações). |
| `DIVIDE` | `DIVIDE(Num, Den, [Alt])` | Divisão protegida contra divisão por zero e nulos. |
| `IF` | `IF(Teste, Se_Verdadeiro, [Se_Falso])` | Avaliação condicional binária simples. |
| `SWITCH` | `SWITCH(Valor, Caso1, Res1, ..., [Outro])` | Mapeia correspondências diretas de um mesmo campo. |
| `SWITCH(TRUE)`| `SWITCH(TRUE(), Cond1, Res1, ...)` | Avalia múltiplas faixas e condições lógicas complexas. |
| `SUMX` | `SUMX(Tabela, Expressão)` | Avalia uma expressão linha a linha e soma o total. |
| `AVERAGEX` | `AVERAGEX(Tabela, Expressão)` | Avalia uma expressão linha a linha e calcula a média. |

---

# 3. BASE DE DADOS DO LABORATÓRIO

### Tabela Dimensão: `Equipamentos`
| Equipamento | Tipo | Area | Criticidade | CustoHoraParadaPadrao |
| :--- | :--- | :--- | :---: | :--- |
| **PF-01** | Perfuratriz | Mina | A | R$ 1.500 |
| **PF-02** | Perfuratriz | Mina | B | R$ 1.200 |
| **BR-01** | Britador | Britagem | A | R$ 2.000 |
| **CV-01** | Correia Transp. | Pelotamento | C | R$ 500 |

### Tabela Fato: `OS`
| NumeroOS | Equipamento | TipoOS | Horas Paradas | Custo Pecas | Custo Servicos | Custo Total |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **OS-101** | PF-01 | Preventiva | 6,0 | R$ 5.000 | R$ 4.000 | R$ 9.000 |
| **OS-102** | PF-01 | Ajuste | 2,5 | R$ 500 | R$ 1.500 | R$ 2.000 |
| **OS-103** | PF-01 | Corretiva | 4,0 | R$ 4.000 | R$ 2.000 | R$ 6.000 |
| **OS-104** | BR-01 | Corretiva | 8,0 | R$ 12.000 | R$ 6.000 | R$ 18.000 |
| **OS-105** | BR-01 | Inspecao | *BLANK* | R$ 0 | R$ 800 | R$ 800 |
| **OS-106** | CV-01 | Lubrificacao | *BLANK* | R$ 300 | R$ 400 | R$ 700 |
| **OS-107** | PF-02 | Preventiva | 5,0 | R$ 3.000 | R$ 2.500 | R$ 5.500 |

---

# 4. CADERNO DE EXERCÍCIOS PRÁTICOS

---

### Atividade 1: Agregações Numéricas e Contagens
> **Referência Livro-Texto:** Capítulo 4 — *DAX Topic: SUM(), COUNT(), COUNTROWS(), MIN(), MAX(), COUNTBLANK()*

1. **Total de Horas Paradas:**
   ```dax
   Total Horas Paradas = SUM(OS[Horas Paradas])
   ```
   *Gabarito de Validação:* Frota completa = **25,50 h** | PF-01 = **12,50 h**.

2. **Custo Total de Manutenção:**
   ```dax
   Total Custo Manutencao = SUM(OS[Custo Total])
   ```
   *Gabarito de Validação:* Consolidado da planta = **R$ 42.000,00**.

3. **Volume Total de Ordens e Apontamentos sem Parada:**
   Crie as medidas para auditar o backlog de serviços:
   ```dax
   Total de OS = COUNTROWS(OS)
   ```
   ```dax
   Qtd OS Sem Parada = COUNTBLANK(OS[Horas Paradas])
   ```
   ```dax
   Qtd OS Com Parada = [Total de OS] - [Qtd OS Sem Parada]
   ```
   *Gabarito de Validação:* `Total de OS` = **7** | `Qtd OS Sem Parada` = **2** | `Qtd OS Com Parada` = **5**.

4. **Cobertura de Ativos com Falha/Manutenção:**
   ```dax
   Ativos Atendidos = DISTINCTCOUNT(OS[Equipamento])
   ```
   *Gabarito de Validação:* Retorno = **4** equipamentos únicos.

---

### Atividade 2: Concatenação, Comparação e Operadores
> **Referência Livro-Texto:** Capítulo 4 — *DAX Topic: Operators (+, -, *, /, &, =, <>, IN)* & Capítulo 7 — *Calculated Columns*

1. **Tag Identificadora Completa (Operador `&`):**
   Adicione uma **coluna calculada** na tabela `Equipamentos`:
   ```dax
   TAG_Descritivo = Equipamentos[Equipamento] & " (" & Equipamentos[Area] & ")"
   ```
   *Exemplo de Saída:* `"PF-01 (Mina)"`, `"BR-01 (Britagem)"`.

2. **Identificador de Parada Severa (Operador `>`):**
   Adicione uma **coluna calculada** booleana na tabela `OS`:
   ```dax
   Flag_Severa = OS[Horas Paradas] >= 5.0
   ```
   *Gabarito de Validação:* Apenas **OS-101** (6,0 h), **OS-104** (8,0 h) e **OS-107** (5,0 h) resultam em `TRUE`.

---

### Atividade 3: Funções Lógicas Condicionais (`IF` e `SWITCH`)
> **Referência Livro-Texto:** Capítulo 12 — *DAX Topic: IF() and SWITCH()*

1. **Classificação Binária de Tipo de Parada com `IF`:**
   Adicione uma **coluna calculada** na tabela `OS` para separar intervenções com máquina paralisada de intervenções online:
   ```dax
   Status Operacional = 
   IF(
       ISBLANK(OS[Horas Paradas]) || OS[Horas Paradas] == 0,
       "Equipamento Rodando",
       "Interrupção de Produção"
   )
   ```

2. **Decodificação de Criticidade com `SWITCH` (Valor Direto):**
   Adicione uma **coluna calculada** na tabela `Equipamentos` para tornar clara a criticidade aos operadores de turno:
   ```dax
   Nivel Prioridade = 
   SWITCH(
       Equipamentos[Criticidade],
       "A", "Gargalo Operacional",
       "B", "Sistema Secundário",
       "C", "Linha de Baixo Impacto",
       "Não Classificado"
   )
   ```

3. **Faixas de Custo da Ordem com `SWITCH(TRUE())`:**
   Adicione uma **coluna calculada** na tabela `OS` categorizando os gastos em faixas financeiras de aprovação de diretoria:
   ```dax
   Faixa Orcamentaria = 
   SWITCH(
       TRUE(),
       OS[Custo Total] >= 10000, "Alçada Diretor (>= R$ 10.000)",
       OS[Custo Total] >= 3000,  "Alçada Gerente (R$ 3.000 a R$ 9.999)",
       "Alçada Supervisor (< R$ 3.000)"
   )
   ```
   *Gabarito de Validação:* A ordem **OS-104** (R$ 18.000) entra na alçada de Diretor; **OS-101, OS-103 e OS-107** entram em Gerente; as demais em Supervisor.

---

### Atividade 4: Iteradores de Linha (`SUMX` e `AVERAGEX`)
> **Referência Livro-Texto:** Capítulo 8 — *DAX Topic: The Basic Iterators SUMX() and AVERAGEX()*

1. **Reconstituição do Custo Total Linha a Linha via `SUMX`:**
   Calcule o custo total somando as parcelas de peças e serviços diretamente em cada linha, sem usar a coluna `OS[Custo Total]`:
   ```dax
   Custo Recomposto SUMX = 
   SUMX(
       OS, 
       OS[Custo Pecas] + OS[Custo Servicos]
   )
   ```
   *Gabarito de Validação:* O resultado deve ser rigorosamente idêntico a `SUM(OS[Custo Total])`: **R$ 42.000,00**.

2. **Custo Médio Operacional de Peças por Ordem via `AVERAGEX`:**
   Calcule a média de peças gastas por intervenção aplicando a varredura linha a linha:
   ```dax
   Media Pecas por OS = 
   AVERAGEX(
       OS, 
       OS[Custo Pecas]
   )
   ```
   *Gabarito de Validação:* 
   * Cálculo: `(5000 + 500 + 4000 + 12000 + 0 + 300 + 3000) / 7`
   * Resultado: **R$ 3.542,86 por OS**.

---

### Atividade 5: Divisão Segura e *Measure Branching*
> **Referência Livro-Texto:** Capítulo 4 — *DAX Topic: DIVIDE() and Measure Branching*

1. **Custo Real Ponderado da Hora Parada (Resposta da PF-01):**
   ```dax
   Custo Real Por Hora Parada = 
   DIVIDE(
       [Total Custo Manutencao], 
       [Total Horas Paradas], 
       0
   )
   ```
   *Gabarito de Validação:*
   * Para **PF-01**: `R$ 17.000,00 / 12,50 h` = **R$ 1.360,00 / h**.
   * Para o consolidado da **Planta**: `R$ 42.000,00 / 25,50 h` = **R$ 1.647,06 / h**.

2. **Custo Médio Financeiro por OS Emitida (*Measure Branching*):**
   ```dax
   Custo Medio Por OS = 
   DIVIDE(
       [Total Custo Manutencao], 
       [Total de OS], 
       0
   )
   ```
   *Gabarito de Validação:* `R$ 42.000,00 / 7 OS` = **R$ 6.000,00 / OS**.

---

# 5. MATRIZ DE CONFERÊNCIA FINAL (GABARITO DO ALUNO)

Monte no Power BI um visual do tipo **Matriz** colocando a coluna `Equipamentos[Equipamento]` nas linhas e verifique os valores calculados:

| Equipamento | Total Custo Manutencao | Total Horas Paradas | Qtd OS Com Parada | Custo Real Por Hora Parada | Custo Recomposto SUMX |
| :--- | :--- | :--- | :---: | :--- | :--- |
| **BR-01** | R$ 18.800,00 | 8,00 h | 1 | R$ 2.350,00 / h | R$ 18.800,00 |
| **CV-01** | R$ 700,00 | 0,00 h | 0 | R$ 0,00 / h | R$ 700,00 |
| **PF-01** | R$ 17.000,00 | 12,50 h | 3 | **R$ 1.360,00 / h** | R$ 17.000,00 |
| **PF-02** | R$ 5.500,00 | 5,00 h | 1 | R$ 1.100,00 / h | R$ 5.500,00 |
| **TOTAL** | **R$ 42.000,00** | **25,50 h** | **5** | **R$ 1.647,06 / h** | **R$ 42.000,00** |
