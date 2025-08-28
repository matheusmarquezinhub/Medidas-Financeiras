# Medidas Financeira

#### *TOTAL DO MES ATUAL DINAMICO*

TOTAL DO MES ATUAL DINAMICO DESPESAS = 

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

#### * *