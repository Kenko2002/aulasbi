O planejamento anterior apresentava duas desconexões importantes que foram corrigidas nesta versão:
1. **Numeração das seções do livro:** As referências antigas citavam capítulos inexistentes (Capítulos 7, 8 e 12 em inglês). O seu livro-texto (*Power BI e DAX para Manutenção*) organiza todos esses temas exatamente no **Capítulo 4 (Seções 4.1 a 4.7)**. Todas as atividades e blocos agora trazem a seção exata da apostila.
2. **Alinhamento com o Dataset Real anexado:** A base enviada possui **350 ordens de serviço** (`51020` a `51869`) e **20 equipamentos** (`BR-01`, `MO-01`, `FO-01`, `TR-02`, etc.), com as colunas exatas `Custo (R$)`, `Horas Paradas`, `Horas Previstas` e `Valor Hora Parada (R$)`. Todo o laboratório prático e o gabarito de conferência da tarde foram recalculados com os números reais do seu arquivo para que os alunos vejam na tela exatamente os mesmos valores da sua apostila de aula.

---

# CRONOGRAMA DE AULA — CAPÍTULO 4: FUNDAMENTOS DA LINGUAGEM DAX

## TURNO DA MANHÃ (08:00 às 12:00) — Teoria Aplicada e Demonstrações Guiadas

| Horário | Bloco / Tópico de Aula | Seção do Livro | Atividade em Sala (Exemplos do Livro + Dataset Real) |
| :--- | :--- | :---: | :--- |
| **08:00 – 08:40** | **Bloco 1:** Modelo de Dados Visto pelo DAX | **Seção 4.1** (p. 87–90) | Conferência da Exibição de Modelo: tabelas dimensão (`Equipamentos` e `Calendário`) filtrando a tabela fato (`OS`) no sentido `1:*`. Demonstração prática filtrando a área **Concentração** (`SM-01` e `SM-02`) na segmentação. |
| **08:40 – 09:20** | **Bloco 2:** Colunas Calculadas vs. Medidas | **Seção 4.2** (p. 90–92) | O dilema da Perfuratriz `PF-01` do livro (média das médias vs. razão das somas). Criação da tabela vazia `Medidas` (via *Inserir Dados*) para organizar o painel Dados. |
| **09:20 – 10:00** | **Bloco 3:** Sintaxe, Operadores e Funções de Agregação | **Seção 4.3** (p. 92–95) | Criação das 7 medidas básicas na base real (`SUM`, `AVERAGE`, `MIN`, `MAX`, `COUNT`, `COUNTROWS`, `DISTINCTCOUNT`). Explicação prática do efeito do `0` nas OS Abertas/Canceladas sobre a função `AVERAGE`. |
| **10:00 – 10:15** | *Intervalo (Coffee Break)* | — | — |
| **10:15 – 10:55** | **Bloco 4:** Divisão Segura e Funções Lógicas | **Seção 4.4** (p. 95–98) | Proteção contra divisão por zero com `DIVIDE` e entendimento do `BLANK`. Classificação de linhas com `IF` (`Parada Longa >= 8h`) e faixas de custo com `SWITCH(TRUE(), ...)` respeitando a ordem restritiva dos testes. |
| **10:55 – 11:25** | **Bloco 5:** Variáveis (`VAR` / `RETURN`) e Formatação | **Seção 4.5** (p. 98–100) | Refatoração de medidas usando `VAR` e `RETURN`. Uso de quebras de linha (`Shift+Enter`), comentários (`--`) e técnica de depuração trocando o retorno pela variável. |
| **11:25 – 12:00** | **Bloco 6:** Contexto de Linha, Contexto de Filtro e Iteradores (`SUMX`/`AVERAGEX`) | **Seções 4.6 e 4.7** (p. 100–105) | Por que `SUM` dentro de uma coluna calculada repete `1.825,21 h` em todas as 350 linhas? Uso do `RELATED` para buscar dados do cadastro e uso do `SUMX` e `AVERAGEX` para calcular o custo de indisponibilidade linha a linha. |

---

## TURNO DA TARDE (13:00 às 15:30) — Laboratório Prático (Hands-on no Dataset Anexo)

