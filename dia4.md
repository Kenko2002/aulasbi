# PLANO DE AULA · DIA 4: FALANDO DAX

* **Curso:** Power BI e DAX para Manutenção Industrial
* **Instituição / Cliente:** CEDTEC Guarapari · Turma Samarco
* **Carga Horária:** 6h30min presenciais
  * **Manhã (08:00 – 12:00):** Teoria, Fundamentos, Demonstrações e Exercícios Conceituais
  * **Almoço (12:00 – 13:00)**
  * **Tarde (13:00 – 15:30):** Oficina Prática de Laboratório (Mão na Massa Passo a Passo)
* **Escopo:** Capítulo 4 (Fundamentos de DAX) e Seções 5.1 / 5.2 (CALCULATE e `% do Total`)
* **Pergunta do Dia:** *“Quanto custa, em reais, cada hora parada de um equipamento e por que a média simples calculada pelo visual distorce o resultado?”*

---

## 1. OBJETIVOS DE APRENDIZAGEM

Ao final do dia, o aluno será capaz de:
1. **Diferenciar** tecnicamente Medida de Coluna Calculada (armazenamento, momento de avaliação e impacto na memória).
2. **Escrever medidas DAX** utilizando funções de agregação escalar (`SUM`, `AVERAGE`, `COUNTROWS`, `DISTINCTCOUNT`).
3. **Construir divisões seguras e códigos legíveis** utilizando `DIVIDE`, manipulação de `BLANK()` e blocos de variáveis (`VAR` / `RETURN`).
4. **Distinguir Contexto de Linha de Contexto de Filtro**, aplicando a função `RELATED` para navegação entre tabelas.
5. **Calcular custos dinâmicos linha a linha** por meio de iteradores DAX (`SUMX` e `AVERAGEX`).
6. **Modificar contextos de filtro** com a função `CALCULATE` e calcular participações relativas (`% do Total`) com `REMOVEFILTERS`.

---

## 2. CRONOGRAMA GERAL DO DIA

| Horário | Bloco | Conteúdo / Foco |
| :--- | :--- | :--- |
| **08:00 – 08:45** | Teoria 1 | Abertura, Pergunta do Dia e o Modelo de Dados visto pelo DAX (4.1) |
| **08:45 – 09:30** | Teoria 2 | Colunas Calculadas vs. Medidas e Funções de Agregação (4.2, 4.3) |
| **09:30 – 09:45** | *Intervalo* | *Pausa para o Café* |
| **09:45 – 10:45** | Teoria 3 | Divisão Segura (`DIVIDE`), Funções Lógicas (`SWITCH`) e Variáveis (`VAR`) (4.4, 4.5) |
| **10:45 – 12:00** | Teoria 4 | Contexto de Linha vs. Filtro, `RELATED`, `SUMX`, `CALCULATE` e `% do Total` (4.6, 4.7, 5.1, 5.2) |
| **12:00 – 13:00** | *Almoço* | *Intervalo de Almoço* |
| **13:00 – 15:15** | Laboratório | Implementação Prática Guiada: Medidas, Colunas, Iteradores e Matrizes |
| **15:15 – 15:30** | Fechamento | Resolução da Pergunta do Dia, Checkpoint e Validação dos Modelos |

---

# PARTE 1: MANHÃ TEÓRICA E DEMONSTRATIVA (08:00 – 12:00)

### 08:00 – 08:45 | Bloco 1: O Modelo de Dados Visto pelo DAX (Seção 4.1)
* **Excel vs. DAX:**
  * No Excel, a fórmula mora numa coordenada fixa de célula e aponta para endereços (`=SOMA(D2:D40)`).
  * No DAX, a fórmula tem um nome exclusivo, reside no modelo de dados sem ocupar células e referencia **colunas completas de tabelas** (`SUM(OS[Horas Paradas])`).
* **Como os filtros atuam:**
  * O DAX não executa cálculos "soltos": quem define quais linhas entram na conta são os **filtros ativos** (linha da matriz, coluna, segmentador ou filtro de página).
  * No Esquema Estrela, o relacionamento direciona o filtro da dimensão (lado 1) para a fato (lado muitos). Ao filtrar a área *Moagem*, a tabela `Equipamentos` é filtrada primeiro, o relacionamento propaga esse filtro para a tabela `OS`, e a fórmula só soma as linhas resultantes.

---

### 08:45 – 09:30 | Bloco 2: Medida vs. Coluna Calculada & Agregações (Seções 4.2 e 4.3)
* **Quadro Comparativo de Arquitetura:**

