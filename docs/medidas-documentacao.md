# Documentação das Medidas: Projeto Vendas — Veloura Beauty

Este documento lista todas as medidas criadas no modelo Power BI, suas regras de negócio, dependências e retornos esperados.

---

## Medidas de Faturamento
<br>

```DAX
faturamento_total = 

-- Medida:
--      faturamento_total
--
-- Descrição:
--      Calcula o faturamento total considerando todas as vendas registradas na tabela de fatos,
--      respeitando o contexto de filtros do relatório.
--
-- Tabela origem:
--      fato_vendas (com suporte da bridge_preco_produtos)
--
-- Regra de negócio:
--      O faturamento é calculado multiplicando a quantidade de produtos vendidos pelo preço unitário
--      correspondente, obtido através da tabela de preços relacionada.
--
--      O cálculo é realizado linha a linha na tabela de vendas, garantindo precisão mesmo em cenários
--      com variação de preços por produto.
--
-- Dependências:
--      fato_vendas[quantidade]
--      bridge_preco_produtos[preco]
--
-- Retorno:
--      Valor numérico representando o faturamento total no contexto filtrado.
--
-- Observação:
--      A função SUMX permite iterar sobre a tabela fato_vendas e calcular o faturamento por linha.
--      RELATED é utilizado para buscar o preço do produto na tabela relacionada (bridge_preco_produtos).
--      COALESCE garante retorno 0 quando o resultado for BLANK().

VAR _Resultado =
    SUMX(
        fato_vendas,
        fato_vendas[quantidade] * 
        RELATED(
            bridge_preco_produtos[preco]
        )
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
cores_maior_menor_faturamento = 

-- Medida:
--      cores_maior_menor_faturamento
--
-- Descrição:
--      Define cores para destacar visualmente os períodos com maior e menor faturamento
--      dentro do contexto selecionado, facilitando a análise temporal no relatório.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_calendario)
--
-- Regra de negócio:
--      Identifica o maior e o menor faturamento considerando os períodos (meses) visíveis
--      no contexto atual do relatório.
--
--          Maior faturamento:
--              Verde (#33CC33)
--
--          Menor faturamento:
--              Vermelho (#FF0000)
--
--          Demais valores:
--              Cinza (#808080)
--
-- Dependências:
--      [faturamento_total]
--      dimensao_calendario[mes_abreviado]
--      dimensao_calendario[mes_numero]
--
-- Retorno:
--      Código de cor em formato hexadecimal (texto), utilizado para formatação condicional.
--
-- Observação:
--      A função ALLSELECTED garante que o cálculo respeite os filtros aplicados pelo usuário,
--      mas considere todos os períodos visíveis no contexto atual.
--
--      MINX e MAXX são utilizados para identificar os extremos (menor e maior faturamento).
--
--      A lógica permite destacar dinamicamente os pontos de maior e menor desempenho no tempo,
--      reforçando o storytelling visual do relatório.

VAR _MenorFaturamento =
    MINX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [faturamento_total]
    )

VAR _MaiorFaturamento =
    MAXX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [faturamento_total]
    )

RETURN
    SWITCH(
        TRUE(),

        [faturamento_total] = _MaiorFaturamento,
        "#33CC33",

        [faturamento_total] = _MenorFaturamento,
        "#FF0000",

        "#808080"
    )
```
<br>

```DAX
faturamento_loja = 

-- Medida:
--      faturamento_loja
--
-- Descrição:
--      Calcula o faturamento total proveniente do canal de vendas Loja Física,
--      respeitando o contexto de filtros do relatório.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Filtra o faturamento total considerando apenas as vendas realizadas
--      no canal "Loja".
--
-- Dependências:
--      [faturamento_total]
--      dimensao_canais[canal]
--
-- Retorno:
--      Valor numérico representando o faturamento do canal Loja no contexto filtrado.
--
-- Observação:
--      A função CALCULATE altera o contexto de filtro para considerar apenas o canal Loja.
--      COALESCE é utilizado para garantir retorno 0 quando o resultado for BLANK().

VAR _Resultado =
    CALCULATE(
        [faturamento_total],
        dimensao_canais[canal] = "Loja"
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
faturamento_ecommerce = 

-- Medida:
--      faturamento_ecommerce
--
-- Descrição:
--      Calcula o faturamento total proveniente do canal de vendas e-Commerce,
--      respeitando o contexto de filtros do relatório.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Filtra o faturamento total considerando apenas as vendas realizadas
--      no canal "e-Commerce".
--
-- Dependências:
--      [faturamento_total]
--      dimensao_canais[canal]
--
-- Retorno:
--      Valor numérico representando o faturamento do canal e-Commerce no contexto filtrado.
--
-- Observação:
--      A função CALCULATE altera o contexto de filtro para considerar apenas o canal e-Commerce.
--      COALESCE é utilizado para garantir retorno 0 quando o resultado for BLANK().

VAR _Resultado =
    CALCULATE(
        [faturamento_total],
        dimensao_canais[canal] = "e-Commerce"
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
maior_menor_percentual_variacao_faturamento_loja_ecommerce = 

-- Medida:
--      maior_menor_percentual_variacao_faturamento_loja_ecommerce
--
-- Descrição:
--      Identifica e retorna os valores de maior e menor variação percentual de faturamento
--      entre os canais Loja e e-Commerce dentro do contexto temporal selecionado.
--
-- Tabela origem:
--      Medida calculada (baseada em dimensao_calendario e métricas de variação)
--
-- Regra de negócio:
--      Calcula a maior e a menor variação percentual considerando os períodos (meses)
--      visíveis no contexto atual do relatório.
--
--      Retorna o valor da variação apenas quando ele corresponde ao maior ou ao menor valor.
--      Para os demais casos, retorna BLANK().
--
-- Dependências:
--      [percentual_variacao_faturamento_loja_ecommerce]
--      dimensao_calendario[mes_abreviado]
--      dimensao_calendario[mes_numero]
--
-- Retorno:
--      Valor numérico representando a maior ou menor variação percentual no período analisado,
--      ou BLANK() para os demais casos.
--
-- Observação:
--      A função ALLSELECTED garante que o cálculo respeite os filtros aplicados pelo usuário,
--      considerando apenas os períodos visíveis no contexto atual.
--
--      MINX e MAXX são utilizados para identificar os extremos da variação percentual.
--
--      A lógica permite destacar visualmente os pontos de maior crescimento e maior queda,
--      sendo ideal para uso combinado com formatação condicional e storytelling temporal.

VAR _MenorVariacao =
    MINX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [percentual_variacao_faturamento_loja_ecommerce]
    )

VAR _MaiorVariacao =
    MAXX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [percentual_variacao_faturamento_loja_ecommerce]
    )

RETURN
    IF(
        [percentual_variacao_faturamento_loja_ecommerce] = _MaiorVariacao
        || [percentual_variacao_faturamento_loja_ecommerce] = _MenorVariacao,
        [percentual_variacao_faturamento_loja_ecommerce]
    )
```
<br>

```DAX
formato_variacao_faturamento = 

-- Medida:
--      formato_variacao_faturamento
--
-- Descrição:
--      Formata a variação percentual de faturamento entre canais (Loja e e-Commerce),
--      adicionando indicadores visuais (setas) para facilitar a interpretação do resultado.
--
-- Tabela origem:
--      Medida calculada (baseada em [maior_menor_percentual_variacao_faturamento_loja_ecommerce])
--
-- Regra de negócio:
--      Aplica formatação condicional ao valor percentual:
--
--          Valores positivos:
--              Exibidos com símbolo ▲
--
--          Valores negativos:
--              Exibidos com símbolo ▼
--
--      O valor é apresentado no formato percentual com duas casas decimais.
--
-- Dependências:
--      [maior_menor_percentual_variacao_faturamento_loja_ecommerce]
--
-- Retorno:
--      Texto formatado representando a variação percentual com indicadores visuais.
--
-- Observação:
--      A função FORMAT converte o valor numérico em texto, permitindo customização visual.
--      O padrão "▲ 0.00%; ▼ 0.00%" define formatos distintos para valores positivos e negativos.
--
--      Atenção: como o resultado é texto, essa medida não deve ser utilizada em cálculos adicionais,
--      sendo indicada apenas para exibição em visuais.

VAR _Resultado =
    FORMAT(
        [maior_menor_percentual_variacao_faturamento_loja_ecommerce],
        "▲ 0.00%; ▼ 0.00%"
    )

RETURN
    _Resultado
```
<br>

```DAX
percentual_variacao_faturamento_loja_ecommerce = 

-- Medida:
--      percentual_variacao_faturamento_loja_ecommerce
--
-- Descrição:
--      Calcula a variação percentual do faturamento entre os canais Loja e e-Commerce,
--      utilizando o faturamento da Loja como base de comparação.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Subtrai o faturamento do e-Commerce do faturamento da Loja e divide pelo faturamento da Loja,
--      resultando na variação percentual relativa ao canal Loja.
--
--      Interpretação:
--
--          Valor positivo:
--              Loja possui faturamento superior ao e-Commerce
--
--          Valor negativo:
--              e-Commerce possui faturamento superior à Loja
--
--          Valor próximo de zero:
--              Canais com desempenho semelhante
--
-- Dependências:
--      [faturamento_loja]
--      [faturamento_ecommerce]
--
-- Retorno:
--      Valor numérico representando a variação percentual entre os canais (entre -∞ e +∞).
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--
--      O resultado pode ser formatado como percentual para melhor interpretação.
--
--      Importante: a escolha do canal Loja como base influencia a interpretação da métrica.
--      Caso necessário, pode-se criar uma variação invertida utilizando o e-Commerce como base.

VAR _Resultado =
    DIVIDE(
        [faturamento_loja] - [faturamento_ecommerce],
        [faturamento_loja]
    )

RETURN
    _Resultado
```
<br>

