# CADERNO DE PRÁTICA: FUNDAMENTOS DE DAX NO PCM
**Ambiente de Laboratório | Foco: Engenharia de Manutenção, Confiabilidade e Gestão de Ativos**

---

## 1. Dicionário de Dados do Laboratório

Para todos os exercícios deste caderno, utilizaremos estritamente o modelo de dados de manutenção composto por duas tabelas relacionadas em esquema estrela:

### Tabela Dimensão: `Equipamentos` (Cadastro Mestre)
| Equipamento (TAG) | Tipo | Area | Criticidade | CustoHoraParadaPadrao |
| :--- | :--- | :--- | :---: | :--- |
| **PF-01** | Perfuratriz | Mina | A | R$ 1.500 |
| **PF-02** | Perfuratriz | Mina | B | R$ 1.200 |
| **BR-01** | Britador | Britagem | A | R$ 2.000 |
| **CV-01** | Correia Transp. | Pelotamento | C | R$ 500 |

### Tabela Fato: `OS` (Apontamentos de Oficina)
| NumeroOS | Equipamento | TipoOS | Horas Paradas | Custo Pecas | Custo Servicos | Custo Total |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **OS-101** | PF-01 | Preventiva | 6,0 | R$ 5.000 | R$ 4.000 | R$ 9.000 |
| **OS-102** | PF-01 | Ajuste | 2,5 | R$ 500 | R$ 1.500 | R$ 2.000 |
| **OS-103** | PF-01 | Corretiva | 4,0 | R$ 4.000 | R$ 2.000 | R$ 6.000 |
| **OS-104** | BR-01 | Corretiva | 8,0 | R$ 12.000 | R$ 6.000 | R$ 18.000 |
| **OS-105** | BR-01 | Inspecao | *BLANK* | R$ 0 | R$ 800 | R$ 800 |
| **OS-106** | CV-01 | Lubrificacao | *BLANK* | R$ 300 | R$ 400 | R$ 700 |
| **OS-107** | PF-02 | Preventiva | 5,0 | R$ 3.000 | R$ 2.500 | R$ 5.500 |

*Relacionamento:* `Equipamentos[Equipamento] 1 ----< N OS[Equipamento]`

---

## 2. Cheat Sheet ("Colinha") Técnica de DAX

### A. Operadores em DAX

| Tipo | Operador | Significado | Exemplo Prático |
| :--- | :---: | :--- | :--- |
| **Aritmético** | `+` | Adição | `[Custo Pecas] + [Custo Servicos]` |
| | `-` | Subtração / Negação | `[Custo Real] - [Custo Orcado]` |
| | `*` | Multiplicação | `[Horas Paradas] * [Taxa Padrao]` |
| | `/` | Divisão ordinária (atenção a div/0) | `[Custo Total] / [Horas Paradas]` |
| | `^` | Potenciação | `[Taxa de Falhas] ^ 2` |
| **Concatenação** | `&` | União de strings ou valores | `Equipamentos[Equipamento] & " - " & Equipamentos[Area]` |
| **Comparação** | `=` | Igual | `OS[TipoOS] = "Corretiva"` |
| | `==` | Estritamente igual (diferencia 0 de BLANK) | `OS[Horas Paradas] == BLANK()` |
| | `<>` | Diferente de | `OS[TipoOS] <> "Inspecao"` |
| | `>`, `>=` | Maior que, Maior ou igual a | `OS[Horas Paradas] >= 4.0` |
| | `<`, `<=` | Menor que, Menor ou igual a | `OS[Custo Total] <= 1000` |
| **Lógico** | `&&` | E (*AND* unário) | `OS[TipoOS] = "Corretiva" && OS[Horas Paradas] > 4` |
| | `\|\|` | OU (*OR* unário) | `OS[Area] = "Mina" \|\| OS[Area] = "Britagem"` |
| | `IN` | Pertence à lista de valores | `Equipamentos[Area] IN { "Mina", "Britagem" }` |
| | `NOT()` | Inverte a condição lógica | `NOT(ISBLANK(OS[Horas Paradas]))` |

---

### B. Funções Agregadoras, Contagens e Robustez

