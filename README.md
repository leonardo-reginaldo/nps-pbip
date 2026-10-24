# Projeto NPS com Power BI (.pbip) + GitHub + Microsoft Teams

## Demanda Executiva Fictícia

Investigar o desempenho mensal e por dia da semana da base NPS 2025. Quando houver múltiplas respostas do mesmo paciente em um intervalo de 15 dias, considerar apenas o registro mais recente. A análise contempla as UBSs e seus respectivos setores que mais impactaram a experiência dos pacientes nas Zonas Norte e Oeste de São Paulo.

## 1. Escopo e Metodologia

O objetivo do projeto foi transformar dados brutos de pesquisas de satisfação em um painel estratégico capaz de orientar tomadas de decisão rápidas por gestores regionais e diretores.

### 1.1. A Metodologia NPS

A classificação das respostas seguiu o padrão global do *Net Promoter Score* (NPS):

- **Promotores (notas 9 e 10):** pacientes com alto nível de satisfação e propensão a recomendar os serviços.
- **Neutros (notas 7 e 8):** pacientes relativamente satisfeitos, mas sem forte propensão a recomendar.
- **Detratores (notas 0 a 6):** pacientes insatisfeitos, cujas avaliações podem indicar oportunidades de melhoria.

**Fórmula do NPS:**

```text
NPS = % de Promotores - % de Detratores
```

As faixas de pontuação foram categorizadas dinamicamente, utilizando referências de benchmarking do mercado corporativo:

| Classificação | Faixa de pontuação |
|---|---:|
| Zona de Excelência | 76 a 100 |
| Zona de Qualidade | 51 a 75 |
| Zona de Aprendizagem | 1 a 50 |
| Zona Crítica | -100 a 0 |

## 2. Arquitetura e Modelagem de Dados

Para garantir desempenho e escalabilidade no processamento de aproximadamente 100 mil registros, a modelagem foi estruturada com base no padrão **Star Schema (Esquema em Estrela)**.

- **Tabela fato:** `FATO_NPS`, contendo transações, notas, setores e referências das unidades.
- **Tabelas dimensão:** `DIM_UBS`, `DIM_CALENDARIO`, `DIM_NOTAS` e `DIM_NPS_LEGENDA`.
- **Tabela auxiliar:** `DATA_ATUALIZAÇÃO`, destinada ao acompanhamento da atualização dos dados.

### 2.1. Definição da Regra de Deduplicação

Quando um mesmo paciente apresenta múltiplas respostas dentro de um intervalo de 15 dias, considera-se apenas a **resposta mais recente do período**, conforme a regra de negócio definida para o projeto.

Essa regra busca reduzir a influência de avaliações repetidas no cálculo do NPS e tornar a comparação entre unidades mais consistente.

> **Nota metodológica:** a implementação deve definir explicitamente se os 15 dias são contados a partir da última resposta válida ou da primeira resposta do ciclo. Essa escolha altera o resultado da deduplicação.

### 2.2. Por que Aplicar essa Regra? — Justificativa de CX e Negócio

**Efeito de fadiga de pesquisa**

Evita atribuir peso excessivo às respostas de pacientes que frequentam a mesma UBS repetidamente em um curto intervalo, por exemplo, para triagem, consulta médica, curativo ou retorno.

**Redução do viés de duplicidade (*over-sampling*)**

Pacientes com tratamentos contínuos ou atendimentos recorrentes podem gerar várias avaliações no mesmo período. A regra reduz a possibilidade de que essas respostas tenham influência desproporcional no resultado agregado da unidade.

**Foco no ciclo de atendimento (*episode of care*)**

O intervalo de 15 dias funciona como uma janela operacional definida para agrupar avaliações próximas. O objetivo é analisar a experiência do paciente de maneira mais consistente, evitando que atendimentos muito próximos dominem os indicadores.

### 2.3. Como a Regra foi Estruturada no Modelo de Dados

No Power Query ou na camada de ETL, o fluxo lógico contempla as seguintes etapas:

1. **Ordenação das respostas:** organização dos registros por `ID_Paciente` e data da resposta.
2. **Identificação das respostas anteriores:** comparação temporal entre as respostas do mesmo paciente.
3. **Cálculo da diferença de dias:** identificação do intervalo entre as respostas, conforme a regra de negócio definida.
4. **Aplicação da regra de deduplicação:** identificação dos registros válidos e das respostas desconsideradas no cálculo.
5. **Criação de uma flag de controle:** classificação dos registros, por exemplo, com `FLAG_NPS_VALIDO = 1` para respostas consideradas válidas.

A implementação deve considerar a regra dos 15 dias de forma consistente, especialmente nos casos em que existam três ou mais respostas consecutivas do mesmo paciente.

### 2.4. Impacto no Dashboard do Power BI

**Métrica de volume bruto versus amostra válida**

Os indicadores apresentam o volume original de respostas e o total de respostas válidas após a aplicação da regra dos 15 dias, permitindo avaliar o impacto da deduplicação.

**Precisão do indicador NPS**