```DAX
formato_variacao_faturamento_mm = 

-- Medida:
--      formato_variacao_faturamento_mm
--
-- Descrição:
--      Formata a variação percentual de faturamento entre canais (Loja e e-Commerce),
--      utilizando indicadores visuais (setas) para facilitar a interpretação do desempenho.
--
-- Tabela origem:
--      Medida calculada (baseada em [percentual_variacao_faturamento_loja_ecommerce])
--
-- Regra de negócio:
--      Aplica formatação condicional ao valor percentual:
--
--          Valores positivos:
--              Exibidos com símbolo ▲
--
--          Valores negativos:
--              Exibidos com símbolo ▼
--
--      O valor é apresentado no formato percentual com duas casas decimais.
--
-- Dependências:
--      [percentual_variacao_faturamento_loja_ecommerce]
--
-- Retorno:
--      Texto formatado representando a variação percentual com indicadores visuais.
--
-- Observação:
--      A função FORMAT converte o valor numérico em texto, permitindo customização visual.
--      O padrão "▲ 0.00%; ▼ 0.00%" define formatos distintos para valores positivos e negativos.
--
--      Atenção: como o resultado é texto, essa medida não deve ser utilizada em cálculos adicionais,
--      sendo indicada apenas para exibição em visuais.

VAR _Resultado =
    FORMAT(
        [percentual_variacao_faturamento_loja_ecommerce],
        "▲ 0.00%; ▼ 0.00%"
    )

RETURN
    _Resultado
```
<br>

```DAX
percentual_faturamento_ecommerce = 

-- Medida:
--      percentual_faturamento_ecommerce
--
-- Descrição:
--      Calcula a participação percentual do faturamento do canal e-Commerce
--      em relação ao faturamento total, respeitando o contexto de filtros do relatório.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Divide o faturamento do canal e-Commerce pelo faturamento total,
--      resultando na representatividade percentual desse canal no contexto analisado.
--
-- Dependências:
--      [faturamento_total]
--      [faturamento_ecommerce]
--
-- Retorno:
--      Valor numérico representando a participação percentual do e-Commerce (entre 0 e 1).
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      A medida pode ser formatada como percentual nos visuais do Power BI
--      para melhor interpretação pelo usuário.

VAR _FaturamentoTotal =
    [faturamento_total]

VAR _FaturamentoEcommerce =
    [faturamento_ecommerce]

VAR _Resultado =
    DIVIDE(
        _FaturamentoEcommerce,
        _FaturamentoTotal
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
percentual_faturamento_ecommerce_mm = 

-- Medida:
--      percentual_faturamento_ecommerce_mm
--
-- Descrição:
--      Calcula a participação do faturamento do canal e-Commerce em relação ao faturamento total
--      de e-Commerce considerando todo o período (ignorando filtros de tempo).
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas, dimensao_canais e dimensao_calendario)
--
-- Regra de negócio:
--      Divide o faturamento de e-Commerce no contexto atual pelo faturamento total de e-Commerce
--      sem aplicação de filtros de calendário, resultando na participação relativa no período completo.
--
-- Dependências:
--      [faturamento_ecommerce]
--      dimensao_calendario
--
-- Retorno:
--      Valor numérico representando a participação percentual do e-Commerce em relação ao total geral (entre 0 e 1).
--
-- Observação:
--      A função REMOVEFILTERS remove o contexto de tempo, permitindo calcular o total global.
--      A função DIVIDE evita erros de divisão por zero.
--
--      Essa medida é útil para análises de contribuição ao longo do tempo,
--      como participação mensal no total acumulado do período.

VAR _Resultado =
    DIVIDE(
        [faturamento_ecommerce],
        CALCULATE(
            [faturamento_ecommerce],
            REMOVEFILTERS(
                dimensao_calendario
            )
        )
    )

RETURN
    _Resultado
```
<br>

```DAX
percentual_faturamento_loja = 

-- Medida:
--      percentual_faturamento_loja
--
-- Descrição:
--      Calcula a participação percentual do faturamento do canal Loja
--      em relação ao faturamento total, respeitando o contexto de filtros do relatório.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Divide o faturamento do canal Loja pelo faturamento total,
--      resultando na representatividade percentual desse canal no contexto analisado.
--
-- Dependências:
--      [faturamento_total]
--      [faturamento_loja]
--
-- Retorno:
--      Valor numérico representando a participação percentual da Loja (entre 0 e 1).
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      A medida pode ser formatada como percentual nos visuais do Power BI
--      para melhor interpretação pelo usuário.

VAR _FaturamentoTotal =
    [faturamento_total]

VAR _FaturamentoLoja =
    [faturamento_loja]

VAR _Resultado =
    DIVIDE(
        _FaturamentoLoja,
        _FaturamentoTotal
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
percentual_faturamento_loja_mm = 

-- Medida:
--      percentual_faturamento_loja_mm
--
-- Descrição:
--      Calcula a participação do faturamento do canal Loja em relação ao faturamento total
--      de Loja considerando todo o período (ignorando filtros de tempo).
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas, dimensao_canais e dimensao_calendario)
--
-- Regra de negócio:
--      Divide o faturamento de Loja no contexto atual pelo faturamento total de Loja
--      sem aplicação de filtros de calendário, resultando na participação relativa no período completo.
--
-- Dependências:
--      [faturamento_loja]
--      dimensao_calendario
--
-- Retorno:
--      Valor numérico representando a participação percentual da Loja em relação ao total geral (entre 0 e 1).
--
-- Observação:
--      A função REMOVEFILTERS remove o contexto de tempo, permitindo calcular o total global.
--      A função DIVIDE evita erros de divisão por zero.
--
--      Essa medida é útil para análises de contribuição ao longo do tempo,
--      como participação mensal no total acumulado do período.

VAR _Resultado =
    DIVIDE(
        [faturamento_loja],
        CALCULATE(
            [faturamento_loja],
            REMOVEFILTERS(
                dimensao_calendario
            )
        )
    )

RETURN
    _Resultado
```

## Medidas de Transações
<br>

```DAX
transacoes_total = 

-- Medida:
--      transacoes_total
--
-- Descrição:
--      Calcula o total de transações registradas, considerando cada nota fiscal como uma transação única.
--
-- Tabela origem:
--      fato_vendas
--
-- Regra de negócio:
--      - Conta as notas fiscais distintas para determinar o número total de transações no período ou contexto filtrado.
--      - Retorna 0 caso não haja transações para evitar BLANK().
--
-- Dependências:
--      fato_vendas[nota_fiscal]
--
-- Retorno:
--      Valor numérico representando o total de transações distintas no contexto filtrado.
--
-- Observação:
--      COALESCE garante que, se não houver transações registradas, o retorno será 0 ao invés de BLANK().

VAR _Resultado =
    DISTINCTCOUNT(
        fato_vendas[nota_fiscal]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
transacoes_loja = 

-- Medida:
--      transacoes_loja
--
-- Descrição:
--      Calcula o total de transações realizadas pelo canal Loja no período ou contexto filtrado.
--
-- Tabela origem:
--      Medida calculada (baseada em [transacoes_total] e [dimensao_canais])
--
-- Regra de negócio:
--      - Filtra as transações pelo canal "Loja".
--      - Retorna 0 caso não haja transações para evitar BLANK().
--
-- Dependências:
--      [transacoes_total]
--      dimensao_canais[canal]
--
-- Retorno:
--      Valor numérico representando o total de transações via canal Loja no contexto filtrado.
--
-- Observação:
--      COALESCE garante que, se não houver transações registradas, o retorno será 0 ao invés de BLANK().

VAR _Resultado =
    CALCULATE(
        [transacoes_total],
        dimensao_canais[canal]="Loja"
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
transacoes_ecommerce = 

-- Medida:
--      transacoes_ecommerce
--
-- Descrição:
--      Calcula o total de transações realizadas pelo canal e-Commerce no período ou contexto filtrado.
--
-- Tabela origem:
--      Medida calculada (baseada em [transacoes_total] e [dimensao_canais])
--
-- Regra de negócio:
--      - Filtra as transações pelo canal "e-Commerce".
--      - Retorna 0 caso não haja transações para evitar BLANK().
--
-- Dependências:
--      [transacoes_total]
--      dimensao_canais[canal]
--
-- Retorno:
--      Valor numérico representando o total de transações via canal e-Commerce no contexto filtrado.
--
-- Observação:
--      COALESCE garante que, se não houver transações registradas, o retorno será 0 ao invés de BLANK().

VAR _Resultado =
    CALCULATE(
        [transacoes_total],
        dimensao_canais[canal]="e-Commerce"
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
maior_menor_transacoes_loja = 

-- Medida:
--      maior_menor_transacoes_loja
--
-- Descrição:
--      Retorna a maior ou menor quantidade de transações realizadas via canal Loja
--      no período selecionado, destacando os extremos de desempenho.
--
-- Tabela origem:
--      Medida calculada (baseada em [transacoes_loja] e dimensao_calendario)
--
-- Regra de negócio:
--      - Calcula o valor mínimo e máximo de transações Loja no contexto filtrado.
--      - Retorna apenas os valores que correspondem ao menor ou maior número de transações.
--
-- Dependências:
--      [transacoes_loja]
--      dimensao_calendario[mes_abreviado], dimensao_calendario[mes_numero]
--
-- Retorno:
--      Valor numérico representando a quantidade de transações extrema (menor ou maior) do canal Loja.
--
-- Observação:
--      Útil para análise de picos e quedas de transações Loja em dashboards,
--      permitindo comparações rápidas entre períodos.

VAR _MenorTransacao =
    MINX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [transacoes_loja]
    )

VAR _MaiorTransacao =
    MAXX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [transacoes_loja]
    )

RETURN
    IF(
        [transacoes_loja] = _MaiorTransacao
        || [transacoes_loja] = _MenorTransacao,
        [transacoes_loja]
    )
```
<br>

