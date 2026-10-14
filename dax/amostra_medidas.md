# Amostra de medidas DAX

> 8 das 97 medidas do modelo, escolhidas para mostrar as principais técnicas: regra de negócio no filtro, taxa como razão de somas, janela móvel que não inventa meses, comparação com a empresa ignorando filtros organizacionais, teste de evidência, classificação de status, concentração e supressão de grupos pequenos. As medidas citadas e não mostradas aqui (por exemplo `[Horas Previstas]` e `[Eventos Ausência]`) fazem parte do projeto completo.

## 1. Horas Ausência

A regra de negócio fica dentro da medida: só os tipos de ausência marcados para entrar na taxa são somados. `KEEPFILTERS` respeita o filtro que o usuário já aplicou em tipo de ausência, em vez de substituí-lo.

```dax
Horas Ausência =
CALCULATE (
    SUM ( Fato_AusenciaDiaria[horas_ausencia] ),
    KEEPFILTERS ( Dim_TipoAusencia[considera_taxa] = 1 )
)
```

## 2. Taxa Absenteísmo %

O KPI principal é uma razão de somas, nunca uma média de taxas. Assim, meses, áreas e jornadas de tamanhos diferentes pesam o que realmente pesam.

```dax
Taxa Absenteísmo % =
DIVIDE ( [Horas Ausência], [Horas Previstas] )
```

## 3. Taxa Móvel 3M

A divisão é refeita sobre a janela inteira. A medida só devolve valor quando a janela tem os 3 meses completos, para não mostrar uma "média de 3 meses" calculada sobre 1 mês no início do período.

```dax
Taxa Móvel 3M =
VAR Janela =
    DATESINPERIOD ( Dim_Calendario[data], MAX ( Dim_Calendario[data] ), -3, MONTH )
VAR Meses =
    CALCULATE ( DISTINCTCOUNT ( Dim_Calendario[ano_mes] ), Janela )
RETURN
    IF ( Meses = 3, CALCULATE ( [Taxa Absenteísmo %], Janela ) )
```

## 4. Taxa Curta Empresa %

A referência da empresa remove os filtros organizacionais e de pessoa, mas mantém período e tipo de ausência. Assim, qualquer área é comparada com a empresa no mesmo recorte de tempo.

```dax
Taxa Curta Empresa % =
CALCULATE (
    [Taxa Curta Duração %],
    REMOVEFILTERS ( Dim_Unidade ),
    REMOVEFILTERS ( Dim_Departamento ),
    REMOVEFILTERS ( Dim_Cargo ),
    REMOVEFILTERS ( Dim_Turno ),
    REMOVEFILTERS ( Dim_Gestor ),
    REMOVEFILTERS ( Dim_Funcionario ),
    REMOVEFILTERS ( Dim_FaixaTempoCasa )
)
```

## 5. Z-Score Frequência

Quantos desvios-padrão os eventos observados estão acima ou abaixo do esperado se o grupo se comportasse como a empresa. O esperado vem da exposição em dias previstos (aproximação de Poisson).

```dax
Z-Score Frequência =
VAR Observado = [Eventos Ausência] + 0
VAR Esperado = [Eventos Esperados]
RETURN
    IF ( Esperado > 0, DIVIDE ( Observado - Esperado, SQRT ( Esperado ) ) )
```

## 6. Status Alerta

O status exige magnitude e evidência ao mesmo tempo. Uma equipe pequena com taxa alta, mas sem evidência estatística, fica como "Dentro do esperado".

```dax
Status Alerta =
VAR Indice = [Índice Curta vs Empresa]
VAR Z = [Z-Score Frequência]
RETURN
    SWITCH (
        TRUE (),
        ISBLANK ( Indice ) || ISBLANK ( Z ), BLANK (),
        Indice >= 1.20 && Z >= 3, "Crítico",
        Indice >= 1.10 && Z >= 2, "Atenção",
        Indice <= 0.90 && Z <= -2, "Favorável",
        "Dentro do esperado"
    )
```

## 7. Concentração Top 10%

Percentual dos eventos gerados pelos 10% de pessoas expostas com mais eventos. A tabela virtual parte da jornada, então quem não teve nenhuma ausência entra com zero; o desempate por id deixa o resultado determinístico.

```dax
Concentração Top 10% =
VAR Base =
    ADDCOLUMNS (
        SUMMARIZE ( Fato_Jornada, Dim_Funcionario[id_funcionario] ),
        "@ev", [Eventos Ausência] + 0
    )
VAR n = INT ( COUNTROWS ( Base ) * 0.10 )
VAR Maiores = TOPN ( n, Base, [@ev], DESC, Dim_Funcionario[id_funcionario], ASC )
RETURN
    DIVIDE ( SUMX ( Maiores, [@ev] ), SUMX ( Base, [@ev] ) )
```

## 8. Taxa Curta (Grupo Protegido) %

Privacidade aplicada na medida: indicadores por grupo de pessoas ficam em branco quando o grupo tem menos de 10 pessoas expostas.

```dax
Taxa Curta (Grupo Protegido) % =
IF ( [Funcionários Expostos] >= [Tamanho Mínimo de Grupo], [Taxa Curta Duração %] )
```
