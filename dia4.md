

# 08:00 – 08:45 | BLOCO 1: O MOTOR ANALÍTICO DO DAX E O FLUXO DE DADOS DA MANUTENÇÃO

## 1. A Provocação Operacional: A Pergunta da PF-01 (10 min)

* **O Cenário de Campo:** O supervisor de mina chega à sala do PCM e faz uma pergunta direta:
  > *“Quanto custa, em reais, cada hora que a perfuratriz PF-01 fica parada na praça de perfuração?”*

* **Os Registros Brutos da Ordem de Serviço:**
  * **OS 1 (Preventiva curta):** 6,0 h paradas | Custo: R$ 9.000 (R$ 1.500/h)
  * **OS 2 (Ajuste rápido com troca pontual):** 2,5 h paradas | Custo: R$ 2.000 (R$ 800/h)
  * **OS 3 (Corretiva média):** 4,0 h paradas | Custo: R$ 6.000 (R$ 1.500/h)

* **O Erro da Análise Ingênua:**
  * Média simples das taxas horárias das OS:
    $$(1.500 + 800 + 1.500) \div 3 = \text{R\$\,1.266,67/h}$$
  * Razão ponderada real de engenharia:
    $$\frac{\text{Custo Total Real}}{\text{Total de Horas Paradas}} = \frac{\text{R\$\,17.000}}{\text{12,5 h}} = \text{R\$\,1.360,00/h}$$

* **O Impacto no PCM:** Uma distorção de quase **R$ 100 por hora parada** por falha de agregação estatística. Multiplicada por centenas de horas anuais, essa distorção compromete o orçamento e o cálculo de perdas da manutenção.

* **O Papel do DAX:** O DAX (*Data Analysis Expressions*) não é uma camada de digitação estática: ele é o **computador de bordo analítico** do modelo, projetado para processar regras de negócio ponderadas sobre grandes volumes de apontamentos de oficina em tempo real.

---

## 2. Como o DAX Enxerga a Base da Planta: Tabelas e Colunas (15 min)

* **O Conceito de Banco Colunar:**
  * No CMMS/ERP (SAP PM, IBM Maximo, TOTVS ou banco SQL relacional), os dados são estruturados em **tabelas de banco de dados**.
  * O DAX não referencia coordenadas físicas de células (como endereços de linha e coluna de planilhas). Ele opera sobre **vetores colunares inteiros**.

* **Anatomia de uma Expressão DAX:**
  $$\text{Nome da Métrica} = \text{FUNÇÃO}(\text{Tabela}[\text{Coluna}])$$

  *Exemplo de aplicação:*
  ```dax
  Total de Horas Paradas = SUM(OS[Horas Paradas])
  ```
  O DAX identifica a entidade de dados (`OS`) e o atributo físico a ser agregado (`[Horas Paradas]`), processando a consulta diretamente na memória colunar do modelo.

---

## 3. A Dinâmica de "Comportas e Válvulas" no Esquema Estrela (15 min)

Para entender a propagação de cálculos no DAX sem a necessidade de reescrever fórmulas por equipamento ou área, utiliza-se a analogia física do fluxo hidráulico:

```text
[Cadastro: Equipamentos]  --- (Válvula / Relacionamento 1:N) --->  [Eventos: Ordens de Serviço]  --->  [Sensor DAX: Medida]
  Filtro: "Área = Pelotamento"                                    Só passam as OS de Pelotamento       Calcula o resultado
```

1. **A Tabela Fato (`OS`): O Fluxo Contínuo de Apontamentos**
   * Tabela transacional volumosa com entradas a cada turno: número de OS, horas de máquina parada, custos de sobressalentes e serviços aplicados.

2. **A Tabela Dimensão (`Equipamentos`): O Cadastro Mestre da Planta**
   * Cadastro padronizado contendo TAGs unívocos, áreas operacionais, criticidade e taxa padrão cadastral de indisponibilidade.

3. **O Relacionamento como Linha de Fluxo Unidirecional:**
   * O relacionamento entre `Equipamentos[Equipamento]` e `OS[Equipamento]` atua como uma **válvula de retenção unidirecional**.
   * Quando o gestor seleciona a área **"Britagem"** no painel:
     * O cadastro restringe o fluxo às TAGs da Britagem;
     * Pelo duto do relacionamento, **apenas as OS associadas a britadores** transitam para a tabela fato;
     * O DAX fica posicionado na extremidade do duto: a medida `SUM(OS[Horas Paradas])` não precisa de parâmetros fixos; ela processa exclusivamente o conjunto de dados filtrado que ultrapassou a válvula naquele instante.

---

## 4. Conclusão e Transição para o Bloco 2 (5 min)

* Se o cálculo precisa reagir dinamicamente a cada filtro acionado na interface (turno, equipe, criticidade ou TAG), a conta **não pode estar fixada em uma linha ou registro estático**.
* Ela precisa operar como um **sensor analítico dinâmico**: esse é o conceito fundamental de uma **Medida DAX**, base da diferenciação contra Colunas Calculadas abordada no bloco seguinte.

---

### Resumo do Encadeamento do Dia

1. **08:00 – 08:45 (Bloco 1):** O problema de negócio da PF-01, DAX como motor de consulta colunar e a dinâmica de propagação de filtros no modelo da planta.
2. **08:45 – 09:30 (Bloco 2):** Medidas (sensores dinâmicos) vs. Colunas Calculadas (atributos físicos gravados) e agregações fundamentais (`SUM`, `AVERAGE`, `COUNTROWS`).
3. **09:45 – 10:45 (Bloco 3):** Tratamento de paradas nulas com `DIVIDE`, classificação de criticidade com `SWITCH(TRUE())` e estruturação com `VAR`/`RETURN`.
4. **10:45 – 12:00 (Bloco 4):** Cruzamento de custos cadastrais com `RELATED`, cálculo do custo total de indisponibilidade com `SUMX`, e modificação de contexto com `CALCULATE` e `REMOVEFILTERS`.
5. **13:00 – 15:30 (Laboratório Prático):** Construção da matriz gerencial de confiabilidade e fechamento analítico do caso da PF-01.
