# Medidas Financeira

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