| Horário | Etapa do Laboratório | Seções do Livro | Entrega / Atividade do Aluno |
| :--- | :--- | :---: | :--- |
| **13:00 – 13:30** | **Lab 1:** Preparação do Modelo e Medidas de Agregação | **Seções 4.1, 4.2 e 4.3** | Relacionar o modelo, criar a tabela `Medidas` e construir os cartões de `Total de Horas Paradas`, `Custo Total`, `Quantidade de OS` e `Equipamentos com OS`. |
| **13:30 – 14:00** | **Lab 2:** Divisão Segura e Colunas Lógicas | **Seção 4.4** | Criar a medida `Custo por Hora Parada` com `DIVIDE` e construir as colunas calculadas `Parada Longa` (`IF`) e `Faixa de Custo` (`SWITCH(TRUE())`). |
| **14:00 – 14:15** | *Intervalo Rápido* | — | — |
| **14:15 – 15:00** | **Lab 3:** Variáveis, `RELATED` e Iteradores (`SUMX` e `AVERAGEX`) | **Seções 4.5, 4.6 e 4.7** | Escrever medidas estruturadas com `VAR`/`RETURN`, buscar atributos com `RELATED` e calcular o **Custo de Parada (Lucro Cessante)** usando `SUMX` e `AVERAGEX`. |
| **15:00 – 15:30** | **Lab 4:** Montagem da Matriz de Conferência e Fechamento | **Seções 4.1 a 4.7** | Montar a Matriz Final por Área e Equipamento, validar os números com o gabarito oficial no projetor e resolver os 5 exercícios da página 106 do livro. |

---

# GUIA DO PROFESSOR: EXPLICAÇÃO DE `SUMX` E `AVERAGEX` (SEÇÃO 4.7)

> **Referência no Livro-Texto:** Seção 4.7 — *Iteradores SUMX e AVERAGEX* (páginas 103 a 105) e Seção 4.6 — *Contexto de Linha e Contexto de Filtro* (páginas 100 a 102).

### 1. O Problema Real na Base da Planta
Na nossa base de dados, temos duas informações separadas:
* Na tabela fato `OS`, temos a coluna `OS[Horas Paradas]`.
* Na tabela dimensão `Equipamentos`, temos a coluna `Equipamentos[Valor Hora Parada (R$)]` (por exemplo: parar o Forno `FO-01` custa **R$ 2.400/h** para a operação; parar o Hidrociclone `HC-01` custa **R$ 380/h**).

Se o gerente perguntar: *"Qual foi o prejuízo operacional (custo de parada) causado pela indisponibilidade das máquinas?"*, **não podemos** simplesmente somar todas as horas paradas (`1.825,21 h`) e multiplicar pela média simples do valor hora do cadastro (`R$ 1.199,50`), pois isso daria **R$ 2.189.339,39** (um número errado, exatamente como alerta a página 103 do livro). Máquinas com valores de hora diferentes param por tempos diferentes!

### 2. Como funcionam `SUMX` e `AVERAGEX`?
As funções terminadas em **X** são **iteradores**. Elas recebem dois argumentos: uma **tabela** a ser percorrida e uma **expressão** a ser calculada linha por linha.

```text
Sintaxe:
SUMX( <Tabela>, <Expressao Avaliada Linha a Linha> )
AVERAGEX( <Tabela>, <Expressao Avaliada Linha a Linha> )
```

1. **Criação de Contexto de Linha:** O iterador vai até a primeira linha da tabela `OS`.
2. **Travessia com `RELATED` (Seção 4.6):** Como a expressão está rodando na linha da tabela `OS` (lado muitos `*`) e o valor da hora está na tabela `Equipamentos` (lado um `1`), usamos `RELATED(Equipamentos[Valor Hora Parada (R$)])` para buscar a tarifa exata daquela máquina.
3. **Cálculo Temporário:** Ele multiplica a hora daquela OS pelo valor da hora daquele equipamento.
4. **Agregação Final:** Depois de repetir isso nas 350 linhas (ou nas linhas filtradas pelo visual), o `SUMX` soma todos os resultados temporários (chegando ao valor exato de **R$ 2.126.716,60**), enquanto o `AVERAGEX` tira a média aritmética desses produtos (**R$ 6.076,33 por OS**).