| Função | Sintaxe | Quando Usar no PCM |
| :--- | :--- | :--- |
| `SUM` | `SUM(Tabela[Coluna])` | Somar horas totais paradas, custo acumulado de peças ou serviços. |
| `AVERAGE` | `AVERAGE(Tabela[Coluna])` | Duração média bruta de ordens com apontamento. Ignora linhas em branco (*BLANK*). |
| `MIN` | `MIN(Tabela[Coluna])` | Identificar a intervenção de menor tempo ou menor custo da planta. |
| `MAX` | `MAX(Tabela[Coluna])` | Encontrar o maior evento de parada isolada (*bad actor*). |
| `COUNT` | `COUNT(Tabela[Coluna])` | Conta valores numéricos/datas em uma coluna específica. |
| `COUNTA` | `COUNTA(Tabela[Coluna])` | Conta qualquer valor não-nulo (inclusive textos). |
| `COUNTROWS` | `COUNTROWS(Tabela)` | **Padrão ouro:** Conta o número total de linhas/eventos na tabela filtrada. |
| `COUNTBLANK` | `COUNTBLANK(Tabela[Coluna])` | Conta quantas OS não geraram parada de máquina (linhas vazias). |
| `DISTINCTCOUNT` | `DISTINCTCOUNT(Tabela[Coluna])` | Conta quantos ativos diferentes sofreram quebras no período. |
| `DIVIDE` | `DIVIDE(Numerador, Denominador, [Alternativo])` | Realiza divisões seguras tratando divisão por zero ou denominadores nulos automaticamente. |

---

### C. Regras de Ouro de Boa Prática
1. **Nunca use `/` para calcular indicadores gerenciais (KPIs):** utilize sempre `DIVIDE()`. A divisão ordinária gera o erro `#ERROR` ou `Infinity` caso o ativo não tenha tido nenhuma falha no período.
2. **Para contagem de eventos, use sempre `COUNTROWS()`:** Evite `COUNT(Tabela[Coluna])`. `COUNTROWS()` é mais claro, expressa intenção e é otimizado internamente pelo VertiPaq.
3. **Padrão de Referência:**
   * Colunas: referencie sempre com a tabela: `Tabela[Coluna]`.
   * Medidas: referencie sempre isoladas: `[Nome da Medida]`.
4. ***Measure Branching* (Medidas em Cadeia):** Não repita fórmulas complexas dentro de outras fórmulas. Construa blocos atômicos e reutilize as medidas já homologadas.

---

## 3. Atividades Práticas

Execute os desafios a seguir criando as medidas no seu modelo com base estrita no dataset da planta.

---

### Atividade 1: Agregações Numéricas Básicas
*Objetivo: Mensurar os totais absolutos de tempo e custo da frota.*

1. **Total de Horas Paradas:**
   Crie uma medida para calcular a soma de todas as horas em que os equipamentos ficaram parados.
   ```dax
   Total Horas Paradas = SUM(OS[Horas Paradas])
   ```
   *Validação no modelo:* O resultado para a base inteira deve ser **25,50 h**. Para a **PF-01**, deve totalizar **12,50 h**.

2. **Custo Total de Manutenção:**
   Crie uma medida para apurar o gasto financeiro consolidado de peças e serviços aplicados.
   ```dax
   Total Custo Manutencao = SUM(OS[Custo Total])
   ```
   *Validação no modelo:* O resultado consolidado de toda a planta deve ser **R$ 42.000,00**.

3. **Maior e Menor Parada Registrada:**
   Crie as medidas para identificar os tempos extremos de indisponibilidade registrados em ordens isoladas.
   ```dax
   Maior Tempo Parada = MAX(OS[Horas Paradas])
   Menor Tempo Parada = MIN(OS[Horas Paradas])
   ```
   *Validação:* `Maior Tempo Parada` = **8,0 h** (OS-104) | `Menor Tempo Parada` = **2,5 h** (OS-102).

---

### Atividade 2: Funções de Contagem de Ativos e Eventos
*Objetivo: Controlar o volume de intervenções de oficina e a cobertura dos ativos.*

1. **Volume Total de Ordens de Serviço (Backlog/Execução):**
   Crie uma medida com `COUNTROWS` para contar a quantidade total de ordens emitidas.
   ```dax
   Total de OS = COUNTROWS(OS)
   ```
   *Validação:* Deve retornar exatamente **7**.

2. **Intervenções Operacionais Sem Parada:**
   Identifique quantas ordens foram executadas com a máquina rodando (inspeções/lubrificações) utilizando `COUNTBLANK`.
   ```dax
   Qtd OS Sem Parada = COUNTBLANK(OS[Horas Paradas])
   ```
   *Validação:* Deve retornar **2** (OS-105 e OS-106).

3. **Total de Ordens com Parada de Equipamento:**
   Subtraia os eventos sem indisponibilidade do total de ordens usando operadores aritméticos.
   ```dax
   Qtd OS Com Parada = [Total de OS] - [Qtd OS Sem Parada]
   ```
   *Validação:* Deve retornar **5**.

4. **Equipamentos Distintos com Apontamentos:**
   Descubra quantos equipamentos únicos deram entrada na oficina no período.
   ```dax
   Equipamentos Atendidos = DISTINCTCOUNT(OS[Equipamento])
   ```
   *Validação:* Deve retornar **4** ativos distintos.

---

### Atividade 3: Uso de Operadores e Colunas Calculadas
*Objetivo: Praticar operadores de concatenação, lógica e comparações nas tabelas.*

