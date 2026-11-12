Aqui está uma reformulação completa do **Bloco 1 (08:00 – 08:45)** com uma abordagem puramente voltada à **operação de manutenção industrial, engenharia de confiabilidade e fluxo de dados sistêmicos (ERP/CMMS)**, eliminando qualquer menção ou comparação com planilhas e células.

---

### 08:00 – 08:45 | BLOCO 1 (REFORMULADO): O MOTOR ANALÍTICO DO DAX E O FLUXO DE DADOS DA MANUTENÇÃO

#### 1. A Provocação Operacional: A Pergunta da PF-01 (10 min)
* **O Cenário de Campo:** O supervisor de mina chega à sala do PCM e faz uma pergunta direta:
  > *“Quanto custa, em reais, cada hora que a perfuratriz PF-01 fica parada na praça de perfuração?”*
* **Os Registros Brutos da Ordem de Serviço:**
  * **OS 1 (Preventiva curta):** 6,0 h paradas | Custo: R$ 9.000 (R$ 1.500/h)
  * **OS 2 (Ajuste rápido com troca pontual):** 2,5 h paradas | Custo: R$ 2.000 (R$ 800/h)
  * **OS 3 (Corretiva média):** 4,0 h paradas | Custo: R$ 6.000 (R$ 1.500/h)
* **O Erro da Análise Ingênua:**
  * Se calcularmos a média das taxas horárias das OS: $(1.500 + 800 + 1.500) \div 3 = \mathbf{R\$\, 1.266,67/h}$.
  * Se calcularmos a razão real de engenharia: $\frac{\text{Custo Total Real}}{\text{Total de Horas Paradas}} = \frac{\text{R\$\, } 17.000}{12,5\text{ h}} = \mathbf{R\$\, 1.360,00/h}$.
* **O Impacto no PCM:** Uma distorção de quase **R$ 100 por hora parada** por falha de cálculo estatístico. Em centenas de horas no ano, o orçamento de manutenção é desvirtuado.
* **O Papel do DAX:** O DAX (*Data Analysis Expressions*) não é uma ferramenta de digitação: ele é o **computador de bordo analítico** do modelo, criado para processar regras de negócio ponderadas sobre grandes volumes de apontamentos de oficina.

---

#### 2. Como o DAX Enxerga a Base da Planta: Tabelas e Colunas (15 min)
* **O Conceito de Banco Colunar:**
  * No sistema de manutenção (seja SAP PM, Maximo, Totvs ou banco SQL), os dados chegam como **tabelas de banco de dados**.
  * O DAX não aponta para posições físicas ou linhas soltas. Ele enxerga **vetores inteiros** (colunas completas de uma tabela).
* **Anatomia de uma Expressão DAX:**
  $$\text{Nome da Métrica} = \text{FUNÇÃO}(\text{Tabela}[\text{Coluna}])$$
  * *Exemplo real:*
    ```dax
    Total de Horas Paradas = SUM(OS[Horas Paradas])
    ```
  * O DAX sabe exatamente qual tabela ler (`OS`) e qual atributo físico somar (`[Horas Paradas]`). Ele não precisa de coordenadas porque ele atua como uma consulta direta aos dados carregados em memória.

---

#### 3. A Dinâmica de "Comportas e Válvulas" no Esquema Estrela (15 min)
Para explicar como o DAX calcula sem que precisemos reescrever a fórmula a cada máquina ou setor, utiliza-se a analogia física da tubulação industrial:

```
[Cadastro: Equipamentos]  --- (Válvula / Relacionamento 1:N) --->  [Eventos: Ordens de Serviço]  --->  [Sensor DAX: Medida]
  Filtro: "Área = Pelotamento"                                    Só passam as OS de Pelotamento       Calcula o resultado
```

1. **A Tabela Fato (`OS`): O Fluxo Contínuo de Apontamentos**
   * É a tabela pesada, com milhares de linhas que entram a cada turno: número de OS, horas de máquina parada, custos de sobressalentes e serviços.
2. **A Tabela Dimensão (`Equipamentos`): O Cadastro Mestre da Planta**
   * Contém as TAGs únicas, áreas, criticidades e custos cadastrais de hora parada.
3. **O Relacionamento como Linha de Fluxo Unidirecional:**
   * O relacionamento entre `Equipamentos[Equipamento]` e `OS[Equipamento]` funciona como uma válvula de retenção.
   * Quando o gestor clica na área **"Britagem"** no painel:
     * A tabela de cadastro fecha a passagem para todas as outras áreas.
     * Pelo duto do relacionamento, **apenas as OS pertencentes aos britadores** chegam à tabela fato.
     * O DAX fica posicionado na saída desse duto: a fórmula `SUM(OS[Horas Paradas])` não sabe e não precisa saber quantas máquinas existem; ela apenas processa as ordens que passaram pela válvula naquele milissegundo.

---

#### 4. Conclusão e Transição para o Bloco 2 (5 min)
* Se o cálculo precisa reagir dinamicamente a cada válvula que a equipe abre ou fecha na tela (filtrar por turno, por equipe, por área ou por TAG), a conta **não pode estar travada dentro de uma linha estática**.
* Ela precisa ser um **sensor em tempo real**: é isso que define uma **Medida DAX**, tema central que exploraremos em detalhe a seguir comparando com as colunas da tabela.

---

### Resumo do Encadeamento do Dia com o Novo Bloco 1

1. **08:00 – 08:45 (Bloco 1):** O problema de negócio da PF-01, DAX como motor de consulta colunar e a dinâmica de válvulas/propagação de filtro no modelo da planta.
2. **08:45 – 09:30 (Bloco 2):** Medidas (sensores dinâmicos) vs. Colunas Calculadas (atributos físicos gravados) e agregações fundamentais (`SUM`, `AVERAGE`, `COUNTROWS`).
3. **09:45 – 10:45 (Bloco 3):** Tratamento de falhas e paradas nulas com `DIVIDE`, classificação operacional com `SWITCH(TRUE())` e organização com `VAR`/`RETURN`.
4. **10:45 – 12:00 (Bloco 4):** Cruzamento de dados de custos cadastrais com `RELATED`, cálculo do prejuízo da indisponibilidade com o iterador `SUMX`, e filtros avançados com `CALCULATE` e `REMOVEFILTERS`.
5. **13:00 – 15:30 (Laboratório):** Construção hands-on da matriz gerencial de manutenção e resposta definitiva da pergunta do dia na prática.