```DAX
maior_menor_transacoes_ecommerce = 

-- Medida:
--      maior_menor_transacoes_ecommerce
--
-- Descrição:
--      Retorna a maior ou menor quantidade de transações realizadas via canal e-Commerce
--      no período selecionado, destacando os extremos de desempenho.
--
-- Tabela origem:
--      Medida calculada (baseada em [transacoes_ecommerce] e dimensao_calendario)
--
-- Regra de negócio:
--      - Calcula o valor mínimo e máximo de transações e-Commerce no contexto filtrado.
--      - Retorna apenas os valores que correspondem ao menor ou maior número de transações.
--
-- Dependências:
--      [transacoes_ecommerce]
--      dimensao_calendario[mes_abreviado], dimensao_calendario[mes_numero]
--
-- Retorno:
--      Valor numérico representando a quantidade de transações extrema (menor ou maior) do canal e-Commerce.
--
-- Observação:
--      Útil para análise de picos e quedas de transações e-Commerce em dashboards,
--      permitindo comparações rápidas entre períodos.

VAR _MenorTransacao =
    MINX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [transacoes_ecommerce]
    )

VAR _MaiorTransacao =
    MAXX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [transacoes_ecommerce]
    )

RETURN
    IF(
        [transacoes_ecommerce] = _MaiorTransacao
        || [transacoes_ecommerce] = _MenorTransacao,
        [transacoes_ecommerce]
    )
```
<br>

```DAX
percentual_transacoes_loja = 

-- Medida:
--      percentual_transacoes_loja
--
-- Descrição:
--      Calcula a proporção de transações realizadas via canal Loja em relação ao total de transações,
--      permitindo analisar a representatividade do canal no período selecionado.
--
-- Tabela origem:
--      Medida calculada (baseada em [transacoes_loja] e [transacoes_total])
--
-- Regra de negócio:
--      - Divide o total de transações Loja pelo total de transações no contexto filtrado.
--      - Retorna 0 caso não haja transações para evitar BLANK().
--
-- Dependências:
--      [transacoes_loja]
--      [transacoes_total]
--
-- Retorno:
--      Valor numérico representando o percentual de transações via Loja no período selecionado.
--
-- Observação:
--      COALESCE garante que, se não houver transações registradas, o retorno será 0 ao invés de BLANK().

VAR _TransacoesTotal =
    [transacoes_total]

VAR _Transacoes_Loja =
    [transacoes_loja]

VAR _Resultado =
    DIVIDE(
        _Transacoes_Loja,
        _TransacoesTotal
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
percentual_transacoes_ecommerce = 

-- Medida:
--      percentual_transacoes_ecommerce
--
-- Descrição:
--      Calcula a proporção de transações realizadas via canal e-Commerce em relação ao total de transações,
--      permitindo analisar a representatividade do canal no período selecionado.
--
-- Tabela origem:
--      Medida calculada (baseada em [transacoes_ecommerce] e [transacoes_total])
--
-- Regra de negócio:
--      - Divide o total de transações e-Commerce pelo total de transações no contexto filtrado.
--      - Retorna 0 caso não haja transações para evitar BLANK().
--
-- Dependências:
--      [transacoes_ecommerce]
--      [transacoes_total]
--
-- Retorno:
--      Valor numérico representando o percentual de transações via e-Commerce no período selecionado.
--
-- Observação:
--      COALESCE garante que, se não houver transações registradas, o retorno será 0 ao invés de BLANK().

VAR _TransacoesTotal =
    [transacoes_total]

VAR _Transacoes_Ecommerce =
    [transacoes_ecommerce]

VAR _Resultado =
    DIVIDE(
        _Transacoes_Ecommerce,
        _TransacoesTotal
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
percentual_variacao_transacoes_loja_ecommerce = 

-- Medida:
--      percentual_variacao_transacoes_loja_ecommerce
--
-- Descrição:
--      Calcula a variação percentual entre o número de transações realizadas pelo canal Loja
--      em relação ao canal e-Commerce, permitindo analisar diferenças de desempenho entre canais.
--
-- Tabela origem:
--      Medida calculada (baseada em [transacoes_loja] e [transacoes_ecommerce])
--
-- Regra de negócio:
--      - Subtrai o total de transações e-Commerce do total de transações Loja.
--      - Divide o resultado pelo total de transações Loja para obter a variação percentual.
--
-- Dependências:
--      [transacoes_loja]
--      [transacoes_ecommerce]
--
-- Retorno:
--      Valor numérico representando a variação percentual de transações entre Loja e e-Commerce.
--
-- Observação:
--      Pode resultar em valores positivos ou negativos, indicando se o canal Loja teve mais ou menos transações
--      comparado ao canal e-Commerce.

VAR _Resultado =
    DIVIDE(
        [transacoes_loja]-[transacoes_ecommerce],
        [transacoes_loja]
    )

RETURN
    _Resultado
```
<br>

```DAX
maior_menor_percentual_variacao_transacoes_loja_ecommerce = 

-- Medida:
--      maior_menor_percentual_variacao_transacoes_loja_ecommerce
--
-- Descrição:
--      Retorna o maior ou menor percentual de variação das transações entre os canais Loja e e-Commerce
--      no período selecionado, facilitando a identificação dos extremos de desempenho.
--
-- Tabela origem:
--      Medida calculada (baseada em [percentual_variacao_transacoes_loja_ecommerce] e dimensao_calendario)
--
-- Regra de negócio:
--      - Calcula o valor mínimo e máximo de variação percentual de transações no contexto filtrado.
--      - Retorna apenas o valor que corresponde ao menor ou maior percentual.
--
-- Dependências:
--      [percentual_variacao_transacoes_loja_ecommerce]
--      dimensao_calendario[mes_abreviado], dimensao_calendario[mes_numero]
--
-- Retorno:
--      Valor numérico representando o percentual de variação da transação que é extremo (menor ou maior) no contexto filtrado.
--
-- Observação:
--      Útil para destacar visualmente picos e quedas nas transações entre canais
--      em gráficos ou cartões de análise comparativa.
--
--      A função IF garante que apenas os valores máximos e mínimos sejam retornados.

VAR _MenorTransacao =
    MINX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [percentual_variacao_transacoes_loja_ecommerce]
    )

VAR _MaiorTransacao =
    MAXX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [percentual_variacao_transacoes_loja_ecommerce]
    )

RETURN
    IF(
        [percentual_variacao_transacoes_loja_ecommerce] = _MaiorTransacao
        || [percentual_variacao_transacoes_loja_ecommerce] = _MenorTransacao,
        [percentual_variacao_transacoes_loja_ecommerce]
    )
```
<br>

```DAX
formato_variacao_transacoes = 

-- Medida:
--      formato_variacao_transacoes
--
-- Descrição:
--      Formata o percentual de variação das transações entre os canais Loja e e-Commerce,
--      exibindo os valores com símbolos de aumento (▲) e redução (▼) para facilitar a interpretação visual.
--
-- Tabela origem:
--      Medida calculada (baseada em [maior_menor_percentual_variacao_transacoes_loja_ecommerce])
--
-- Regra de negócio:
--      Aplica a função FORMAT ao valor da variação percentual, transformando o número em string
--      no formato “▲ 0.00%; ▼ 0.00%”.
--
-- Dependências:
--      [maior_menor_percentual_variacao_transacoes_loja_ecommerce]
--
-- Retorno:
--      Valor formatado em string representando a variação percentual entre os canais.
--
-- Observação:
--      Útil para exibição em dashboards e relatórios, facilitando a rápida identificação
--      de tendências de aumento ou queda nas transações.
--
--      O formato escolhido mantém consistência com outras medidas de variação (faturamento, produtos vendidos).

VAR _Resultado =
    FORMAT(
        [maior_menor_percentual_variacao_transacoes_loja_ecommerce], "▲ 0.00%; ▼ 0.00%"
    )

RETURN
    _Resultado
```

## Medidas de Produtos Vendidos
<br>

```DAX
quantidade_produtos_vendidos_total = 

-- Medida:
--      quantidade_produtos_vendidos_total
--
-- Descrição:
--      Calcula a quantidade total de produtos vendidos, considerando todas as vendas
--      registradas na tabela de fatos e respeitando o contexto de filtros do relatório.
--
-- Tabela origem:
--      fato_vendas
--
-- Regra de negócio:
--      Soma os valores da coluna de quantidade da tabela de vendas,
--      representando o total de produtos vendidos no período ou contexto selecionado.
--
-- Dependências:
--      fato_vendas[quantidade]
--
-- Retorno:
--      Valor numérico representando a quantidade total de produtos vendidos.
--
-- Observação:
--      A função SUM realiza a agregação direta da coluna de quantidade.
--
--      Diferente do faturamento_total (que exige cálculo linha a linha com SUMX),
--      aqui a soma simples é suficiente, pois a quantidade já está no nível correto de granularidade.

VAR _Resultado =
    SUM(
        fato_vendas[quantidade]
    )

RETURN
    _Resultado
```
<br>