| Critério | Coluna Calculada | Medida |
| :--- | :--- | :--- |
| **Momento de Cálculo** | Na carga/atualização dos dados | Em tempo real, sob demanda do visual |
| **Armazenamento** | Gravada no modelo (ocupa memória RAM) | Apenas a fórmula é guardada (ocupa CPU na hora) |
| **O que enxerga** | A linha atual da tabela | O conjunto de linhas filtradas pelo visual |
| **Resposta ao Filtro** | Valor fixo linha a linha (não muda com slicer) | O resultado recalcula a cada interação |
| **Uso Típico** | Chaves de ligação, faixas de agrupamento | Somas, médias, contagens, razões entre totais |

* **A Demonstração da "Pergunta do Dia" (Perfuratriz PF-01):**
  * *Dados:*
    * OS 1: Custo = R\$ 9.000 | Horas = 6,0 h | R\$/hora = 1.500
    * OS 2: Custo = R\$ 2.000 | Horas = 2,5 h | R\$/hora = 800
    * OS 3: Custo = R\$ 6.000 | Horas = 4,0 h | R\$/hora = 1.500
  * *Média simples da coluna calculada:* $(1.500 + 800 + 1.500) \div 3 = \mathbf{1.266,67 \text{ R\$/h}}$ (pesa igualmente paradas curtas e longas).
  * *Razão dos totais (Medida DAX):* $\frac{17.000 \text{ R\$}}{12,5 \text{ h}} = \mathbf{1.360,00 \text{ R\$/h}}$ (custo real ponderado por hora da máquina).
* **Agregações Escalares:**
  * `SUM`, `AVERAGE`, `MIN`, `MAX`.
  * **A Lição de Qualidade:** `COUNT(OS[Horas Paradas])` conta células com número (ignora nulos/vazios), enquanto `COUNTROWS(OS)` conta registros físicos. Na manutenção, a diferença entre as duas denuncia OS encerradas sem apontamento de horas.

---

☕ *09:30 – 09:45 | Intervalo para Café*

---

### 09:45 – 10:45 | Bloco 3: Divisão Segura, Funções Lógicas e Variáveis (Seções 4.4 e 4.5)
* **Divisão Segura com `DIVIDE`:**
  * A barra `/` gera erro `#DIV/0!` ou `Infinity` caso um equipamento tenha 0 horas paradas (ex: manutenção executada com máquina rodando).
  * `DIVIDE(Numerador, Denominador, [Alternativo])`: caso o denominador seja zero ou nulo, retorna `BLANK()` (ou o valor alternativo, se fornecido), permitindo que a linha seja automaticamente omitida em visuais sem poluir o painel.
* **Classificação sem aninhar com `SWITCH(TRUE())`:**
  * A ordem de avaliação é estritamente sequencial (de cima para baixo). O primeiro teste verdadeiro encerra a execução.
  * *Regra de ouro:* Ordene sempre da condição mais restritiva para a menos restritiva.
* **Variáveis (`VAR` / `RETURN`):**
  * Armazenam o resultado de uma expressão intermediária.
  * São avaliadas apenas uma vez, otimizando o processamento do mecanismo tabular.
  * Facilitam a leitura do código e a depuração de medidas complexas.
* **Tabela Dedicada de Medidas:** Boas práticas de governança criando uma tabela vazia (`_Medidas`) para separar o modelo dimensional dos indicadores calculados.

---

### 10:45 – 12:00 | Bloco 4: Contextos, Iteradores e Troca de Filtros (Seções 4.6, 4.7, 5.1 e 5.2)
* **Contexto de Linha vs. Contexto de Filtro:**
  * O contexto de linha percorre uma linha por vez (não filtra a tabela). `SUM(OS[Horas Paradas])` dentro de uma coluna calculada repete o total geral em todas as linhas.
  * Para buscar atributos da dimensão a partir da tabela fato: utiliza-se a função `RELATED(Equipamentos[Coluna])`.
* **Iteradores (`SUMX`, `AVERAGEX`):**
  * Funções terminadas em `X` recebem uma tabela e criam internamente um contexto de linha para avaliar uma expressão em cada linha, agregando ao final.
  * Essencial para valorização financeira de horas paradas:
    ```dax
    Custo de Parada = SUMX(OS, OS[Horas Paradas] * RELATED(Equipamentos[Valor Hora Parada]))
    ```
* **Introdução ao `CALCULATE`:**
  * Avalia expressões modificando o contexto de filtro original do visual.
  * Filtro booleano: acrescenta ou sobrescreve filtros da coluna indicada.
* **Cálculo de `% do Total`:**
  * Combinação de `DIVIDE` com `CALCULATE` e `REMOVEFILTERS` no denominador:
    ```dax
    % da Área = 
    DIVIDE(
        [Total de Horas Paradas],
        CALCULATE([Total de Horas Paradas], REMOVEFILTERS(Equipamentos[Área]))
    )
    ```
