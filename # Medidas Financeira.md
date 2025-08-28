# Medidas Financeira

### Dashboard Atualizado
```bash
let
    // Pega o horário UTC atual
    FonteUTC = DateTimeZone.UtcNow(),

    // Converte para UTC-3 (Brasília, sem horário de verão)
    HorarioBrasilia = DateTimeZone.SwitchZone(FonteUTC, -3),

    // Remove a parte do fuso (se quiser só a data/hora)
    DataHoraSemFuso = DateTimeZone.RemoveZone(HorarioBrasilia),

    // Cria tabela
    Tabela = #table(1, {{DataHoraSemFuso}}),

    // Renomeia coluna
    Resultado = Table.RenameColumns(Tabela, {{"Column1", "Reload"}})
in
    Resultado

--- Tabela para apresentar quando o dashboard foi atualizado.
```
---

### *VALOR TOTAL*

```bash
SUM(fDespesas[Valor])

--- Soma o valor total 
```
---
### *VALOR TOTAL ANO ANTERIOR* 

```bash
CALCULATE(
    [Valor Total],
    SAMEPERIODLASTYEAR(dCalendario[Id Data])
)

--- Calcula o valor total no mesmo período do ano anterior.
```
---
### *VALOR TOTAL MES ANTERIOR* 
```bash
CALCULATE(
    [Valor Total],
    DATEADD(dCalendario[Id Data],-1,MONTH))

--- Calcula o valor total no mês anterior selecionado.
```
---

### *VALOR TOTAL FORMATADO* 
```bash
VAR vFormatado = [Valor Total]

VAR vResultado =
    SWITCH(
            TRUE(),
            vFormatado >= 1000000000, FORMAT(vFormatado, "R$ #,0,,,.00 Bi"),
            vFormatado >= 1000000, FORMAT(vFormatado, "R$ #,0,,.00 Mi"),
            vFormatado >= 1000, FORMAT(vFormatado, "R$ #,0,.00 K"),
            FORMAT(vFormatado, "R$ #")
        )
RETURN
    vResultado

--- Formata o valor total em unidades abreviadas: milhares (K), milhões (Mi) e bilhões (Bi).
```
---
### *VARIAÇÂO DO ANO ANTERIOR VS ANO ATUAL*

```bash
DIVIDE(
    [Valor Total]-
    [Valor Total Ano Anterior],
    [Valor Total Ano Anterior]
)

--- Calcula a variação percentual entre o valor total do ano atual e do ano anterior.
```
---

### *VARIAÇÂO DO MÊS ANTERIOR VS MÊS ATUAL*
```bash
DIVIDE(
    [Valor Total]-
    [Valor Total Mes Anterior],
    [Valor Total Mes Anterior]
)

--- Calcula a variação percentual entre o valor total do mês atual e do mês anterior.
```
---

### *TOTAL DO MES ATUAL DINAMICO*

```bash
VAR DataSelecionada = 
    SELECTEDVALUE(dCalendario[Id Data], MAX(fDespesas[Data])) 

VAR DataReferencia = 
    SWITCH(
        TRUE(),
        DataSelecionada = TODAY(), EOMONTH(TODAY(), -1)+1,
        EOMONTH(DataSelecionada, -1)+1
    )

RETURN

    CALCULATE(
        [TOTAL DESPESAS],
            FILTER(
            ALLSELECTED(dCalendario[Id Data]),
            YEAR(dCalendario[Id Data]) = YEAR(DataReferencia) &&
            MONTH(dCalendario[Id Data]) = MONTH(DataReferencia)
        )
    )

--- Retorna o total de despesas do mês atual, considerando a data selecionada no filtro.
--- Ideal para exibir em um cartão (card) dinâmico.
```
---

### *TOTAL DO MES ANTERIOR DINAMICO*

```bash 
VAR DataSelecionada = 
    SELECTEDVALUE(dCalendario[Id Data], MAX(fDespesas[Data]))

VAR DataReferencia = 
    SWITCH(
        TRUE(),
        DataSelecionada = TODAY(), EOMONTH(TODAY(), -1),
        EOMONTH(DataSelecionada, -1)
    )

RETURN
    CALCULATE(
        [Valor Total],
        FILTER(
            ALLSELECTED(dCalendario[Id Data]),
            YEAR(dCalendario[Id Data]) = YEAR(DataReferencia) &&
            MONTH(dCalendario[Id Data]) = MONTH(DataReferencia)
        )
    )

--- Retorna o total de despesas do mês anterior, considerando a data selecionada no filtro.

---Ideal para exibir em um cartão (card) dinâmico. 
```
---
### *NOME DO MES ATUAL DINAMICO*

```bash
VAR DataSelecionada = 
    SELECTEDVALUE(dCalendario[Id Data], MAX(fDespesas[Data]))

VAR DataReferencia = 
    SWITCH(
        TRUE(),
        DataSelecionada = TODAY(), EOMONTH(TODAY(), -1)+1,
                EOMONTH(DataSelecionada, -1)+1
    )

RETURN 
    FORMAT(DataReferencia, "MMM/yyyy")

--- Retorna o nome do mês atual no formato abreviado (ex: "Ago/2024"), baseado na data selecionada. 

--- Ideal para exibir em um cartão (card) dinâmico.
```