```DAX
quantidade_produtos_vendidos_loja = 

-- Medida:
--      quantidade_produtos_vendidos_loja
--
-- Descrição:
--      Calcula a quantidade total de produtos vendidos no canal Loja,
--      respeitando o contexto de filtros do relatório.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Filtra a quantidade total de produtos vendidos considerando apenas
--      as vendas realizadas no canal "Loja".
--
-- Dependências:
--      [quantidade_produtos_vendidos_total]
--      dimensao_canais[canal]
--
-- Retorno:
--      Valor numérico representando a quantidade de produtos vendidos no canal Loja.
--
-- Observação:
--      A função CALCULATE altera o contexto de filtro para considerar apenas o canal Loja.
--      COALESCE é utilizado para garantir retorno 0 quando o resultado for BLANK().

VAR _Resultado =
    CALCULATE(
        [quantidade_produtos_vendidos_total],
        dimensao_canais[canal] = "Loja"
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
quantidade_produtos_vendidos_ecommerce = 

-- Medida:
--      quantidade_produtos_vendidos_ecommerce
--
-- Descrição:
--      Calcula a quantidade total de produtos vendidos no canal e-Commerce,
--      respeitando o contexto de filtros do relatório.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Filtra a quantidade total de produtos vendidos considerando apenas
--      as vendas realizadas no canal "e-Commerce".
--
-- Dependências:
--      [quantidade_produtos_vendidos_total]
--      dimensao_canais[canal]
--
-- Retorno:
--      Valor numérico representando a quantidade de produtos vendidos no canal e-Commerce.
--
-- Observação:
--      A função CALCULATE altera o contexto de filtro para considerar apenas o canal e-Commerce.
--      COALESCE é utilizado para garantir retorno 0 quando o resultado for BLANK().

VAR _Resultado =
    CALCULATE(
        [quantidade_produtos_vendidos_total],
        dimensao_canais[canal] = "e-Commerce"
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
maior_menor_percentual_variacao_produtos_vendidos_loja_ecommerce = 

-- Medida:
--      maior_menor_percentual_variacao_produtos_vendidos_loja_ecommerce
--
-- Descrição:
--      Identifica e retorna os valores de maior e menor variação percentual da quantidade
--      de produtos vendidos entre os canais Loja e e-Commerce dentro do contexto temporal selecionado.
--
-- Tabela origem:
--      Medida calculada (baseada em dimensao_calendario e métricas de variação)
--
-- Regra de negócio:
--      Calcula a maior e a menor variação percentual considerando os períodos (meses)
--      visíveis no contexto atual do relatório.
--
--      Retorna o valor da variação apenas quando ele corresponde ao maior ou ao menor valor.
--      Para os demais casos, retorna BLANK().
--
-- Dependências:
--      [percentual_variacao_produtos_vendidos_loja_ecommerce]
--      dimensao_calendario[mes_abreviado]
--      dimensao_calendario[mes_numero]
--
-- Retorno:
--      Valor numérico representando a maior ou menor variação percentual no período analisado,
--      ou BLANK() para os demais casos.
--
-- Observação:
--      A função ALLSELECTED garante que o cálculo respeite os filtros aplicados pelo usuário,
--      considerando apenas os períodos visíveis no contexto atual.
--
--      MINX e MAXX são utilizados para identificar os extremos da variação percentual.
--
--      A lógica permite destacar visualmente os pontos de maior crescimento e maior queda
--      na quantidade de produtos vendidos, reforçando o storytelling temporal.

VAR _MenorVariacao =
    MINX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [percentual_variacao_produtos_vendidos_loja_ecommerce]
    )

VAR _MaiorVariacao =
    MAXX(
        ALLSELECTED(
            dimensao_calendario[mes_abreviado],
            dimensao_calendario[mes_numero]
        ),
        [percentual_variacao_produtos_vendidos_loja_ecommerce]
    )

RETURN
    IF(
        [percentual_variacao_produtos_vendidos_loja_ecommerce] = _MaiorVariacao
        || [percentual_variacao_produtos_vendidos_loja_ecommerce] = _MenorVariacao,
        [percentual_variacao_produtos_vendidos_loja_ecommerce]
    )
```
<br>

```DAX
formato_variacao_produtos_vendidos = 

-- Medida:
--      formato_variacao_produtos_vendidos
--
-- Descrição:
--      Formata a variação percentual da quantidade de produtos vendidos entre os canais
--      Loja e e-Commerce, adicionando indicadores visuais (setas) para facilitar a interpretação.
--
-- Tabela origem:
--      Medida calculada (baseada em [maior_menor_percentual_variacao_produtos_vendidos_loja_ecommerce])
--
-- Regra de negócio:
--      Aplica formatação condicional ao valor percentual:
--
--          Valores positivos:
--              Exibidos com símbolo ▲
--
--          Valores negativos:
--              Exibidos com símbolo ▼
--
--      O valor é apresentado no formato percentual com duas casas decimais.
--
-- Dependências:
--      [maior_menor_percentual_variacao_produtos_vendidos_loja_ecommerce]
--
-- Retorno:
--      Texto formatado representando a variação percentual com indicadores visuais.
--
-- Observação:
--      A função FORMAT converte o valor numérico em texto, permitindo customização visual.
--      O padrão "▲ 0.00%; ▼ 0.00%" define formatos distintos para valores positivos e negativos.
--
--      Atenção: como o resultado é texto, essa medida não deve ser utilizada em cálculos adicionais,
--      sendo indicada apenas para exibição em visuais.

VAR _Resultado =
    FORMAT(
        [maior_menor_percentual_variacao_produtos_vendidos_loja_ecommerce],
        "▲ 0.00%; ▼ 0.00%"
    )

RETURN
    _Resultado
```
<br>

```DAX
percentual_quantidade_produtos_vendidos_loja = 

-- Medida:
--      percentual_quantidade_produtos_vendidos_loja
--
-- Descrição:
--      Calcula a participação percentual da quantidade de produtos vendidos pelo canal Loja
--      em relação ao total de produtos vendidos, respeitando o contexto de filtros do relatório.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Divide a quantidade de produtos vendidos no canal Loja pela quantidade total
--      de produtos vendidos, resultando na representatividade percentual desse canal.
--
-- Dependências:
--      [quantidade_produtos_vendidos_total]
--      [quantidade_produtos_vendidos_loja]
--
-- Retorno:
--      Valor numérico representando a participação percentual da Loja (entre 0 e 1).
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      A medida pode ser formatada como percentual nos visuais do Power BI
--      para melhor interpretação pelo usuário.

VAR _QuantidadeProdutosVendidosTotal =
    [quantidade_produtos_vendidos_total]

VAR _QuantidadeProdutosVendidosLoja =
    [quantidade_produtos_vendidos_loja]

VAR _Resultado =
    DIVIDE(
        _QuantidadeProdutosVendidosLoja,
        _QuantidadeProdutosVendidosTotal
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
percentual_quantidade_produtos_vendidos_ecommerce = 

-- Medida:
--      percentual_quantidade_produtos_vendidos_ecommerce
--
-- Descrição:
--      Calcula a participação percentual da quantidade de produtos vendidos pelo canal e-Commerce
--      em relação ao total de produtos vendidos, respeitando o contexto de filtros do relatório.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Divide a quantidade de produtos vendidos no canal e-Commerce pela quantidade total
--      de produtos vendidos, resultando na representatividade percentual desse canal.
--
-- Dependências:
--      [quantidade_produtos_vendidos_total]
--      [quantidade_produtos_vendidos_ecommerce]
--
-- Retorno:
--      Valor numérico representando a participação percentual do e-Commerce (entre 0 e 1).
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      A medida pode ser formatada como percentual nos visuais do Power BI
--      para melhor interpretação pelo usuário.

VAR _QuantidadeProdutosVendidosTotal =
    [quantidade_produtos_vendidos_total]

VAR _QuantidadeProdutosVendidosEcommerce =
    [quantidade_produtos_vendidos_ecommerce]

VAR _Resultado =
    DIVIDE(
        _QuantidadeProdutosVendidosEcommerce,
        _QuantidadeProdutosVendidosTotal
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
percentual_variacao_produtos_vendidos_loja_ecommerce = 

-- Medida:
--      percentual_variacao_produtos_vendidos_loja_ecommerce
--
-- Descrição:
--      Calcula a variação percentual da quantidade de produtos vendidos entre os canais
--      Loja e e-Commerce, utilizando o canal Loja como base de comparação.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Subtrai a quantidade de produtos vendidos no e-Commerce da quantidade vendida na Loja
--      e divide pelo total da Loja, resultando na variação percentual relativa ao canal Loja.
--
--      Interpretação:
--
--          Valor positivo:
--              Loja vendeu mais produtos que o e-Commerce
--
--          Valor negativo:
--              e-Commerce vendeu mais produtos que a Loja
--
--          Valor próximo de zero:
--              Canais com volume semelhante
--
-- Dependências:
--      [quantidade_produtos_vendidos_loja]
--      [quantidade_produtos_vendidos_ecommerce]
--
-- Retorno:
--      Valor numérico representando a variação percentual entre os canais (entre -∞ e +∞).
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--
--      O resultado pode ser formatado como percentual para melhor interpretação.
--
--      Importante: a escolha do canal Loja como base influencia a leitura da métrica.
--      Caso necessário, pode-se criar uma versão alternativa utilizando o e-Commerce como base.

VAR _Resultado =
    DIVIDE(
        [quantidade_produtos_vendidos_loja] - [quantidade_produtos_vendidos_ecommerce],
        [quantidade_produtos_vendidos_loja]
    )

RETURN
    _Resultado
```

## Medidas de Ticket Médio
<br>

