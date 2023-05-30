# Somar Valores únicosa partir de ID's e Valores Duplicados utilizando Linguagem DAX
Este repositório oferece trechos de código DAX e scripts para identificar e gerenciar IDs duplicados, garantindo a integridade dos dados e uma análise precisa. Aborde efetivamente o problema comum de duplicação de IDs e aprimore a qualidade dos dados em apenas algumas linhas de código.

## Introdução

A ocorrência de duplicação de IDs em bancos de dados é um problema comum que pode afetar a integridade dos dados e a precisão das análises feitas a partir desses dados. Duplicatas podem surgir devido a várias razões, como erros humanos, problemas no processo de geração de IDs únicos ou integração de dados de diferentes fontes. Isso pode levar a complicações na análise de dados, como cálculos incorretos de indicadores de desempenho ou dificuldades em rastrear o histórico de uma proposta específica.

Neste post, apresento uma solução em DAX para lidar com essas duplicatas, caso ocorram.

## Problema de Negócio

No caso em questão, queremos calcular o valor total das propostas ganhas em uma região específica, considerando apenas as propostas com data de fechamento a partir de 2023. No entanto, existem valores duplicados de ID_PROPOSTA devido à duplicação de produtos incluídos na mesma proposta.

##### Observemos esta tabela problemática

| ID_PROPOSAL | REGION | STATUS                   | CLOSING_DATE | PROPOSAL_TOTAL_VALUE |  PRODUCT  |
|-------------|--------|--------------------------|--------------|----------------------|-----------|
| ID_1        | A      | Won proposal             | 2023-02-01   | 100                  | Product A |
| ID_1        | A      | Won proposal             | 2023-02-01   | 100                  | Product B |
| ID_2        | B      | Won proposal             | 2023-02-03   | 200                  | Product A |
| ID_3        | C      | Won proposal             | 2023-01-04   | 300                  | Product C |
| ID_3        | C      | Won proposal             | 2023-01-04   | 300                  | Product C |
| ID_4        | A      | Unsuccessful proposal    | 2023-02-01   | 100                  | Product A |
| ID_5        | A      | Unsuccessful proposal    | 2023-02-01   | 100                  | Product B |
| ID_6        | B      | Won proposal             | 2022-02-03   | 200                  | Product A |
| ID_7        | C      | Won proposal             | 2022-01-04   | 300                  | Product C |
| ID_8        | C      | Unsuccessful proposal    | 2022-01-04   | 300                  | Product C |

## Solução: Utilizando Medida DAX

Para calcular corretamente o valor total de propostas distintas ganhas sem a duplicação causada por produtos duplicados para o ano de 2023, podemos usar a seguinte medida DAX:

### Fórmula DAX

```DAX
DISTINCT_VALUES_FROM_DISTINCT_WON_PROPOSAL =
CALCULATE(
    SUMX(
        DISTINCT('Table_1'[ID_PROPOSAL]),
        FIRSTNONBLANK('Table_1'[TOTAL_VALUE], 0)
    ),
    FILTER('Table_1', 'Table_1'[STATUS] = "Won proposal"),
    FILTER('Table_1', 'Table_1'[CLOSING_DATE].[Year] >= 2023)
)
```

### Descrição da Medida
# A medida utiliza as seguintes variáveis:

- 'Table_1': É o nome da tabela que contém os dados a serem analisados.
- 'ID_PROPOSAL': É o nome da coluna que contém o identificador único para cada proposta.
- 'TOTAL_VALUE': É o nome da coluna que contém o valor total de cada proposta.
- 'STATUS': É o nome da coluna que indica o status de cada proposta (ganha, perdida, em andamento, etc.).
- 'CLOSING_DATE': É o nome da coluna que indica a data de fechamento de cada proposta.
- 'Year': É uma função que extrai o ano a partir da data de fechamento da proposta.

### Solução e Resultados

Os objetivos da medida são os seguintes:

- Somar o valor total de todas as propostas ganhas em uma região específica.
- Considerar apenas as propostas com data de fechamento a partir de 2023.
- Remover duplicatas ao contar o valor total, considerando apenas o primeiro valor não nulo encontrado.
- Aplicar filtros para restringir a análise às propostas ganhas na região A.

Ao implementar essa medida no Power BI, você pode calcular a soma dos valores totais de todas as propostas ganhas ('Proposta ganha') em uma região e produto específicos, considerando apenas as propostas que foram fechadas após o ano de 2023.

##### A função 'CALCULATE' é utilizada para modificar o contexto de filtro da tabela 'Table_1' e filtrar os dados com base nas condições especificadas posteriormente na medida.

##### A função 'SUMX' é utilizada para somar o resultado de uma expressão para cada linha de uma tabela. Neste caso, a expressão está sendo aplicada a cada ID_PROPOSAL distinto na tabela, que é obtido usando a função DISTINCT.

##### A função 'FIRSTNONBLANK' está sendo usada para obter o primeiro valor não vazio de 'Table_1'[TOTAL_VALUE]. Se o primeiro valor encontrado for vazio, o valor 0 é retornado. Em outras palavras, essa função é usada para substituir os valores nulos na coluna TOTAL_VALUE por zero.

A medida  "DISTINCT_VALUES_FROM_DISTINCT_WON_PROPOSAL" é uma solução poderosa e eficiente para calcular o valor total de propostas ganhas, considerando critérios específicos e tratando duplicatas adequadamente. Ao implementar essa medida no Power BI, é possível obter resultados precisos e confiáveis ao somar os valores totais das propostas ganhas em uma região e produto específicos, levando em consideração as restrições definidas. 

Através do uso das funções 'CALCULATE', 'SUMX' e 'FIRSTNONBLANK', é possível modificar o contexto de filtro, realizar a soma dos valores por ID_PROPOSAL distinto e substituir valores nulos por zero, garantindo a integridade dos cálculos. Com essa medida, os profissionais de análise de dados podem ter uma visão mais completa e precisa das propostas ganhas, permitindo uma tomada de decisão mais embasada e eficaz.