---

# COLINHA TÉCNICA (CHEAT SHEET DO ALUNO — CAPÍTULO 4)

### A. Operadores do DAX (Seção 4.3 — Tabela 4.4, p. 93)

| Grupo | Operadores | Uso e Exemplo no Dataset da Planta |
| :--- | :---: | :--- |
| **Aritméticos** | `+` `-` `*` `/` `^` | `OS[Horas Paradas] * 60` (converte horas em minutos) |
| **Comparação** | `=` `==` `<>` `>` `<` `>=` `<=` | `OS[Tipo de Manutenção] = "Corretiva"` (`==` diferencia vazio de zero) |
| **Texto** | `&` | `OS[Equipamento] & " - " & OS[Tipo de Manutenção]` |
| **Lógicos** | `&&` (E), `\|\|` (OU), `IN`, `NOT` | `OS[Horas Paradas] > 4 && OS[Tipo de Manutenção] = "Corretiva"` |

### B. Funções do Capítulo 4 (Seções 4.3 a 4.7)

| Função | Seção do Livro | Sintaxe no Dataset | O que faz |
| :--- | :---: | :--- | :--- |
| `SUM` | **4.3** (p. 93) | `SUM(OS[Horas Paradas])` | Soma todos os números da coluna no filtro atual. |
| `AVERAGE` | **4.3** (p. 93) | `AVERAGE(OS[Horas Paradas])` | Média aritmética (o `0` entra na conta; o vazio `BLANK` fica de fora). |
| `MIN` / `MAX` | **4.3** (p. 93) | `MAX(OS[Horas Paradas])` | Menor ou maior valor numérico ou de data no contexto. |
| `COUNT` | **4.3** (p. 93) | `COUNT(OS[Data de Encerramento])` | Conta linhas onde a coluna possui valor preenchido (não vazio). |
| `COUNTROWS` | **4.3** (p. 93) | `COUNTROWS(OS)` | Conta o total de linhas da tabela no contexto (com ou sem vazio). |
| `DISTINCTCOUNT`| **4.3** (p. 93) | `DISTINCTCOUNT(OS[Equipamento])`| Conta quantos valores únicos existem na coluna. |
| `DIVIDE` | **4.4** (p. 95) | `DIVIDE(Num, Den, [Alternativo])`| Divisão segura: devolve `BLANK` (ou alternativo) se o divisor for 0. |
| `IF` | **4.4** (p. 96) | `IF(Teste, Valor_Se_V, [Valor_Se_F])`| Testa uma condição e devolve um entre dois resultados. |
| `SWITCH` | **4.4** (p. 96) | `SWITCH(Expressao, Val1, Res1, ...)`| Compara um valor ou `TRUE()` contra uma lista sequencial de pares. |
| `VAR` / `RETURN`| **4.5** (p. 98) | `VAR Nome = ... RETURN ...` | Guarda resultados parciais em variáveis e organiza a fórmula. |
| `RELATED` | **4.6** (p. 102)| `RELATED(Equipamentos[Área])` | Em contexto de linha (lado `*`), busca um valor na dimensão (lado `1`). |
| `SUMX` | **4.7** (p. 103)| `SUMX(OS, Expressao)` | Calcula uma expressão linha a linha na tabela e soma os resultados. |
| `AVERAGEX` | **4.7** (p. 103)| `AVERAGEX(OS, Expressao)` | Calcula uma expressão linha a linha na tabela e tira a média final. |

---

# CADERNO DE EXERCÍCIOS PRÁTICOS (TURNO DA TARDE)
*(Todos os exercícios utilizam a base real anexada: `OS`, `Equipamentos` e `Calendário`)*

---

### Atividade 1: Verificação do Modelo e Criação da Tabela de Medidas
> **Referência no Livro-Texto:** **Seção 4.1** (*Modelo de Dados Visto pelo DAX*, p. 87–90) e **Seção 4.2** (*Colunas Calculadas e Medidas*, p. 90–92).