```DAX
ticket_medio_transacao_total = 

-- Medida:
--      ticket_medio_transacao_total
--
-- Descrição:
--      Calcula o ticket médio por transação considerando todos os canais,
--      representando o valor médio de cada venda realizada no total.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas)
--
-- Regra de negócio:
--      Divide o faturamento total pela quantidade total de transações,
--      retornando o valor médio por venda (pedido) no contexto analisado.
--
-- Dependências:
--      [faturamento_total]
--      [transacoes_total]
--
-- Retorno:
--      Valor numérico representando o ticket médio por transação geral.
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      Importante: esta medida considera o comportamento agregado de compra,
--      sendo diferente do ticket médio por produto (valor por unidade).
--
--      Permite análise comparativa entre canais e acompanhamento da performance
--      geral das vendas.

VAR _Resultado =
    DIVIDE(
        [faturamento_total],
        [transacoes_total]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
ticket_medio_transacao_loja = 

-- Medida:
--      ticket_medio_transacao_loja
--
-- Descrição:
--      Calcula o ticket médio por transação no canal Loja,
--      representando o valor médio de cada venda realizada neste canal.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Divide o faturamento total do canal Loja pela quantidade
--      de transações realizadas nesse canal.
--
--      O resultado representa o valor médio por venda (pedido) na loja.
--
-- Dependências:
--      [faturamento_loja]
--      [transacoes_loja]
--
-- Retorno:
--      Valor numérico representando o ticket médio por transação no canal Loja.
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      Importante: esta medida representa o ticket médio por venda (pedido),
--      sendo diferente do ticket médio por produto (valor por unidade).
--
--      Pode refletir comportamento de compra na loja, como quantidade de itens
--      por transação e valor agregado por pedido.

VAR _Resultado =
    DIVIDE(
        [faturamento_loja],
        [transacoes_loja]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
ticket_medio_transacao_ecommerce = 

-- Medida:
--      ticket_medio_transacao_ecommerce
--
-- Descrição:
--      Calcula o ticket médio por transação no canal e-Commerce,
--      representando o valor médio de cada venda realizada.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Divide o faturamento total do canal e-Commerce pela quantidade
--      de transações realizadas nesse canal.
--
--      O resultado representa o valor médio por venda (pedido).
--
-- Dependências:
--      [faturamento_ecommerce]
--      [transacoes_ecommerce]
--
-- Retorno:
--      Valor numérico representando o ticket médio por transação no canal e-Commerce.
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      Importante: esta medida representa o ticket médio por venda (pedido),
--      sendo diferente do ticket médio por produto (valor por unidade).
--
--      Pode refletir comportamento de compra, como quantidade de itens por pedido
--      e valor agregado por transação.

VAR _Resultado =
    DIVIDE(
        [faturamento_ecommerce],
        [transacoes_ecommerce]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
ticket_medio_produto_total = 

-- Medida:
--      ticket_medio_produto_total
--
-- Descrição:
--      Calcula o ticket médio geral por produto, representando o valor médio
--      de venda por unidade comercializada considerando todos os canais.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas)
--
-- Regra de negócio:
--      Divide o faturamento total pela quantidade total de produtos vendidos,
--      resultando no valor médio por unidade no contexto analisado.
--
-- Dependências:
--      [faturamento_total]
--      [quantidade_produtos_vendidos_total]
--
-- Retorno:
--      Valor numérico representando o ticket médio geral por produto.
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      Importante: esta medida representa o preço médio efetivo de venda,
--      podendo refletir descontos, variações de preço, mix de produtos e diferenças entre canais.

VAR _Resultado =
    DIVIDE(
        [faturamento_total],
        [quantidade_produtos_vendidos_total]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
ticket_medio_produto_loja = 

-- Medida:
--      ticket_medio_produto_loja
--
-- Descrição:
--      Calcula o ticket médio por produto no canal Loja,
--      representando o valor médio de venda por unidade comercializada.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Divide o faturamento total do canal Loja pela quantidade total
--      de produtos vendidos nesse canal.
--
--      O resultado representa o valor médio por unidade vendida.
--
-- Dependências:
--      [faturamento_loja]
--      [quantidade_produtos_vendidos_loja]
--
-- Retorno:
--      Valor numérico representando o ticket médio por produto no canal Loja.
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      Importante: esta medida representa o preço médio efetivo de venda,
--      podendo refletir descontos, variações de preço ou mix de produtos.

VAR _Resultado =
    DIVIDE(
        [faturamento_loja],
        [quantidade_produtos_vendidos_loja]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
ticket_medio_produto_ecommerce = 

-- Medida:
--      ticket_medio_produto_ecommerce
--
-- Descrição:
--      Calcula o ticket médio por produto no canal e-Commerce,
--      representando o valor médio de venda por unidade comercializada.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Divide o faturamento total do canal e-Commerce pela quantidade total
--      de produtos vendidos nesse canal.
--
--      O resultado representa o valor médio por unidade vendida.
--
-- Dependências:
--      [faturamento_ecommerce]
--      [quantidade_produtos_vendidos_ecommerce]
--
-- Retorno:
--      Valor numérico representando o ticket médio por produto no canal e-Commerce.
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      Importante: esta medida representa o preço médio efetivo de venda,
--      podendo refletir descontos, variações de preço ou mix de produtos.

VAR _Resultado =
    DIVIDE(
        [faturamento_ecommerce],
        [quantidade_produtos_vendidos_ecommerce]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
ticket_medio_vendedor_total = 

-- Medida:
--      ticket_medio_vendedor_total
--
-- Descrição:
--      Calcula o ticket médio por vendedor considerando todos os canais,
--      representando o valor médio faturado por cada vendedor no total.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas)
--
-- Regra de negócio:
--      Divide o faturamento total pela quantidade distinta de vendedores
--      que realizaram vendas no período ou contexto selecionado.
--
--      O resultado representa o ticket médio por vendedor agregado,
--      considerando o desempenho global na organização.
--
-- Dependências:
--      [faturamento_total]
--      fato_vendas[id_vendedor]
--
-- Retorno:
--      Valor numérico representando o ticket médio por vendedor total.
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      Permite analisar a performance média global de todos os vendedores,
--      útil para comparações, bônus, e decisões estratégicas de gestão de equipe.

VAR _Resultado =
    DIVIDE(
        [faturamento_total],
        DISTINCTCOUNT(
            fato_vendas[id_vendedor]
        )
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
ticket_medio_vendedor_loja = 

-- Medida:
--      ticket_medio_vendedor_loja
--
-- Descrição:
--      Calcula o ticket médio por vendedor no canal Loja,
--      representando o valor médio faturado por cada vendedor nesse canal.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Divide o faturamento total do canal Loja pela quantidade
--      distinta de vendedores que realizaram vendas nesse canal.
--
--      O resultado representa o ticket médio por vendedor (valor médio gerado por cada vendedor).
--
-- Dependências:
--      [faturamento_loja]
--      fato_vendas[id_vendedor]
--
-- Retorno:
--      Valor numérico representando o ticket médio por vendedor no canal Loja.
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      Permite analisar a performance média de cada vendedor no canal Loja,
--      útil para comparações, bonificações e gestão de equipe.

VAR _Resultado =
    DIVIDE(
        [faturamento_loja],
        CALCULATE(
            DISTINCTCOUNT(
                fato_vendas[id_vendedor]
            ),
            fato_vendas[id_canal]="1"
        )
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
ticket_medio_vendedor_ecommerce = 

-- Medida:
--      ticket_medio_vendedor_ecommerce
--
-- Descrição:
--      Calcula o ticket médio por vendedor no canal e-Commerce,
--      representando o valor médio faturado por cada vendedor nesse canal.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Divide o faturamento total do canal e-Commerce pela quantidade
--      distinta de vendedores que realizaram vendas nesse canal.
--
--      O resultado representa o ticket médio por vendedor (valor médio gerado por cada vendedor).
--
-- Dependências:
--      [faturamento_ecommerce]
--      fato_vendas[id_vendedor]
--
-- Retorno:
--      Valor numérico representando o ticket médio por vendedor no canal e-Commerce.
--
-- Observação:
--      A função DIVIDE é utilizada para evitar erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      Permite analisar a performance média de cada vendedor no canal e-Commerce,
--      útil para comparações, bonificações e gestão de equipe.

VAR _Resultado =
    DIVIDE(
        [faturamento_ecommerce],
        CALCULATE(
            DISTINCTCOUNT(
                fato_vendas[id_vendedor]
            ),
            fato_vendas[id_canal]="2"
        )
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
ticket_medio_localidade_total = 

-- Medida:
--      ticket_medio_localidade_total
--
-- Descrição:
--      Calcula o ticket médio geral por localidade (UF), considerando o faturamento total
--      dividido pela quantidade de localidades distintas com vendas.
--
-- Tabela origem:
--      fato_vendas
--
-- Regra de negócio:
--      Divide o faturamento total pela quantidade distinta de UFs presentes
--      no contexto atual do relatório.
--
-- Dependências:
--      [faturamento_total]
--      fato_vendas[id_uf]
--
-- Retorno:
--      Valor numérico representando o ticket médio por localidade no contexto analisado.
--
-- Observação:
--      A função DISTINCTCOUNT é utilizada para contar a quantidade de UFs distintas.
--
--      A função DIVIDE evita erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      Importante: esta medida representa o ticket médio por localidade,
--      e não por cliente, pedido ou produto.

VAR _Resultado =
    DIVIDE(
        [faturamento_total],
        DISTINCTCOUNT(
            fato_vendas[id_uf]
        )
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
ticket_medio_localidade_loja = 

-- Medida:
--      ticket_medio_localidade_loja
--
-- Descrição:
--      Calcula o ticket médio do canal Loja por localidade (UF),
--      considerando o faturamento total dividido pela quantidade de localidades atendidas.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Divide o faturamento do canal Loja pela quantidade distinta de localidades (UF)
--      que realizaram vendas nesse canal.
--
--      O cálculo considera apenas registros do canal Loja.
--
-- Dependências:
--      [faturamento_loja]
--      fato_vendas[id_uf]
--      fato_vendas[id_canal]
--
-- Retorno:
--      Valor numérico representando o ticket médio por localidade no canal Loja.
--
-- Observação:
--      A função DISTINCTCOUNT é utilizada para contar a quantidade de UFs distintas.
--
--      A função CALCULATE aplica o filtro do canal Loja para garantir consistência
--      entre numerador e denominador.
--
--      A função DIVIDE evita erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      Importante: esta medida representa o ticket médio por localidade,
--      e não por cliente ou por venda.

VAR _Resultado =
    DIVIDE(
        [faturamento_loja],
        CALCULATE(
            DISTINCTCOUNT(
                fato_vendas[id_uf]
            ),
            fato_vendas[id_canal] = "1"
        )
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
ticket_medio_localidade_ecommerce = 

-- Medida:
--      ticket_medio_localidade_ecommerce
--
-- Descrição:
--      Calcula o ticket médio do canal e-Commerce por localidade (UF),
--      considerando o faturamento total dividido pela quantidade de localidades atendidas.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_canais)
--
-- Regra de negócio:
--      Divide o faturamento do canal e-Commerce pela quantidade distinta de localidades (UF)
--      que realizaram vendas nesse canal.
--
--      O cálculo considera apenas registros do canal e-Commerce.
--
-- Dependências:
--      [faturamento_ecommerce]
--      fato_vendas[id_uf]
--      fato_vendas[id_canal]
--
-- Retorno:
--      Valor numérico representando o ticket médio por localidade no canal e-Commerce.
--
-- Observação:
--      A função DISTINCTCOUNT é utilizada para contar a quantidade de UFs distintas.
--
--      A função CALCULATE aplica o filtro do canal e-Commerce para garantir consistência
--      entre numerador e denominador.
--
--      A função DIVIDE evita erros de divisão por zero.
--      COALESCE garante retorno 0 quando o resultado for BLANK().
--
--      Importante: esta medida representa o ticket médio por localidade,
--      e não por cliente ou por venda.

VAR _Resultado =
    DIVIDE(
        [faturamento_ecommerce],
        CALCULATE(
            DISTINCTCOUNT(
                fato_vendas[id_uf]
            ),
            fato_vendas[id_canal] = "2"
        )
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```