* **Resolução e Discussão dos Exercícios da Apostila (Capítulo 4, p. 106):**
  1. *Ex. 1:* Sintaxe de coluna `Tabela[Coluna]` $\rightarrow$ **Gabarito B**.
  2. *Ex. 2:* Médias e contagens com zero e vazios $\rightarrow$ **Gabarito A**.
  3. *Ex. 3:* `SUM` em coluna calculada $\rightarrow$ **Gabarito B**.
  4. *Ex. 4:* `SUMX` vs. média simples de hora $\rightarrow$ **Gabarito A** ($24.000$ e $31.500$).
  5. *Ex. 5:* Ordem de testes no `SWITCH(TRUE())` $\rightarrow$ **Gabarito C**.

---

🍽️ *12:00 – 13:00 | Intervalo de Almoço*

---

# PARTE 2: OFICINA PRÁTICA EM LABORATÓRIO (13:00 – 15:30)

> **Cenário de Negócio:** A coordenação de PCM precisa de um painel analítico para a reunião mensal de indicadores. A equipe deve construir, em ambiente de laboratório, as métricas oficiais de ordens de serviço, custos e criticidade operacional.

---

### ETAPA 1: CRIAÇÃO DA TABELA DEDICADA DE MEDIDAS (13:00 – 13:15)
1. No menu superior do **Power BI Desktop**, acesse a guia **Página Inicial**.
2. Clique em **Inserir Dados**.
3. Na janela exibida:
   * Altere o nome da tabela de *Tabela1* para: `_Medidas`
   * Deixe a coluna vazia e clique em **Carregar**.
4. No painel de dados, certifique-se de que a tabela `_Medidas` está disponível.

---

### ETAPA 2: AGREGAÇÕES ESCALARES BÁSICAS (13:15 – 13:35)

Crie as medidas abaixo dentro da tabela `_Medidas` (clique com o botão direito na tabela $\rightarrow$ **Nova Medida**):

#### 2.1. Total de Horas Paradas
```dax
Total de Horas Paradas = SUM(OS[Horas Paradas])
```
* *Formatação:* Guia Ferramentas de Medida $\rightarrow$ Número Decimal, 2 casas.

#### 2.2. Custo Total de Manutenção
```dax
Custo Total = SUM(OS[Custo (R$)])
```
* *Formatação:* Formato Moeda (`R$`), 2 casas decimais.

#### 2.3. Quantidade Total de Ordens de Serviço
```dax
Quantidade de OS = COUNTROWS(OS)
```
* *Formatação:* Número Inteiro.

#### 2.4. Quantidade de OS com Horas Apontadas (Indicador de Qualidade)
```dax
OS com Horas Apontadas = COUNT(OS[Horas Paradas])
```
* *Formatação:* Número Inteiro.

> **Teste Visual:** Adicione 4 cartões no relatório com cada uma dessas medidas. O aluno deverá observar se `Quantidade de OS` diverge de `OS com Horas Apontadas`, evidenciando registros incompletos.

---

### ETAPA 3: DIVISÃO SEGURA E MEDIDAS COM VARIÁVEIS (13:35 – 14:00)

#### 3.1. Custo por Hora Parada (A Resposta da Pergunta do Dia)
```dax
Custo por Hora Parada = 
DIVIDE(
    [Custo Total],
    [Total de Horas Paradas]
)
```
* *Formatação:* Formato Moeda (`R$`), 2 casas decimais.

#### 3.2. Horas Médias por OS (utilizando `VAR` / `RETURN`)
```dax
Horas Médias por OS = 
VAR HorasTotais = [Total de Horas Paradas]
VAR QtdOS = [Quantidade de OS]
RETURN
    DIVIDE(HorasTotais, QtdOS)
```
* *Formatação:* Número Decimal, 2 casas.

---

### ETAPA 4: COLUNA CALCULADA COM REGRAS DE CLASSIFICAÇÃO (14:00 – 14:20)

Crie uma **Coluna Calculada** na tabela de fatos `OS` para permitir segmentar as ordens por porte financeiro.

1. Selecione a tabela `OS` no painel Dados $\rightarrow$ clique em **Nova Coluna**.
2. Digite a expressão utilizando `SWITCH(TRUE())`:
```dax
Faixa de Custo OS = 
SWITCH(
    TRUE(),
    OS[Custo (R$)] <= 2000, "1. Baixo Custo (Até 2k)",
    OS[Custo (R$)] <= 8000, "2. Médio Custo (2k a 8k)",
    "3. Alto Custo (> 8k)"
)
```
3. **Validação visual:** Crie um gráfico de barras horizontais com `OS[Faixa de Custo OS]` no **Eixo Y** e `[Quantidade de OS]` no **Eixo X**.