1. **Conferência dos Relacionamentos (Seção 4.1):**
   * Abra a **Exibição de Modelo** e verifique se `Equipamentos[Equipamento]` está ligado a `OS[Equipamento]` (`1:*`, filtro único) e se `Calendário[Data]` está ligado a `OS[Data de Abertura]` (`1:*`, filtro único).
2. **Tabela Organizadora de Medidas (Seção 4.2, p. 92):**
   * Na guia *Página Inicial*, clique em **Inserir dados**, dê o nome `Medidas` à tabela e clique em *Carregar*. Depois de criar sua primeira medida nela, clique com o botão direito na `Coluna 1` e escolha **Ocultar da visualização de relatório**.

---

### Atividade 2: Funções de Agregação e a Armadilha do Zero vs. Vazio
> **Referência no Livro-Texto:** **Seção 4.3** (*Sintaxe e Funções de Agregação*, p. 92–95).

Crie as medidas abaixo dentro da tabela `Medidas` e coloque-as em visuais de Cartão para conferir os resultados:

1. **Soma de Horas e Custos (Seção 4.3):**
   ```dax
   Total de Horas Paradas = SUM(OS[Horas Paradas])
   ```
   ```dax
   Custo Total = SUM(OS[Custo (R$)])
   ```
   *Gabarito na Base Real:* `Total de Horas Paradas` = **1.825,21 h** | `Custo Total` = **R$ 1.856.370,13**.

2. **Contagem de Linhas vs. Contagem de Valores Preenchidos (Seção 4.3, Tabela 4.5, p. 94):**
   ```dax
   Quantidade de OS = COUNTROWS(OS)
   ```
   ```dax
   OS com Encerramento = COUNT(OS[Data de Encerramento])
   ```
   ```dax
   Equipamentos com OS = DISTINCTCOUNT(OS[Equipamento])
   ```
   *Gabarito na Base Real:*
   * `Quantidade de OS` = **350** (todas as linhas da tabela: 288 encerradas, 40 abertas e 22 canceladas).
   * `OS com Encerramento` = **288** (a função `COUNT` ignora as 62 células em branco das OS abertas e canceladas!).
   * `Equipamentos com OS` = **20** equipamentos distintos.

3. **Maior Parada e Média Simples (Seção 4.3, Figura 4.7, p. 94–95):**
   ```dax
   Maior Parada = MAX(OS[Horas Paradas])
   ```
   ```dax
   Média de Horas Paradas = AVERAGE(OS[Horas Paradas])
   ```
   *Gabarito e Discussão em Sala:*
   * `Maior Parada` = **22,37 h** (OS `51216` do equipamento `DC-01`).
   * `Média de Horas Paradas` = **5,21 h** (`1.825,21 / 350`).
   * **Pergunta de Ouro para a Turma (Orientações da Base + p. 94 do Livro):** Por que `AVERAGE` deu `5,21 h` se a média real das OS encerradas é `6,34 h` (`1.825,21 / 288`)? Porque na nossa base as 62 OS Abertas e Canceladas vieram preenchidas com `0` na coluna `Horas Paradas`, e não vazias (`BLANK`)! Como ensina a **Seção 4.3 (p. 94)**, o `0` entra no divisor do `AVERAGE`.

---

### Atividade 3: Divisão Segura (`DIVIDE`) e Classificações Lógicas (`IF` e `SWITCH`)
> **Referência no Livro-Texto:** **Seção 4.2** (p. 91) e **Seção 4.4** (*Divisão Segura e Funções Lógicas*, p. 95–98).

1. **Medida de Custo por Hora Parada com `DIVIDE` (Seções 4.2 e 4.4, p. 91 e 95):**
   ```dax
   Custo por Hora Parada = DIVIDE(SUM(OS[Custo (R$)]), SUM(OS[Horas Paradas]))
   ```
   *Gabarito na Base Real:* Geral da Planta = **R$ 1.017,07 / h**.
   *Teste de Divisão por Zero:* Coloque uma segmentação de `Status` na tela e clique em **Aberta** ou **Cancelada** (onde a soma de horas paradas é `0`). Com `DIVIDE`, o cartão fica em branco (`BLANK`) em vez de estourar o erro `Infinito`.