## Medidas de Classificação
<br>

```DAX
media_faturamento_produto = 

-- Medida:
--      media_faturamento_produto
--
-- Descrição:
--      Calcula a média de faturamento por produto, considerando o desempenho médio entre
--      todos os produtos disponíveis no modelo, independentemente de filtros aplicados.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_produtos)
--
-- Regra de negócio:
--      A média é calculada a partir do faturamento total de cada produto distinto, garantindo que
--      cada produto contribua igualmente para o resultado final (média de faturamento por produto).
--
--      O cálculo ignora filtros aplicados na dimensão de produtos, permitindo uma referência global
--      para comparação com o desempenho individual de cada item.
--
-- Dependências:
--      [faturamento_total]
--      dimensao_produtos[nome_produto]
--
-- Retorno:
--      Valor numérico representando a média de faturamento entre os produtos.
--
-- Observação:
--      A função VALUES garante a avaliação no contexto dos produtos distintos.
--      O uso de AVERAGEX permite iterar sobre os produtos e calcular a média do faturamento total.
--      REMOVEFILTERS é aplicado para desconsiderar filtros de produto e garantir uma média global consistente.

VAR _Resultado =
    CALCULATE(
        AVERAGEX(
            VALUES(
                dimensao_produtos[nome_produto]
            ),
            [faturamento_total]
        ),
        REMOVEFILTERS(
            dimensao_produtos[nome_produto]
        )
    )

RETURN
    _Resultado
```
<br>

```DAX
media_faturamento_vendedor = 

-- Medida:
--      media_faturamento_vendedor
--
-- Descrição:
--      Calcula a média de faturamento por vendedor, considerando o desempenho médio entre
--      todos os vendedores disponíveis no modelo, independentemente de filtros aplicados.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_vendedores)
--
-- Regra de negócio:
--      A média é calculada a partir do faturamento total de cada vendedor distinto, garantindo que
--      cada vendedor contribua igualmente para o resultado final (média de faturamento por vendedor).
--
--      O cálculo ignora filtros aplicados na dimensão de vendedores, permitindo uma referência global
--      para comparação com o desempenho individual de cada vendedor.
--
-- Dependências:
--      [faturamento_total]
--      dimensao_vendedores[nome_vendedor]
--
-- Retorno:
--      Valor numérico representando a média de faturamento entre os vendedores.
--
-- Observação:
--      A função VALUES garante a avaliação no contexto dos vendedores distintos.
--      O uso de AVERAGEX permite iterar sobre os vendedores e calcular a média do faturamento total.
--      REMOVEFILTERS é aplicado para desconsiderar filtros de vendedor e garantir uma média global consistente.

VAR _Resultado =
    CALCULATE(
        AVERAGEX(
            VALUES(
                dimensao_vendedores[nome_vendedor]
            ),
            [faturamento_total]
        ),
        REMOVEFILTERS(
            dimensao_vendedores[nome_vendedor]
        )
    )

RETURN
    _Resultado
```
<br>

```DAX
media_faturamento_localidade = 

-- Medida:
--      media_faturamento_localidade
--
-- Descrição:
--      Calcula a média de faturamento por localidade (UF), considerando o desempenho médio entre
--      todas as localidades disponíveis no modelo, independentemente de filtros de estado aplicados.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas, dimensao_vendedores e dimensao_estados_brasileiros)
--
-- Regra de negócio:
--      A média é calculada a partir do faturamento total de cada UF distinta, garantindo que cada
--      localidade contribua igualmente para o resultado final (média de médias por UF).
--
--      O cálculo ignora filtros aplicados na dimensão de estados, permitindo uma referência global
--      para comparação com o desempenho individual de cada localidade.
--
-- Dependências:
--      [faturamento_total]
--      dimensao_vendedores[uf]
--      dimensao_estados_brasileiros[nome_estado]
--
-- Retorno:
--      Valor numérico representando a média de faturamento entre as localidades (UFs).
--
-- Observação:
--      A função DISTINCT garante que cada UF seja considerada apenas uma vez no cálculo.
--      O uso de AVERAGEX permite iterar sobre as UFs e calcular a média do faturamento total.
--      REMOVEFILTERS é aplicado para desconsiderar filtros de estado e garantir uma média global consistente.

VAR _Resultado =
    CALCULATE(
        AVERAGEX(
            DISTINCT(
                dimensao_vendedores[uf]
            ),
            [faturamento_total]
        ),
        REMOVEFILTERS(
            dimensao_estados_brasileiros[nome_estado]
        )
    )

RETURN
    _Resultado
```
<br>

```DAX
media_quantidade_produtos_vedidos_produto = 

-- Medida:
--      media_quantidade_produtos_vedidos_produto
--
-- Descrição:
--      Calcula a média da quantidade de produtos vendidos por produto, considerando o desempenho médio
--      entre todos os produtos disponíveis no modelo, independentemente de filtros aplicados.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_produtos)
--
-- Regra de negócio:
--      A média é calculada a partir da quantidade total de produtos vendidos por produto distinto,
--      garantindo que cada produto contribua igualmente para o resultado final
--      (média de quantidade vendida por produto).
--
--      O cálculo ignora filtros aplicados na dimensão de produtos, permitindo uma referência global
--      para comparação com o desempenho individual de cada item.
--
-- Dependências:
--      [quantidade_produtos_vendidos_total]
--      dimensao_produtos[nome_produto]
--
-- Retorno:
--      Valor numérico representando a média de quantidade de produtos vendidos entre os produtos.
--
-- Observação:
--      A função VALUES garante a avaliação no contexto dos produtos distintos.
--      O uso de AVERAGEX permite iterar sobre os produtos e calcular a média da quantidade vendida.
--      REMOVEFILTERS é aplicado para desconsiderar filtros de produto e garantir uma média global consistente.

VAR _Resultado =
    CALCULATE(
        AVERAGEX(
            VALUES(
                dimensao_produtos[nome_produto]
            ),
            [quantidade_produtos_vendidos_total]
        ),
        REMOVEFILTERS(
            dimensao_produtos[nome_produto]
        )
    )

RETURN
    _Resultado
```
<br>

```DAX
media_quantidade_produtos_vedidos_vendedor = 

-- Medida:
--      media_quantidade_produtos_vedidos_vendedor
--
-- Descrição:
--      Calcula a média da quantidade de produtos vendidos por vendedor, considerando o desempenho médio
--      entre todos os vendedores disponíveis no modelo, independentemente de filtros aplicados.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e dimensao_vendedores)
--
-- Regra de negócio:
--      A média é calculada a partir da quantidade total de produtos vendidos por vendedor distinto,
--      garantindo que cada vendedor contribua igualmente para o resultado final
--      (média de quantidade vendida por vendedor).
--
--      O cálculo ignora filtros aplicados na dimensão de vendedores, permitindo uma referência global
--      para comparação com o desempenho individual de cada vendedor.
--
-- Dependências:
--      [quantidade_produtos_vendidos_total]
--      dimensao_vendedores[nome_vendedor]
--
-- Retorno:
--      Valor numérico representando a média de quantidade de produtos vendidos entre os vendedores.
--
-- Observação:
--      A função VALUES garante a avaliação no contexto dos vendedores distintos.
--      O uso de AVERAGEX permite iterar sobre os vendedores e calcular a média da quantidade vendida.
--      REMOVEFILTERS é aplicado para desconsiderar filtros de vendedor e garantir uma média global consistente.

VAR _Resultado =
    CALCULATE(
        AVERAGEX(
            VALUES(
                dimensao_vendedores[nome_vendedor]
            ),
            [quantidade_produtos_vendidos_total]
        ),
        REMOVEFILTERS(
            dimensao_vendedores[nome_vendedor]
        )
    )

RETURN
    _Resultado
```
<br>

