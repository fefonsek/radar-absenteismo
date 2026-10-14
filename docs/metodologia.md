# Metodologia

> Como o Radar de Absenteísmo calcula a taxa, separa sinal de ruído e protege as pessoas. Todos os números citados vêm da base fictícia da Vallora Logística S.A.

## 1. Qual definição de absenteísmo

Não existe uma fórmula única. As quatro mais usadas respondem a perguntas diferentes:

| Definição | Fórmula | O que mede | Limitação |
|---|---|---|---|
| Índice de horas perdidas | horas de ausência ÷ horas previstas | Proporção do tempo contratado que não foi trabalhado | Precisa de jornada prevista diária confiável |
| Índice de dias perdidos | dias de ausência ÷ dias previstos | O mesmo, em dias | Trata um atraso de 30 min igual a uma falta de dia inteiro, ou ignora o atraso |
| Fórmula clássica por headcount | dias-pessoa perdidos ÷ (headcount médio × dias úteis) | Aproximação quando não há ponto | O headcount médio esconde admissões, desligamentos, férias e jornadas parciais |
| Índice de frequência | eventos ÷ pessoas (ou ÷ tempo exposto) | Quantas vezes as pessoas se ausentam | Não diz quanto tempo foi perdido |

O painel usa o **índice de horas perdidas** como KPI principal e a frequência como complemento:

```text
Taxa de Absenteísmo % = Σ horas de ausência (tipos que contam) ÷ Σ horas previstas de trabalho
```

Horas são a única unidade que trata ao mesmo tempo as ausências parciais (atrasos, consultas), as jornadas diferentes (44 h, 36 h e 20 h semanais) e as pessoas que entram e saem no meio do mês. Para isso a base registra, para cada pessoa e cada dia, as horas que estavam previstas.

## 2. O que entra e o que sai da conta

| Entra na taxa | Não entra na taxa |
|---|---|
| Atestado médico; consulta ou exame (parcial); acidente de trabalho; afastamento previdenciário (INSS); falta injustificada; atraso ou saída antecipada; licença legal da CLT art. 473; acompanhamento de dependente | Férias; folgas da escala; feriados nacionais; licença maternidade e paternidade |

| Situação | Tratamento | Motivo |
|---|---|---|
| Férias | Fora do numerador e do denominador | Ausência planejada e de direito; incluir distorceria a comparação entre áreas que tiram férias em meses diferentes. |
| Feriados nacionais e folgas da escala | Fora do denominador | Não havia jornada prevista. Pontos facultativos (Carnaval, Corpus Christi) são dias de trabalho na operação, e por isso aparecem com mais faltas. |
| Licença maternidade e paternidade | Fora da taxa; contada à parte | Licença protegida por lei. Mantê-la na taxa puniria, estatisticamente, as áreas com mais mulheres. |
| Afastamentos longos (INSS) | Dentro da taxa total, mas separados como longa duração | O tempo foi perdido, mas é gerido como caso pela saúde ocupacional, não pelo gestor no dia a dia. |
| Admissões e desligamentos no meio do período | Só os dias de vínculo entram no denominador | Ninguém é cobrado por dias em que não era funcionário. |
| Jornada parcial (SAC 6 h, aprendiz 4 h) | Horas previstas reais por dia | A taxa em horas compara de forma justa grupos com jornadas diferentes. |
| Ausência parcial | Horas efetivamente perdidas | Um atraso de 45 min conta 0,75 h, não um dia. |

**Período.** A taxa é sempre a razão das somas no período filtrado, nunca a média de taxas mensais, que daria o mesmo peso a meses com quadros diferentes. Eventos são atribuídos à data de início; horas, ao dia em que foram perdidas.

## 3. Decomposição: curta e longa duração

```text
Taxa Absenteísmo % = Taxa Curta Duração % (eventos até 15 dias corridos) + Taxa Longa Duração % (acima de 15 dias)
```

