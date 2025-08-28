# Medidas Financeira

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

--- Retorna o total de despesas do mês atual, considerando a data selecionada no filtro. Ideal para exibir em um cartão (card) dinâmico.
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

--- Retorna o total de despesas do mês anterior, considerando a data selecionada no filtro.Ideal para exibir em um cartão (card) dinâmico. 
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

--- Retorna o nome do mês atual no formato abreviado (ex: "Ago/2024"), baseado na data selecionada. Ideal para exibir em um cartão (card) dinâmico.
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

--- Retorna o nome do mês anterior no formato abreviado (ex: "Jul/2024"), baseado na data selecionada. Ideal para exibir em um cartão (card) dinâmico. 
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
--- Calcula o acumulado das despesas do ano anterior até a mesma data do ano atual. Exemplo: de 01/Jan até 28/Ago do ano anterior.
---