```DAX
media_quantidade_produtos_vedidos_localidade = 

-- Medida:
--      media_quantidade_produtos_vedidos_localidade
--
-- Descrição:
--      Calcula a média da quantidade de produtos vendidos por localidade (UF),
--      considerando o desempenho médio entre todas as localidades disponíveis no modelo.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas, dimensao_vendedores e dimensao_estados_brasileiros)
--
-- Regra de negócio:
--      A média é calculada a partir da quantidade total de produtos vendidos em cada UF distinta,
--      garantindo que cada localidade contribua igualmente para o resultado final
--      (média de quantidade vendida por UF).
--
--      O cálculo ignora filtros aplicados na dimensão de estados, permitindo uma referência global
--      para comparação com o desempenho individual de cada localidade.
--
-- Dependências:
--      [quantidade_produtos_vendidos_total]
--      dimensao_vendedores[uf]
--      dimensao_estados_brasileiros[nome_estado]
--
-- Retorno:
--      Valor numérico representando a média de quantidade de produtos vendidos entre as localidades.
--
-- Observação:
--      A função DISTINCT garante que cada UF seja considerada apenas uma vez no cálculo.
--      O uso de AVERAGEX permite iterar sobre as UFs e calcular a média da quantidade vendida.
--      REMOVEFILTERS é aplicado para desconsiderar filtros de estado e garantir uma média global consistente.

VAR _Resultado =
    CALCULATE(
        AVERAGEX(
            DISTINCT(
                dimensao_vendedores[uf]
            ),
            [quantidade_produtos_vendidos_total]
        ),
        REMOVEFILTERS(
            dimensao_estados_brasileiros[nome_estado]
        )
    )

RETURN
    _Resultado
```
<br>

```DAX
classificacao_produto = 

-- Medida:
--      classificacao_produto
--
-- Descrição:
--      Classifica os produtos com base no desempenho de faturamento e quantidade vendida,
--      utilizando como referência as médias dessas métricas no contexto analisado.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e tabelas relacionadas)
--
-- Regra de negócio:
--      Compara o faturamento e a quantidade de produtos vendidos de cada produto com suas respectivas médias,
--      classificando em quatro categorias estratégicas:
--
--          ⭐ Estrela:
--              Faturamento acima da média e quantidade acima da média
--
--          🐄 Vaca Leiteira:
--              Faturamento acima da média e quantidade abaixo da média
--
--          ❓ Interrogação:
--              Faturamento abaixo da média e quantidade acima da média
--
--          🍍 Abacaxi:
--              Faturamento abaixo da média e quantidade abaixo da média
--
-- Dependências:
--      [faturamento_total]
--      [quantidade_produtos_vendidos_total]
--      [media_faturamento_produto]
--      [media_quantidade_produtos_vedidos_produto]
--
-- Retorno:
--      Texto representando a classificação estratégica do produto no contexto filtrado.
--
-- Observação:
--      A lógica utiliza SWITCH(TRUE()) para avaliação condicional sequencial.
--      Caso nenhuma condição seja atendida, retorna BLANK().

VAR _Faturamento =
    [faturamento_total]

VAR _QuantidadeProdutosVendidos =
    [quantidade_produtos_vendidos_total]

VAR _MediaFaturamento =
    [media_faturamento_produto]

VAR _MediaQuantidadeProdutosVendidos =
    [media_quantidade_produtos_vedidos_produto]

RETURN
    SWITCH(
        TRUE(),

        _QuantidadeProdutosVendidos >= _MediaQuantidadeProdutosVendidos && _Faturamento >= _MediaFaturamento, "⭐ Estrela (↑Faturamento ↑Quantidade)",
        _QuantidadeProdutosVendidos < _MediaQuantidadeProdutosVendidos && _Faturamento >= _MediaFaturamento, "🐄 Vaca Leiteira (↑Faturamento ↓Quantidade)",
        _QuantidadeProdutosVendidos >= _MediaQuantidadeProdutosVendidos && _Faturamento < _MediaFaturamento, "❓ Interrogação (↓Faturamento ↑Quantidade)",
        _QuantidadeProdutosVendidos < _MediaQuantidadeProdutosVendidos && _Faturamento < _MediaFaturamento, "🍍 Abacaxi (↓Faturamento ↓Quantidade)",
        BLANK()
    )
```
<br>

```DAX
classificacao_vendedor = 

-- Medida:
--      classificacao_vendedor
--
-- Descrição:
--      Classifica os vendedores com base no desempenho de faturamento e quantidade de produtos vendidos,
--      utilizando como referência as médias dessas métricas no contexto analisado.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e tabelas relacionadas)
--
-- Regra de negócio:
--      Compara o faturamento e a quantidade de produtos vendidos de cada vendedor com suas respectivas médias,
--      classificando em quatro categorias estratégicas:
--
--          ⭐ Estrela:
--              Faturamento acima da média e quantidade acima da média
--
--          🐄 Vaca Leiteira:
--              Faturamento acima da média e quantidade abaixo da média
--
--          ❓ Interrogação:
--              Faturamento abaixo da média e quantidade acima da média
--
--          🍍 Abacaxi:
--              Faturamento abaixo da média e quantidade abaixo da média
--
-- Dependências:
--      [faturamento_total]
--      [quantidade_produtos_vendidos_total]
--      [media_faturamento_vendedor]
--      [media_quantidade_produtos_vedidos_vendedor]
--
-- Retorno:
--      Texto representando a classificação estratégica do vendedor no contexto filtrado.
--
-- Observação:
--      A lógica utiliza SWITCH(TRUE()) para avaliação condicional sequencial.
--      Caso nenhuma condição seja atendida, retorna BLANK().

VAR _Faturamento =
    [faturamento_total]

VAR _QuantidadeProdutosVendidos =
    [quantidade_produtos_vendidos_total]

VAR _MediaFaturamento =
    [media_faturamento_vendedor]

VAR _MediaQuantidadeProdutosVendidos =
    [media_quantidade_produtos_vedidos_vendedor]

RETURN
    SWITCH(
        TRUE(),

        _QuantidadeProdutosVendidos >= _MediaQuantidadeProdutosVendidos && _Faturamento >= _MediaFaturamento, "⭐ Estrela (↑Faturamento ↑Quantidade)",
        _QuantidadeProdutosVendidos < _MediaQuantidadeProdutosVendidos && _Faturamento >= _MediaFaturamento, "🐄 Vaca Leiteira (↑Faturamento ↓Quantidade)",
        _QuantidadeProdutosVendidos >= _MediaQuantidadeProdutosVendidos && _Faturamento < _MediaFaturamento, "❓ Interrogação (↓Faturamento ↑Quantidade)",
        _QuantidadeProdutosVendidos < _MediaQuantidadeProdutosVendidos && _Faturamento < _MediaFaturamento, "🍍 Abacaxi (↓Faturamento ↓Quantidade)",
        BLANK()
    )
```
<br>

```DAX
classificacao_localidade = 

-- Medida:
--      classificacao_localidade
--
-- Descrição:
--      Classifica as localidades com base no desempenho de faturamento e quantidade de produtos vendidos,
--      utilizando como referência as médias dessas métricas no contexto analisado.
--
-- Tabela origem:
--      Medida calculada (baseada em fVendas e tabelas relacionadas)
--
-- Regra de negócio:
--      Compara o faturamento e a quantidade de produtos vendidos da localidade com suas respectivas médias,
--      classificando em quatro categorias estratégicas:
--
--          ⭐ Estrela:
--              Faturamento acima da média e quantidade acima da média
--
--          🐄 Vaca Leiteira:
--              Faturamento acima da média e quantidade abaixo da média
--
--          ❓ Interrogação:
--              Faturamento abaixo da média e quantidade acima da média
--
--          🍍 Abacaxi:
--              Faturamento abaixo da média e quantidade abaixo da média
--
-- Dependências:
--      [faturamento_total]
--      [quantidade_produtos_vendidos_total]
--      [media_faturamento_localidade]
--      [media_quantidade_produtos_vedidos_localidade]
--
-- Retorno:
--      Texto representando a classificação estratégica da localidade no contexto filtrado.
--
-- Observação:
--      A lógica utiliza SWITCH(TRUE()) para avaliação condicional sequencial.
--      Caso nenhuma condição seja atendida, retorna BLANK().

VAR _Faturamento =
    [faturamento_total]

VAR _QuantidadeProdutosVendidos =
    [quantidade_produtos_vendidos_total]

VAR _MediaFaturamento =
    [media_faturamento_localidade]

VAR _MediaQuantidadeProdutosVendidos =
    [media_quantidade_produtos_vedidos_localidade]

RETURN
    SWITCH(
        TRUE(),

        _QuantidadeProdutosVendidos >= _MediaQuantidadeProdutosVendidos && _Faturamento >= _MediaFaturamento, "⭐ Estrela (↑Faturamento ↑Quantidade)",
        _QuantidadeProdutosVendidos < _MediaQuantidadeProdutosVendidos && _Faturamento >= _MediaFaturamento, "🐄 Vaca Leiteira (↑Faturamento ↓Quantidade)",
        _QuantidadeProdutosVendidos >= _MediaQuantidadeProdutosVendidos && _Faturamento < _MediaFaturamento, "❓ Interrogação (↓Faturamento ↑Quantidade)",
        _QuantidadeProdutosVendidos < _MediaQuantidadeProdutosVendidos && _Faturamento < _MediaFaturamento, "🍍 Abacaxi (↓Faturamento ↓Quantidade)",
        BLANK()
    )
```
<br>