### *NOME DO MES ANTERIOR DINAMICO*
```bash
VAR DataSelecionada = 
    SELECTEDVALUE(dCalendario[Id Data], MAX(fDespesas[Data]))

VAR DataReferencia = 
    SWITCH(
        TRUE(),
        DataSelecionada = TODAY(), EOMONTH(TODAY(), -1),
        EOMONTH(DataSelecionada, -1)
    )

RETURN 
    FORMAT(DataReferencia, "MMM/yyyy")

--- Retorna o nome do mês anterior no formato abreviado (ex: "Jul/2024"), baseado na data selecionada. 

--- Ideal para exibir em um cartão (card) dinâmico. 
```
---
### *YTD (Year-to-Date)*
```bash
CALCULATE(
    [TOTAL DESPESAS],
    DATESYTD(dCalendario[Id Data]),
    dCalendario[Id Data]<=TODAY()
)

--- Calcula o acumulado das despesas desde o início do ano até a data atual.
```
---
### *PYTD (Previous-Year-to-Date)*

```bash
CALCULATE(
    [TOTAL DESPESAS],
    DATESYTD(
        DATEADD(dCalendario[Id Data], -1, YEAR)
    ),
    dCalendario[Id Data] <= EOMONTH(TODAY(), -12)
)
--- Calcula o acumulado das despesas do ano anterior até a mesma data do ano atual. 

--- Exemplo: de 01/Jan até 28/Ago do ano anterior.

```
---

### *MAIOR E MENOR (GASTOS)*
```bash
VAR Maior = 
    CALCULATE(
        MAXX(
            SUMMARIZE(fDespesasCarros,fDespesasCarros[CARROS], "Total", SUM(fDespesasCarros[R$ TOTAL])),
            [Total]
        )
    )
VAR Menor = 
    CALCULATE(
        MINX(
            SUMMARIZE(fDespesasCarros,fDespesasCarros[CARROS], "Total", SUM(fDespesasCarros[R$ TOTAL])),
            [Total]
        )
    )
VAR CategoriaMaior = 
    CALCULATE(
        FIRSTNONBLANK(fDespesasCarros[CARROS], 1),
        FILTER(
            VALUES(fDespesasCarros[CARROS]),
            CALCULATE(SUM(fDespesasCarros[R$ TOTAL])) = Maior
        )
    )
VAR CategoriaMenor = 
    CALCULATE(
       FIRSTNONBLANK(fDespesasCarros[CARROS], 1),
        FILTER(
            VALUES(fDespesasCarros[CARROS]),
            CALCULATE(SUM(fDespesasCarros[R$ TOTAL])) = Menor
        )
    )
RETURN
"🔧 SEU TEXTO AQUI PARA MAIOR: " & CategoriaMaior & " ( " & FORMAT(Maior, "R$ #,##0") & " )" &
"  |  💡 SEU TEXTO AQUI PARA MENOR " & CategoriaMenor & " ( " & FORMAT(Menor, "R$ #,##0") & " )"
```
---
### *TOOLTIP MES ATUAL VS MES ANTERIOR + VARIAÇÃO*

```bash
VAR vCategoria = SELECTEDVALUE(fSemParar[Carros])
VAR vData = MAX(dCalendario[Id Data])
VAR vMesAtual = FORMAT(vData, "mmmm")

VAR vAtual =
    CALCULATE(
        [Total de despesas sem parar],
        KEEPFILTERS(dCalendario)
    )

VAR vAnterior =
    CALCULATE(
        [Total de despesas sem parar],
        DATEADD(dCalendario[Id Data], -1, MONTH)
    )
    
VAR vMesAnterior = FORMAT(EDATE(vData, -1), "mmmm")
VAR vDif = vAtual - vAnterior
VAR vVarPct = DIVIDE(vDif, vAnterior, 0)

RETURN

"🚗 Categoria: " & vCategoria & UNICHAR(10) &
"📅 " & vMesAtual & ": R$ " & FORMAT(vAtual, "0.00") & UNICHAR(10) &
"📅 " & vMesAnterior & ": R$ " & FORMAT(vAnterior, "0.00") & UNICHAR(10) &
"〰 Diferença: R$ " & FORMAT(vDif, "+0.00;-0.00") & UNICHAR(10) &
IF(
    vDif > 0,
    "Variação: 🔴 Aumento de " & FORMAT(vVarPct, "0.00%"),
    IF(
        vDif < 0,
        "Variação: 🟢 Redução de " & FORMAT(ABS(vVarPct), "0.00%"),
        "Variação: 🔵 Sem variação"
    )
)

--- Medida dedicada para facilitar no tooltip entre mes atual vs mes anterior (gráfico de barras ou linhas)
```
---