1. **Identificador Amigável de Cadastro (Concatenação `&`):**
   Adicione uma **coluna calculada** na tabela `Equipamentos` para exibir o código da TAG junto de sua respectiva área física de instalação.
   ```dax
   TAG_Area = Equipamentos[Equipamento] & " - " & Equipamentos[Area]
   ```
   *Exemplo de saída:* `"PF-01 - Mina"`, `"BR-01 - Britagem"`.

2. **Detecção de Parada Crítica (Operador de Comparação e Aritmético):**
   Adicione uma **coluna calculada** booleana na tabela `OS` identificando se o reparo foi de longa duração (mais de 4 horas de máquina parada).
   ```dax
   Parada_Critica = OS[Horas Paradas] > 4.0
   ```
   *Validação:* Deve retornar `TRUE` apenas para as ordens **OS-101** (6,0 h), **OS-104** (8,0 h) e **OS-107** (5,0 h).

3. **Custo Médio de Sobressalentes por Hora de Trabalho Interno:**
   Crie uma medida calculando a razão entre custo de peças e custo de serviços.
   ```dax
   Razao Pecas Servicos = DIVIDE(SUM(OS[Custo Pecas]), SUM(OS[Custo Servicos]), 0)
   ```
   *Validação:* $\text{Total Pecas} = \text{R\$\, 24.800}$ / $\text{Total Servicos} = \text{R\$\, 17.200} \rightarrow \mathbf{1,44}$.

---

### Atividade 4: Divisão Segura e Indicadores Estratégicos (`DIVIDE`)
*Objetivo: Resolver distorções de cálculo e garantir robustez contra divisões por zero.*

1. **A Resolução Definitiva da Pergunta da PF-01 (Custo Real Ponderado da Hora Parada):**
   Implemente a medida de engenharia que rateia corretamente o custo real acumulado pelas horas paradas, usando `DIVIDE`.
   ```dax
   Custo Real Por Hora Parada = 
   DIVIDE(
       [Total Custo Manutencao], 
       [Total Horas Paradas], 
       0
   )
   ```
   *Comprovação em matriz no Power BI:*
   * Arraste a coluna `Equipamentos[Equipamento]` para as linhas de uma Matriz.
   * Adicione `[Total Custo Manutencao]`, `[Total Horas Paradas]` e `[Custo Real Por Hora Parada]`.
   * **Resultado da PF-01:**
     $$\text{Custo Real Por Hora Parada (PF-01)} = \frac{\text{R\$\, 17.000}}{\text{12,5 h}} = \mathbf{R\$\, 1.360,00/h}$$
   * Note como a função retorna com segurança `R$ 0,00` ou vazio caso um equipamento não tenha horas paradas registradas (como ocorreria se filtrado por linhas puramente de inspeção).

2. **Duração Média Real por Intervenção com Parada:**
   Crie uma medida de tempo médio que considere apenas as ordens que de fato causaram paralisação (evitando a distorção gerada por ordens com valor nulo/zero).
   ```dax
   Tempo Medio Por Falha = 
   DIVIDE(
       [Total Horas Paradas], 
       [Qtd OS Com Parada], 
       0
   )
   ```
   *Validação:* $\frac{25,50\text{ h}}{5\text{ OS}} = \mathbf{5,10\text{ h/OS}}$.

---

### Atividade 5: Aplicação de *Measure Branching*
*Objetivo: Demonstrar como medidas simples tornam-se blocos construtores de KPIs sofisticados.*

1. **Índice de Custo por OS Executada:**
   Calcule o custo médio financeiro de cada ordem de manutenção expedida, reaproveitando as medidas atômicas anteriores sem reescrever `SUM` ou colunas de tabelas.
   ```dax
   Custo Medio Por OS = 
   DIVIDE(
       [Total Custo Manutencao], 
       [Total de OS], 
       0
   )
   ```
   *Validação:* $\frac{\text{R\$\, 42.000,00}}{7\text{ OS}} = \mathbf{R\$\, 6.000,00\text{ / OS}}$.

---

## 4. Tabela de Conferência Final para o Aluno

Ao montar uma tabela visual no Power BI contendo todos os equipamentos nas linhas, os valores apurados pelas medidas devem corresponder estritamente a este gabarito:

| Equipamento | Total Custo Manutencao | Total Horas Paradas | Qtd OS Com Parada | Custo Real Por Hora Parada |
| :--- | :--- | :--- | :---: | :--- |
| **BR-01** | R$ 18.800,00 | 8,00 h | 1 | R$ 2.350,00/h |
| **CV-01** | R$ 700,00 | 0,00 h | 0 | R$ 0,00/h |
| **PF-01** | R$ 17.000,00 | 12,50 h | 3 | **R$ 1.360,00/h** |
| **PF-02** | R$ 5.500,00 | 5,00 h | 1 | R$ 1.100,00/h |
| **TOTAL** | **R$ 42.000,00** | **25,50 h** | **5** | **R$ 1.647,06/h** |