```DAX
cores_classificacao_produtos = 

-- Medida:
--      cores_classificacao_produtos
--
-- Descrição:
--      Define cores hexadecimais associadas a cada classificação de produto,
--      permitindo a aplicação de formatação condicional em visuais do Power BI.
--
-- Tabela origem:
--      Medida calculada (baseada na medida [classificacao_produto])
--
-- Regra de negócio:
--      Atribui uma cor específica para cada categoria estratégica de produto:
--
--          ⭐ Estrela:
--              Verde (#00C853)
--
--          🐄 Vaca Leiteira:
--              Azul (#2962FF)
--
--          ❓ Interrogação:
--              Laranja (#FF8F00)
--
--          🍍 Abacaxi:
--              Vermelho (#D50000)
--
-- Dependências:
--      [classificacao_produto]
--
-- Retorno:
--      Código de cor em formato hexadecimal (texto), utilizado para formatação condicional.
--
-- Observação:
--      Caso a classificação não corresponda a nenhuma das categorias definidas,
--      o resultado será BLANK(), podendo impactar visuais que dependem de cor.

SWITCH(
    [classificacao_produto],

    "⭐ Estrela (↑Faturamento ↑Quantidade)", "#00C853",
    "🐄 Vaca Leiteira (↑Faturamento ↓Quantidade)", "#2962FF",
    "❓ Interrogação (↓Faturamento ↑Quantidade)", "#FF8F00",
    "🍍 Abacaxi (↓Faturamento ↓Quantidade)", "#D50000"

)
```
<br>

```DAX
cores_classificacao_vendedores = 

-- Medida:
--      cores_classificacao_vendedores
--
-- Descrição:
--      Define cores hexadecimais associadas a cada classificação de vendedor,
--      permitindo a aplicação de formatação condicional em visuais do Power BI.
--
-- Tabela origem:
--      Medida calculada (baseada na medida [classificacao_vendedor])
--
-- Regra de negócio:
--      Atribui uma cor específica para cada categoria estratégica de vendedor:
--
--          ⭐ Estrela:
--              Verde (#00C853)
--
--          🐄 Vaca Leiteira:
--              Azul (#2962FF)
--
--          ❓ Interrogação:
--              Laranja (#FF8F00)
--
--          🍍 Abacaxi:
--              Vermelho (#D50000)
--
-- Dependências:
--      [classificacao_vendedor]
--
-- Retorno:
--      Código de cor em formato hexadecimal (texto), utilizado para formatação condicional.
--
-- Observação:
--      Caso a classificação não corresponda a nenhuma das categorias definidas,
--      o resultado será BLANK(), podendo impactar visuais que dependem de cor.

SWITCH(
    [classificacao_vendedor],

    "⭐ Estrela (↑Faturamento ↑Quantidade)", "#00C853",
    "🐄 Vaca Leiteira (↑Faturamento ↓Quantidade)", "#2962FF",
    "❓ Interrogação (↓Faturamento ↑Quantidade)", "#FF8F00",
    "🍍 Abacaxi (↓Faturamento ↓Quantidade)", "#D50000"

)
```
<br>

```DAX
cores_classificacao_localidade = 

-- Medida:
--      cores_classificacao_localidade
--
-- Descrição:
--      Define cores hexadecimais associadas a cada classificação de localidade,
--      permitindo a aplicação de formatação condicional em visuais do Power BI.
--
-- Tabela origem:
--      Medida calculada (baseada na medida [classificacao_localidade])
--
-- Regra de negócio:
--      Atribui uma cor específica para cada categoria estratégica de localidade:
--
--          ⭐ Estrela:
--              Verde (#00C853)
--
--          🐄 Vaca Leiteira:
--              Azul (#2962FF)
--
--          ❓ Interrogação:
--              Laranja (#FF8F00)
--
--          🍍 Abacaxi:
--              Vermelho (#D50000)
--
-- Dependências:
--      [classificacao_localidade]
--
-- Retorno:
--      Código de cor em formato hexadecimal (texto), utilizado para formatação condicional.
--
-- Observação:
--      Caso a classificação não corresponda a nenhuma das categorias definidas,
--      o resultado será BLANK(), podendo impactar visuais que dependem de cor.

SWITCH(
    [classificacao_localidade],

    "⭐ Estrela (↑Faturamento ↑Quantidade)", "#00C853",
    "🐄 Vaca Leiteira (↑Faturamento ↓Quantidade)", "#2962FF",
    "❓ Interrogação (↓Faturamento ↑Quantidade)", "#FF8F00",
    "🍍 Abacaxi (↓Faturamento ↓Quantidade)", "#D50000"

)
```
<br>

```DAX
nome_produto = 

-- Medida:
--      nome_produto
--
-- Descrição:
--      Retorna o nome do produto selecionado no contexto atual do relatório,
--      sendo útil para exibição dinâmica em títulos, cartões e elementos de interface.
--
-- Tabela origem:
--      dimensao_produtos
--
-- Regra de negócio:
--      Recupera o valor único da coluna de nome do produto no contexto filtrado.
--      Caso haja mais de um produto selecionado ou nenhum, o resultado será BLANK().
--
-- Dependências:
--      dimensao_produtos[nome_produto]
--
-- Retorno:
--      Texto contendo o nome do produto selecionado.
--
-- Observação:
--      A função SELECTEDVALUE retorna o valor quando há apenas um no contexto;
--      caso contrário, retorna BLANK(), evitando ambiguidade em cenários com múltiplas seleções.

VAR _Resultado =
    SELECTEDVALUE(
        dimensao_produtos[nome_produto]
    )

RETURN
    _Resultado
```
<br>

```DAX
nome_vendedor = 

-- Medida:
--      nome_vendedor
--
-- Descrição:
--      Retorna o nome do vendedor selecionado no contexto atual do relatório,
--      sendo útil para exibição dinâmica em títulos, cartões e elementos de interface.
--
-- Tabela origem:
--      dimensao_vendedores
--
-- Regra de negócio:
--      Recupera o valor único da coluna de nome do vendedor no contexto filtrado.
--      Caso haja mais de um vendedor selecionado ou nenhum, o resultado será BLANK().
--
-- Dependências:
--      dimensao_vendedores[nome_vendedor]
--
-- Retorno:
--      Texto contendo o nome do vendedor selecionado.
--
-- Observação:
--      A função SELECTEDVALUE retorna o valor quando há apenas um no contexto;
--      caso contrário, retorna BLANK(), evitando ambiguidade em cenários com múltiplas seleções.

VAR _Resultado =
    SELECTEDVALUE(
        dimensao_vendedores[nome_vendedor]
    )

RETURN
    _Resultado
```
<br>

```DAX
nome_gerente = 

-- Medida:
--      nome_gerente
--
-- Descrição:
--      Retorna o nome do gerente associado ao contexto de estado selecionado,
--      utilizando um relacionamento alternativo entre as tabelas de estados e vendedores.
--
-- Tabela origem:
--      dimensao_vendedores (com suporte da dimensao_estados_brasileiros)
--
-- Regra de negócio:
--      Recupera o nome do gerente correspondente à UF selecionada,
--      ativando explicitamente o relacionamento entre as tabelas de estados e vendedores.
--
--      Caso haja mais de um gerente no contexto ou nenhum, o resultado será BLANK().
--
-- Dependências:
--      dimensao_vendedores[nome_gerente]
--      dimensao_vendedores[uf]
--      dimensao_estados_brasileiros[uf]
--
-- Retorno:
--      Texto contendo o nome do gerente associado ao estado selecionado.
--
-- Observação:
--      A função USERELATIONSHIP é utilizada para ativar um relacionamento inativo
--      entre as tabelas dimensao_estados_brasileiros e dimensao_vendedores.
--
--      SELECTEDVALUE garante que apenas um único gerente seja retornado,
--      evitando ambiguidades em cenários com múltiplos valores no contexto.

VAR _Resultado =
    CALCULATE(
        SELECTEDVALUE(
            dimensao_vendedores[nome_gerente]
        ),
        USERELATIONSHIP(
            dimensao_estados_brasileiros[uf],
            dimensao_vendedores[uf]
        )
    )

RETURN
    _Resultado
```
<br>

```DAX
nome_estado = 

-- Medida:
--      nome_estado
--
-- Descrição:
--      Retorna o nome do estado selecionado no contexto atual do relatório,
--      sendo útil para exibição dinâmica em títulos, cartões e elementos de interface.
--
-- Tabela origem:
--      dimensao_estados_brasileiros
--
-- Regra de negócio:
--      Recupera o valor único da coluna de nome do estado no contexto filtrado.
--      Caso haja mais de um estado selecionado ou nenhum, o resultado será BLANK().
--
-- Dependências:
--      dimensao_estados_brasileiros[nome_estado]
--
-- Retorno:
--      Texto contendo o nome do estado selecionado.
--
-- Observação:
--      A função SELECTEDVALUE retorna o valor quando há apenas um no contexto;
--      caso contrário, retorna BLANK(), evitando ambiguidade em cenários com múltiplas seleções.

VAR _Resultado =
    SELECTEDVALUE(
        dimensao_estados_brasileiros[nome_estado]
    )

RETURN
    _Resultado
```