2. **Coluna Calculada com `IF` (Seção 4.4, p. 96):**
   Clique na tabela `OS`, escolha **Nova coluna** e crie o classificador de paradas longas (8 horas ou mais):
   ```dax
   Parada Longa = IF(OS[Horas Paradas] >= 8, "Longa", "Normal")
   ```
   *Gabarito na Base Real:* **66 OS** serão classificadas como `"Longa"` e **284 OS** como `"Normal"`.

3. **Coluna Calculada com `SWITCH(TRUE())` (Seção 4.4, Figura 4.9 e Tabela 4.7, p. 97):**
   Ainda na tabela `OS`, clique em **Nova coluna** e classifique as ordens por faixa de custo respeitando a ordem do menor para o maior:
   ```dax
   Faixa de Custo = 
   SWITCH(
       TRUE(),
       OS[Custo (R$)] <= 1000, "Baixo",
       OS[Custo (R$)] <= 5000, "Médio",
       "Alto"
   )
   ```
   *Gabarito na Base Real:* **52 OS** caem em `"Baixo"`, **166 OS** em `"Médio"` e **132 OS** em `"Alto"`.

---

### Atividade 4: Variáveis (`VAR` / `RETURN`) e Formatação de Fórmulas
> **Referência no Livro-Texto:** **Seção 4.5** (*Variáveis e Formatação das Fórmulas*, p. 98–100).

1. **Medida Estruturada com Variáveis e Comentário (Seção 4.5, Figura 4.11, p. 98–99):**
   Crie na tabela `Medidas` o cálculo de horas médias por OS usando variáveis e quebras de linha (`Shift+Enter`):
   ```dax
   Horas Médias por OS = 
   VAR Horas = SUM(OS[Horas Paradas])
   VAR Quantidade = COUNTROWS(OS)
   RETURN
       DIVIDE(Horas, Quantidade)
   -- Regra: divide o total de horas pelo total de OS do contexto
   ```
2. **Teste de Depuração (Seção 4.5, p. 98):**
   Peça aos alunos para trocarem temporariamente a linha após o `RETURN` apenas por `Horas` e depois por `Quantidade`, observando como as variáveis permitem auditar cada etapa da conta sem criar medidas extras.

---

### Atividade 5: Contexto de Linha, `RELATED` e Iteradores (`SUMX` e `AVERAGEX`)
> **Referência no Livro-Texto:** **Seção 4.6** (*Contexto de Linha e Contexto de Filtro*, p. 100–102) e **Seção 4.7** (*Iteradores SUMX e AVERAGEX*, p. 103–105).

1. **Buscando Dados do Cadastro com `RELATED` (Seção 4.6, Figura 4.14, p. 102):**
   Na tabela `OS`, crie uma **Nova coluna** para trazer a Criticidade do equipamento:
   ```dax
   Criticidade do Equipamento = RELATED(Equipamentos[Criticidade])
   ```
   *Explicação em Sala:* Sem o `RELATED`, digitar `Equipamentos[Criticidade]` direto na coluna calculada da `OS` gera erro, pois o contexto de linha não atravessa o relacionamento sozinho.

2. **Calculando o Custo de Parada (Indisponibilidade) com `SUMX` (Seção 4.7, p. 103):**
   Crie na tabela `Medidas` a fórmula que percorre a tabela `OS` multiplicando as horas paradas de cada ordem pelo valor da hora parada daquela máquina no cadastro:
   ```dax
   Custo de Parada = 
   SUMX(
       OS, 
       OS[Horas Paradas] * RELATED(Equipamentos[Valor Hora Parada (R$)])
   )
   ```
   *Gabarito na Base Real:* **R$ 2.126.716,60** (muito diferente do atalho errado de multiplicar o total de horas pela média simples dos valores hora, que daria R$ 2.189.339,39).

3. **Custo de Parada Médio por OS com `AVERAGEX` (Seção 4.7, p. 104):**
   Crie a medida que tira a média dos produtos calculados linha a linha:
   ```dax
   Custo de Parada Médio por OS = 
   AVERAGEX(
       OS, 
       OS[Horas Paradas] * RELATED(Equipamentos[Valor Hora Parada (R$)])
   )
   ```
   *Gabarito na Base Real:* **R$ 6.076,33** por OS.

