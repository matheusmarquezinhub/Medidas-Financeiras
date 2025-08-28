# Medidas Financeira

### *VALOR TOTAL*

```bash
SUM(fDespesas[Valor])

--- Somatoria do valor total 
```
---
### *VALOR TOTAL ANO ANTERIOR* 

```bash
CALCULATE(
    [VALOR TOTAL],
    SAMEPERIODLASTYEAR(dCalendario[Id Data])
)

--- Somatoria do valor total do ano anterior
```
---
### *VALOR TOTAL MES ANTERIOR* 
```bash
[VALOR TOTAL] = 
CALCULATE(
    [TOTAL DESPESAS],
DATEADD(dCalendario[Id Data],-1,MONTH)
)

--- Somatoria do valor total do mês anterior
```
---

### *VALOR TOTAL FORMATADO* 
```bash
VAR vFormatado = [VALOR TOTAL]

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

--- Aqui temos a formatação dos valores por K,Mi,Bi.
```
---

### *TOTAL DO MES ATUAL DINAMICO*

```bash VAR DataSelecionada = 
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

--- Essa medida é para ser inserida em um card e ela irá mostrar o valor total atual.
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
        [TOTAL DESPESAS],
        FILTER(
            ALLSELECTED(dCalendario[Id Data]),
            YEAR(dCalendario[Id Data]) = YEAR(DataReferencia) &&
            MONTH(dCalendario[Id Data]) = MONTH(DataReferencia)
        )
    )

--- Essa medida é para ser inserida em um card e ela irá mostrar o valor total do mês anterior. 
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

--- Essa medida é para ser inserida em um card e ela irá mostrar o mês atual.
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

--- Essa medida é para ser inserida em um card e ela irá mostrar o mês anterior. 
```
---