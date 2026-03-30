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

## Medidas de Devolução (Percentual)
<br>

```DAX
percentual_arrependimento = 

-- Medida:
--      percentual_arrependimento
--
-- Descrição:
--      Percentual de pedidos devolvidos por arrependimento em relação ao total de pedidos
--
-- Tabela origem:
--      fPedidos_Realizados
--
-- Regra de negócio:
--      Divide o número de pedidos devolvidos por arrependimento pela quantidade total de pedidos
--
-- Dependência:
--      [devolucao_arrependimento], [quantidade_pedidos]
--
-- Retorno:
--      Percentual (0 a 1) de pedidos devolvidos por arrependimento
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK() e divisão por zero

VAR _Resultado =
    DIVIDE(
        [devolucao_arrependimento],
        [quantidade_pedidos]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
percentual_produto_danificado = 

-- Medida:
--      percentual_produto_danificado
--
-- Descrição:
--      Percentual de pedidos devolvidos por motivo "Produto Danificado" em relação ao total de pedidos
--
-- Tabela origem:
--      fPedidos_Realizados
--
-- Regra de negócio:
--      Divide o número de pedidos devolvidos por dano pelo total de pedidos
--
-- Dependência:
--      [devolucao_produto_danificado], [quantidade_pedidos]
--
-- Retorno:
--      Percentual (0 a 1) de pedidos devolvidos por motivo Produto Danificado
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK() e divisão por zero

VAR _Resultado =
    DIVIDE(
        [devolucao_produto_danificado],
        [quantidade_pedidos]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
percentual_produto_errado = 

-- Medida:
--      percentual_produto_errado
--
-- Descrição:
--      Percentual de pedidos devolvidos por motivo "Produto Errado" em relação ao total de pedidos
--
-- Tabela origem:
--      fPedidos_Realizados
--
-- Regra de negócio:
--      Divide o número de pedidos devolvidos por motivo "Produto Errado" pelo total de pedidos
--
-- Dependência:
--      [devolucao_produto_errado], [quantidade_pedidos]
--
-- Retorno:
--      Percentual (0 a 1) de pedidos devolvidos por motivo Produto Errado
--
-- Observação:
--     COALESCE é utilizado para evitar retorno BLANK() e divisão por zero

VAR _Resultado =
    DIVIDE(
        [devolucao_produto_errado],
        [quantidade_pedidos]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
percentual_produtos_devolvidos = 

-- Medida:
--      percentual_produtos_devolvidos
--
-- Descrição:
--      Percentual de produtos devolvidos em relação ao total de produtos vendidos
--
-- Tabela origem:
--      fPedidos_Realizados
--
-- Regra de negócio:
--      Divide o total de produtos devolvidos pelo total de produtos vendidos
--
-- Dependência:
--      [produtos_devolvidos], [quantidade_produtos]
--
-- Retorno:
--      Percentual (0 a 1) de produtos devolvidos
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK() e divisão por zero

VAR _Resultado =
    DIVIDE(
        [produtos_devolvidos],
        [quantidade_produtos]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
percentual_sem_devolucao = 

-- Medida:
--      percentual_sem_devolucao
--
-- Descrição:
--      Percentual de pedidos sem devolução em relação ao total de pedidos
--
-- Tabela origem:
--      fPedidos_Realizados
--
-- Regra de negócio:
--      Divide o número de pedidos sem devolução pelo total de pedidos
--
-- Dependência:
--      [sem_devolução], [quantidade_pedidos]
--
-- Retorno:
--      Percentual (0 a 1) de pedidos sem devolução
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK() e divisão por zero

VAR _Resultado =
    DIVIDE(
        [sem_devolucao],
        [quantidade_pedidos]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```

## Medidas de Entregas
<br>

```DAX
entregas_prazo = 

-- Medida:
--      entregas_prazo
--
-- Descrição:
--      Quantidade de pedidos entregues dentro do prazo
--
-- Tabela origem:
--      fPedidos_Realizados
--
-- Regra de negócio:
--      Considera apenas registros onde status = "No Prazo"
--
-- Dependência:
--      [quantidade_pedidos]
--
-- Retorno:
--      Número inteiro de pedidos entregues no prazo
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK()

VAR _Resultado =
    CALCULATE(
        [quantidade_pedidos],
        fPedidos_Realizados[id_status]="2"
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
percentual_entregas_prazo = 

-- Medida:
--      percentual_entregas_prazo
--
-- Descrição:
--      Percentual de pedidos entregues dentro do prazo em relação ao total de pedidos
--
-- Tabela origem:
--      fPedidos_Realizados
--
-- Regra de negócio:
--      Divide o número de pedidos entregues no prazo pelo total de pedidos
--
-- Dependência:
--      [entregas_prazo], [quantidade_pedidos]
--
-- Retorno:
--      Percentual (0 a 1) de pedidos entregues dentro do prazo
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK() e divisão por zero

VAR _Resultado =
    DIVIDE(
        [entregas_prazo],
        [quantidade_pedidos]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```

## Medida de Faturamento
<br>

```DAX
faturamento = 

-- Medida:
--      faturamento
--
-- Descrição:
--      Soma do valor faturado de todos os pedidos
--
-- Tabela origem:
--      fPedidos_Realizados
--
-- Regra de negócio:
--      Soma simples da coluna [faturamento] da tabela fPedidos_Realizados
--
-- Dependência:
--      'fPedidos_Realizados'[faturamento]
--
-- Retorno:
--      Valor monetário total faturado
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK()

VAR _Resultado =
    SUM(
        fPedidos_Realizados[faturamento]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```

## Medidas de Contagem
<br>

```DAX
quantidade_motoristas = 

-- Medida:
--      quantidade_motoristas
--
-- Descrição:
--      Quantidade de motoristas ativos na tabela dMotoristas
--
-- Tabela origem:
--      dMotoristas
--
-- Regra de negócio:
--      Conta todos os motoristas ativos na tabela dMotoristas
--
-- Dependência:
--      dMotoristas
--
-- Retorno:
--      Número inteiro de motoristas ativos
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK()

VAR _Resultado =
    COUNTROWS(
        dMotoristas
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
quantidade_pedidos = 

-- Medida:
--      quantidade_pedidos
--
-- Descrição:
--      Total de pedidos registrados na tabela fPedidos_Realizados
--
-- Tabela origem:
--      fPedidos_Realizados
--
-- Regra de negócio:
--      Conta todas as linhas da tabela fPedidos_Realizados, representando cada pedido
--
-- Dependência:
--      'fPedidos_Realizados'
--
-- Retorno:
--      Número inteiro de pedidos
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK()

VAR _Resultado =
    COUNTROWS(
        fPedidos_Realizados
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
<br>

```DAX
quantidade_produtos = 

-- Medida:
--      quantidade_produtos
--
-- Descrição:
--      Total de produtos transportados na tabela fPedidos_Realizados
--
-- Tabela origem:
--      fPedidos_Realizados
--
-- Regra de negócio:
--      Soma todos os itens de produtos na coluna [quantidade_itens] da tabela fPedidos_Realizados
--
-- Dependência:
--      'fPedidos_Realizados'[quantidade_itens]
--
-- Retorno:
--      Número total de produtos transportados
--
-- Observação:
--      COALESCE é utilizado para evitar retorno BLANK()

VAR _Resultado =
    SUM(
        fPedidos_Realizados[quantidade_itens]
    )

RETURN
    COALESCE(
        _Resultado,
        0
    )
```
