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

```
<br>

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
<br>

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
<br>

## Medidas de Classificação
<br>

```DAX

```
<br>