As medidas DAX de percentual de promotores, percentual de detratores e NPS consideram a flag de validade dos registros. Dessa forma, os indicadores, gráficos e mapas refletem a amostra definida pela regra de negócio.

## 3. Principais Desafios Técnicos e Soluções em DataViz

### 3.1. Classificação Dinâmica do NPS

Implementação de um cartão dinâmico (*New Card Visual*), com formatação condicional em DAX para atualizar automaticamente a classificação e a cor de destaque conforme o NPS calculado e os filtros aplicados.

### 3.2. Geolocalização sem Erros

Tratamento e padronização dos dados geográficos no Power Query, incluindo bairro, município e referências das unidades, para melhorar a precisão da representação espacial das UBSs em mapas do Azure Maps.

### 3.3. Consistência de KPIs e Linhas de Tendência

Construção de gráficos de evolução mensal com linhas de referência para comparar o desempenho observado com médias históricas ou metas definidas.

Essa abordagem facilita a identificação de períodos com desempenho abaixo do esperado e apoia a investigação de possíveis causas.

## 4. Governança, CI/CD e Distribuição em Produção

### 4.1. Controle de Versão com PBIP e GitHub

O projeto foi estruturado no formato **Power BI Project (`.pbip`)**, permitindo organizar os artefatos do relatório e do modelo semântico em arquivos passíveis de versionamento.

A integração com o GitHub Desktop permite:

- Rastrear alterações nos arquivos do projeto.
- Manter histórico de versões.
- Trabalhar com branches e revisão de alterações.
- Facilitar a colaboração entre desenvolvedores.
- Apoiar práticas de governança e implantação controlada.

### 4.2. Integração com Microsoft Teams

O relatório foi preparado para distribuição e consumo executivo por meio do Microsoft Teams, facilitando o acesso aos indicadores por gestores e diretores.

A arquitetura considera diretrizes de segurança e controle de acesso integradas ao Microsoft Entra ID, além da possibilidade de implementação de **Row-Level Security (RLS)** para restringir a visualização dos dados por unidade ou região.

**Requisito de licenciamento**

A documentação oficial da Microsoft explicita os requisitos de licenciamento para compartilhamento e consumo de conteúdo do Power BI. Em cenários de capacidade compartilhada, os usuários que publicam e consomem o conteúdo geralmente precisam de licenças Power BI Pro. Para distribuição a usuários gratuitos, é necessário atender aos requisitos de capacidade elegível, como Power BI Premium P1 ou capacidade equivalente ou superior, conforme o cenário. O Power BI Embedded F64 possui características de capacidade que devem ser avaliadas de acordo com o tipo de implantação e os usuários envolvidos.

Consulte a documentação oficial para confirmar os requisitos aplicáveis à arquitetura adotada.

## 5. Ferramentas Utilizadas

| Ferramenta | Aplicação no projeto |
|---|---|
| Power BI Desktop | Desenvolvimento dos relatórios e visualizações. |
| Power Query M | Limpeza, transformação e preparação dos dados. |
| DAX (*Data Analysis Expressions*) | Cálculo do NPS, indicadores percentuais e formatação condicional. |
| Power BI Project (`.pbip`) | Organização dos artefatos do projeto para versionamento. |
| GitHub Desktop | Controle de versão, histórico e colaboração. |
| Microsoft Teams | Distribuição e consumo dos relatórios. |
| Power BI Service | Publicação, compartilhamento e atualização dos relatórios. |
| Azure Maps | Visualização geográfica das unidades de saúde. |

## 6. Referências Bibliográficas e Metodológicas

- REICHHELD, Fred. *A Pergunta Definitiva 2.0: Como as Empresas Inteligentes Gerenciam a Satisfação do Cliente e Lucram Mais*. Alta Books.
- FEW, Stephen. *Information Dashboard Design: Displaying Data for At-a-Glance Monitoring*. Analytics Press.
- FERRARI, Alberto; RUSSO, Marco. *The Definitive Guide to DAX: Business Intelligence for Microsoft Power BI, SQL Server Analysis Services, and Excel*. Microsoft Press.
- MICROSOFT. *Power BI Project (PBIP) files*. Microsoft Learn.
- MICROSOFT. *Azure Maps Power BI visual*. Microsoft Learn.
- MICROSOFT. *Power BI service features by license type*. Microsoft Learn.

## Conclusão

Projetar dashboards vai muito além de criar gráficos visualmente atraentes: trata-se de transformar dados em informações confiáveis, indicadores acionáveis e decisões estratégicas.

Este projeto demonstra como a integração entre **Power BI, modelagem dimensional, DAX, governança de dados, GitHub e Microsoft Teams** pode apoiar a análise da experiência dos pacientes e identificar oportunidades de melhoria nos serviços de saúde.

A aplicação de regras explícitas de deduplicação, associada à transparência metodológica e ao controle de versões, contribui para a rastreabilidade dos indicadores e para uma análise mais consistente do desempenho das UBSs.

O resultado é uma proposta de solução analítica orientada a dados, preparada para apoiar gestores na identificação de padrões, na priorização de ações corretivas e na melhoria contínua da experiência dos pacientes.
