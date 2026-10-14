# Dicionário de dados

> Resumo das tabelas do modelo semântico do Radar de Absenteísmo. Os números de linhas são da base fictícia da Vallora Logística S.A.; nenhum dado descreve uma empresa ou pessoa real.

O modelo é um esquema estrela com 3 tabelas fato de grãos diferentes e 10 dimensões compartilhadas, ligadas por 28 relacionamentos 1:N com filtro em uma única direção. As chaves organizacionais (unidade, departamento, cargo, turno, gestor e tempo de casa) ficam gravadas nas fatos, com o valor do dia, para que a análise respeite a lotação de cada pessoa em cada data.

## Tabelas fato

| Tabela | Grão | Papel no modelo | Relacionamentos principais | Linhas |
|---|---|---|---|---:|
| `Fato_Jornada` | Funcionário × dia com jornada prevista (sem folgas, feriados, férias e licenças protegidas) | Denominador da taxa: horas e dias previstos; base de exposição | Calendário (data), Funcionário, Unidade, Departamento, Cargo, Turno, Gestor, Faixa de tempo de casa | 662.876 |
| `Fato_AusenciaDiaria` | Evento × dia previsto afetado | Numerador da taxa: horas perdidas no dia em que foram perdidas, separadas em curta e longa duração | As mesmas da jornada, mais Tipo de ausência e Grupo de motivo | 29.085 |
| `Fato_EventoAusencia` | Evento de ausência (atestado, falta, afastamento...) | Contagem de eventos pela data de início, duração e distribuição | Calendário (data de início), Funcionário, organização, Tipo de ausência, Grupo de motivo, Faixa de tempo de casa | 13.639 |

## Dimensões

| Tabela | Grão | Papel no modelo | Relacionamentos principais | Linhas |
|---|---|---|---|---:|
| `Dim_Calendario` | Dia (01/01/2024 a 31/12/2025) | Tabela de datas: mês, trimestre, dia da semana, feriados, pontos facultativos e pontes | Filtra as três fatos | 731 |
| `Dim_Funcionario` | Funcionário, pseudonimizado | Atributos de grupo (sexo, faixa etária, contrato, regime, jornada); sem nome, CPF ou e-mail | Filtra as três fatos | 1.826 |
| `Dim_Unidade` | Unidade | Centros de distribuição e escritório; base da segurança por unidade | Filtra as três fatos | 5 |
| `Dim_Departamento` | Departamento | Área e diretoria; alvo do detalhamento por departamento | Filtra as três fatos | 11 |
| `Dim_Cargo` | Cargo | Nível, família e faixa salarial | Filtra as três fatos | 20 |
| `Dim_Turno` | Turno | Manhã, tarde, noite e administrativo | Filtra as três fatos | 4 |
| `Dim_Gestor` | Gestor imediato (equipe) | Equipes identificadas só por código; tamanho e tipo de equipe; base da segurança por gestor | Filtra as três fatos | 58 |
| `Dim_TipoAusencia` | Tipo de ausência | Define o que entra na taxa e o que é justificado | Filtra as fatos de ausência | 9 |
| `Dim_GrupoMotivo` | Grupo de motivo de saúde | Motivos agregados por capítulo da CID, nunca o CID individual | Filtra as fatos de ausência | 7 |
| `Dim_FaixaTempoCasa` | Faixa de tempo de casa | Tempo de casa calculado na data de cada registro | Filtra as três fatos | 6 |

## Tabelas auxiliares

| Tabela | Grão | Papel no modelo | Relacionamentos principais | Linhas |
|---|---|---|---|---:|
| `Aux_FaixaEventos` | Faixa de quantidade de eventos | Eixo do gráfico de concentração de ausências por pessoa | Desconectada; usada pelas medidas | 7 |
| `_Medidas` | Nenhum | Hospeda as 97 medidas DAX, organizadas em pastas | Nenhum | 0 |
| `Param_Dimensao` | Dimensão selecionável | Parâmetro de campo "Analisar por" da página de Áreas Críticas | Desconectada | 6 |
| `Param_Grupo` | Atributo de grupo | Parâmetro de campo "Agrupar por" da página de Análise de Grupos | Desconectada | 8 |
| `Seg_AcessoUsuario` | Usuário × unidade permitida | Perfis de acesso da segurança em nível de linha (usuários fictícios) | Usada pelas regras de RLS | 10 |
| `DQ_Resumo` | Verificação de qualidade | Registros afetados e tratamento de cada defeito da base | Desconectada | 9 |