O limite de 15 dias é o ponto em que, pela Lei 8.213/91, o afastamento por doença passa da empresa para o INSS. A curta duração é a parte que o gestor influencia; a longa depende de saúde ocupacional, ergonomia e retorno ao trabalho. Por isso os alertas olham só a curta duração, e a taxa total é sempre mostrada decomposta, também por tipo de ausência e por grupo de motivo de saúde.

## 4. Regra de alerta: magnitude e evidência

Comparar grupos só pela taxa gera falsos alarmes: uma equipe de 6 pessoas com um afastamento longo vira "a pior da empresa". O status de cada grupo exige dois critérios ao mesmo tempo:

- **magnitude:** índice de curta duração = taxa de curta duração do grupo ÷ taxa de curta duração da empresa;
- **evidência:** z-score da frequência = (eventos observados − esperados) ÷ √esperados, com esperados = dias previstos do grupo × eventos por dia previsto da empresa.

| Status | Regra | Leitura |
|---|---|---|
| Crítico | índice ≥ 1,20 e z ≥ 3 | Desvio grande e muito improvável por acaso. Prioridade de investigação. |
| Atenção | índice ≥ 1,10 e z ≥ 2 | Desvio relevante; acompanhar. |
| Favorável | índice ≤ 0,90 e z ≤ −2 | Abaixo da empresa de forma consistente; boas práticas a entender. |
| Dentro do esperado | demais casos | Inclui grupos pequenos com taxa alta e pouca evidência. |

A exposição do z-score é medida em dias, não em horas, para não penalizar jornadas parciais. Como a variância real é bem maior que a de Poisson (índice de dispersão de 6,7), os limiares são conservadores.

## 5. Testes estatísticos

Além do painel, as associações foram testadas fora do Power BI, em Python:

- **Modelo binomial negativo** para a contagem de eventos de curta duração por pessoa, com exposição em dias previstos, ajustado por unidade, departamento, turno, idade, sexo, tempo de casa e ano. Ele confirma o efeito do turno da noite (razão de taxas 1,31, IC 95% 1,23 a 1,39) e do tempo de casa até 6 meses (1,26, IC 1,18 a 1,35), e não encontra efeito de sexo (1,005, IC 0,95 a 1,06).
- **Confundimento.** A associação entre tamanho de equipe e absenteísmo some quando se separa por turno: a única equipe muito grande é noturna.
- **Sazonalidade.** Índice sazonal mensal da curta duração e comparação sempre com o mesmo mês do ano anterior ou com média móvel, para não confundir sazonalidade com tendência.
- **Concentração.** Gini, participação dos 10% e 20% com mais eventos e índice de dispersão (variância ÷ média).

Todos os números do painel foram recalculados de forma independente em Python e conferidos contra as medidas DAX.

## 6. Qualidade dos dados

A base simula a exportação de um ERP brasileiro e recebeu 146 defeitos reais de extração (duplicidades, datas invertidas, chaves órfãs, grafias inconsistentes, horas acima da jornada). O Power Query trata cada um e uma página do painel mostra, verificação por verificação, quantos registros foram afetados e o que foi feito, além de checagens de integridade calculadas no próprio modelo.

## 7. Privacidade e uso responsável

- Sem nome, CPF ou e-mail real: pessoas são identificadas por um código pseudonimizado, oculto no relatório.
- Motivos de saúde só aparecem agregados por capítulo da CID, nunca o CID individual.
- Indicadores por grupo de pessoas (faixa etária, sexo, contrato, tempo de casa) só são exibidos quando o grupo tem 10 ou mais pessoas expostas.
- Rankings e alertas existem no nível de equipe, nunca de pessoa.
- Segurança em nível de linha (RLS) por unidade e por gestor.
- O painel traz uma nota explícita de uso não punitivo: os padrões são hipóteses para investigar, não provas de causa.

## Limitações

Lotação estática (ninguém muda de equipe), gestores fora do quadro, feriados apenas nacionais e ausência de dados de horas extras, clima organizacional ou volume operacional. Por isso nenhuma causa pode ser testada: o painel aponta onde olhar, e a investigação qualitativa diz por quê.