---

# 5. GABARITO DE CONFERÊNCIA FINAL DO LABORATÓRIO (BASE REAL ANEXADA)

Ao final da tarde (15:00), peça aos alunos para montarem uma **Matriz** com `Equipamentos[Área]` e `Equipamentos[Equipamento]` nas linhas e as medidas criadas nos valores. Os números na tela de todos devem bater exatamente com as tabelas abaixo:

### A. Resumo Consolidado por Área (`Equipamentos[Área]`)

| Área | Quantidade de OS | Total de Horas Paradas | Custo Total (OS) | Custo por Hora Parada | Custo de Parada (`SUMX`) | Custo Parada Médio (`AVERAGEX`) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Beneficiamento** | 95 | 491,20 h | R$ 418.210,87 | R$ 851,41 / h | R$ 367.855,60 | R$ 3.872,16 |
| **Britagem** | 109 | 561,01 h | R$ 585.816,74 | R$ 1.044,22 / h | R$ 584.530,80 | R$ 5.362,67 |
| **Concentração** | 39 | 212,67 h | R$ 217.560,72 | R$ 1.023,00 / h | R$ 141.158,60 | R$ 3.619,45 |
| **Filtragem** | 40 | 222,22 h | R$ 207.285,37 | R$ 932,79 / h | R$ 228.597,20 | R$ 5.714,93 |
| **Moagem** | 32 | 164,89 h | R$ 220.474,01 | R$ 1.337,10 / h | R$ 272.770,10 | R$ 8.524,07 |
| **Pelotização** | 35 | 173,22 h | R$ 207.022,42 | R$ 1.195,14 / h | R$ 348.765,50 | R$ 9.964,73 |
| **Utilidades** | 20 | 106,33 h | R$ 143.930,42 | R$ 1.353,62 / h | R$ 140.355,60 | R$ 7.017,78 |
| **TOTAL PLANTA** | **350** | **1.825,21 h** | **R$ 1.856.370,13** | **R$ 1.017,07 / h** | **R$ 2.126.716,60** | **R$ 6.076,33** |

*(Nota técnica: A soma das linhas de área acima reflete os agrupamentos; o Total Geral da Planta para as 350 OS é exatamente **1.825,21 h** e **R$ 1.856.370,13**).*

### B. Amostra por Equipamento (Para Validação de `SUMX` vs. `DIVIDE`)

| Equipamento | Área | Valor Hora (Cadastro) | Qtd OS | Total de Horas Paradas | Custo Total (OS) | Custo por Hora Parada (`DIVIDE`) | Custo de Parada (`SUMX`) |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **BR-01** | Britagem | R$ 1.850 | 18 | 93,97 h | R$ 104.264,53 | R$ 1.109,55 / h | R$ 173.844,50 |
| **FO-01** | Pelotização | R$ 2.400 | 18 | 86,29 h | R$ 84.991,92 | R$ 984,96 / h | R$ 207.096,00 |
| **MO-01** | Moagem | R$ 2.100 | 16 | 93,86 h | R$ 117.163,91 | R$ 1.248,28 / h | R$ 197.106,00 |
| **MO-02** | Moagem | R$ 1.980 | 16 | 71,03 h | R$ 75.595,04 | R$ 1.064,27 / h | R$ 140.639,40 |
| **PE-01** | Pelotização | R$ 1.750 | 17 | 80,95 h | R$ 96.403,38 | R$ 1.190,90 / h | R$ 141.662,50 |
| **SM-01** | Concentração| R$ 680 | 16 | 85,48 h | R$ 72.186,87 | R$ 844,49 / h | R$ 58.126,40 |
| **SM-02** | Concentração| R$ 650 | 23 | 127,19 h | R$ 145.373,85 | R$ 1.142,97 / h | R$ 82.673,50 |
| **TR-02** | Britagem | R$ 640 | 25 | 168,26 h | R$ 160.343,19 | R$ 952,95 / h | R$ 107.686,40 |