---

### ETAPA 5: ITERADORES E NAVEGAÇÃO ENTRE TABELAS (14:20 – 14:45)

O custo da máquina parada não se limita à peça e à mão de obra; existe o prejuízo financeiro da indisponibilidade da planta. O cadastro de equipamentos (`Equipamentos`) contém a taxa `Valor Hora Parada`.

#### 5.1. Custo Real de Parada da Planta com `SUMX` e `RELATED`
Crie a medida na tabela `_Medidas`:
```dax
Custo Total de Indisponibilidade = 
SUMX(
    OS,
    OS[Horas Paradas] * RELATED(Equipamentos[Valor Hora Parada])
)
```
* *Formatação:* Moeda (`R$`).

#### 5.2. Demonstração de Erro Estatístico (O Atalho Perigoso)
Crie uma medida temporária de teste:
```dax
Custo Indisponibilidade Incorreto = 
[Total de Horas Paradas] * AVERAGE(Equipamentos[Valor Hora Parada])
```
* **Conferência em Matriz:**
  * Coloque `Equipamentos[Área]` nas linhas da matriz.
  * Adicione `[Custo Total de Indisponibilidade]` e `[Custo Indisponibilidade Incorreto]`.
  * *Ponto de Discussão:* Observe a linha de total. Como áreas com equipamentos de taxas horárias distintas possuem volumes de paradas diferentes, o produto dos agregados comete um erro de ponderação grave. Exclua a medida de teste após a validação.

---

### ETAPA 6: CALCULATE E PERCENTUAL DO TOTAL (14:45 – 15:15)

#### 6.1. Horas Paradas por Falhas Corretivas
```dax
Horas Corretivas = 
CALCULATE(
    [Total de Horas Paradas],
    OS[Tipo de Manutenção] = "Corretiva"
)
```
* *Formatação:* Número Decimal, 2 casas.

#### 6.2. Proporção de Corretivas sobre o Total do Equipamento
```dax
% Horas Corretivas = 
DIVIDE(
    [Horas Corretivas],
    [Total de Horas Paradas]
)
```
* *Formatação:* Porcentagem (`%`), 1 casa decimal.

#### 6.3. Participação da Área no Total Geral (`REMOVEFILTERS`)
```dax
% Horas da Área = 
DIVIDE(
    [Total de Horas Paradas],
    CALCULATE(
        [Total de Horas Paradas],
        REMOVEFILTERS(Equipamentos[Área])
    )
)
```
* *Formatação:* Porcentagem (`%`), 1 casa decimal.

---

### ETAPA 7: MATRIZ DE FECHAMENTO E CHECKPOINT DO DIA 4 (15:15 – 15:30)

Monte a visualização consolidada do dia para apresentação gerencial:

1. Insira um visual de **Matriz** na tela.
2. Configure os campos:
   * **Linhas:**
     * `Equipamentos[Área]`
     * `Equipamentos[Equipamento]` (nível hierárquico inferior)
   * **Valores:**
     * `[Quantidade de OS]`
     * `[Total de Horas Paradas]`
     * `[Horas Corretivas]`
     * `[% Horas Corretivas]`
     * `[Custo Total]`
     * `[Custo por Hora Parada]`
     * `[Custo Total de Indisponibilidade]`
     * `[% Horas da Área]`
3. **Formatação Condicional:**
   * Clique com o botão direito em `[% Horas Corretivas]` no campo Valores $\rightarrow$ **Formatação condicional** $\rightarrow$ **Barras de dados**. Marque a cor vermelha para destacar áreas críticas.
4. **Resolução Final da Pergunta do Dia:**
   * Ative o Drill-Down na matriz ou use um segmentador de equipamento.
   * Selecione um equipamento específico e mostre como o `[Custo por Hora Parada]` reflete precisamente a divisão ponderada do custo real pelas horas faturadas daquela máquina, mantendo a consistência tanto no nível de componente quanto na raiz da área.

---

## 3. CHECKPOINT DO INSTRUTOR (LISTA DE VERIFICAÇÃO)

Antes de encerrar o laboratório, garanta que cada aluno tenha em seu arquivo `.pbix`:
- [ ] Tabela `_Medidas` isolada e organizada com o ícone de calculadora.
- [ ] Medida de contagem criada com `COUNTROWS(OS)`.
- [ ] Medida de custo horário implementada exclusivamente com `DIVIDE`.
- [ ] Coluna calculada `Faixa de Custo OS` testada e validada.
- [ ] Iterador `SUMX` multiplicando horas pelo valor retornado por `RELATED`.
- [ ] Modificador de filtro `REMOVEFILTERS` operando corretamente o `% Horas da Área` fechando a linha de subtotal em $100\%$.