# 📊 Medidas DAX para Dashboard Financeiro


## 🔄 Controle de Atualização

### Dashboard Atualizado
```dax
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
```
**Função:** Tabela para apresentar quando o dashboard foi atualizado.

---

## 💰 Valores Básicos

### Valor Total
```dax
SUM(fDespesas[Valor])
```
**Função:** Soma o valor total das despesas.

---

## ⏰ Comparações Temporais

### Valor Total Ano Anterior
```dax
CALCULATE(
    [Valor Total],
    SAMEPERIODLASTYEAR(dCalendario[Id Data])
)
```
**Função:** Calcula o valor total no mesmo período do ano anterior.

### Valor Total Mês Anterior
```dax
CALCULATE(
    [Valor Total],
    DATEADD(dCalendario[Id Data],-1,MONTH)
)
```
**Função:** Calcula o valor total no mês anterior selecionado.

---

## 🎨 Formatação

### Valor Total Formatado
```dax
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
```
**Função:** Formata o valor total em unidades abreviadas: milhares (K), milhões (Mi) e bilhões (Bi).

---

## 📈 Variações Percentuais

### Variação do Ano Anterior vs Ano Atual
```dax
DIVIDE(
    [Valor Total] - [Valor Total Ano Anterior],
    [Valor Total Ano Anterior]
)
```
**Função:** Calcula a variação percentual entre o valor total do ano atual e do ano anterior.

### Variação do Mês Anterior vs Mês Atual
```dax
DIVIDE(
    [Valor Total] - [Valor Total Mes Anterior],
    [Valor Total Mes Anterior]
)
```
**Função:** Calcula a variação percentual entre o valor total do mês atual e do mês anterior.

---

## 🔄 Medidas Dinâmicas

### Total do Mês Atual Dinâmico
```dax
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
```
**Função:** Retorna o total de despesas do mês atual, considerando a data selecionada no filtro. Ideal para cartões dinâmicos.

### Total do Mês Anterior Dinâmico
```dax
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
```
**Função:** Retorna o total de despesas do mês anterior, considerando a data selecionada no filtro. Ideal para cartões dinâmicos.

### Nome do Mês Atual Dinâmico
```dax
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
```
**Função:** Retorna o nome do mês atual no formato abreviado (ex: "Ago/2024"), baseado na data selecionada. Ideal para cartões dinâmicos.

### Nome do Mês Anterior Dinâmico
```dax
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
```
**Função:** Retorna o nome do mês anterior no formato abreviado (ex: "Jul/2024"), baseado na data selecionada. Ideal para cartões dinâmicos.

---

## 📅 Análises YTD

### YTD (Year-to-Date)
```dax
CALCULATE(
    [TOTAL DESPESAS],
    DATESYTD(dCalendario[Id Data]),
    dCalendario[Id Data] <= TODAY()
)
```
**Função:** Calcula o acumulado das despesas desde o início do ano até a data atual.

### PYTD (Previous-Year-to-Date)
```dax
CALCULATE(
    [TOTAL DESPESAS],
    DATESYTD(
        DATEADD(dCalendario[Id Data], -1, YEAR)
    ),
    dCalendario[Id Data] <= EOMONTH(TODAY(), -12)
)
```
**Função:** Calcula o acumulado das despesas do ano anterior até a mesma data do ano atual. Exemplo: de 01/Jan até 28/Ago do ano anterior.

---

## 🔍 Análises Comparativas

### Maior e Menor (Gastos)
```dax
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
**Função:** Identifica as categorias com maior e menor gastos, exibindo os valores formatados.

---

## 💬 Tooltip Avançado

### Tooltip Mês Atual vs Mês Anterior + Variação
```dax
VAR vData = MAX(dCalendario[Id Data])
VAR vMesAtual = FORMAT(vData, "mmmm")

VAR vLucroAtual =
    CALCULATE(
        [Lucro],
        KEEPFILTERS(dCalendario)
    )

VAR vLucroAnterior =
    CALCULATE(
        [Lucro],
        DATEADD(dCalendario[Id Data], -1, MONTH)
    )
    
VAR vMesAnterior = FORMAT(EDATE(vData, -1), "mmmm")
VAR vDif = vLucroAtual - vLucroAnterior
VAR vVarPct = DIVIDE(vDif, vLucroAnterior, 0)

RETURN

"📅 " & vMesAtual & ": R$ " & FORMAT(vLucroAtual, "#,0.00") & UNICHAR(10) &
"📅 " & vMesAnterior & ": R$ " & FORMAT(vLucroAnterior, "#,0.00") & UNICHAR(10) &
"🚩 Diferença: R$ " & FORMAT(vDif, "+#,0.00;-#,0.00") & UNICHAR(10) &
IF(
    vDif > 0,
    "Resultado: 🎯 Aumento de lucro " & FORMAT(vVarPct, "0.00%"),
    IF(
        vDif < 0,
        "Resultado: 💸 Redução de lucro " & FORMAT(ABS(vVarPct), "0.00%"),
        "Resultado: 🆗 Sem variação"
    )
)
```
**Função:** Medida dedicada para tooltip comparativo entre mês atual vs mês anterior em gráficos de barras ou linhas.

---
