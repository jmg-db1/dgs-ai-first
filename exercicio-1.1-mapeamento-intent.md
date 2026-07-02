# Exercício 1.1 — Mapeamento de Intent com Engenharia de Contexto

**Papel:** Product Specialist (BA/UX)
**Cenário:** NovaTech — Assistente de IA para atendimento ao cliente
**Autor:** Julia
**Data:** 02/07/2026

---

## 1. Estratégia de Contexto

Este exercício aplica **progressive disclosure**: em vez de fornecer toda a documentação da NovaTech de uma vez, o contexto foi revelado ao Claude em 3 etapas, cada uma se apoiando no resultado da etapa anterior. O objetivo é observar como essa abordagem afeta a qualidade e a profundidade da análise, evitando degradação por excesso de informação (context rot).

---

## 2. Etapa 1 — Visão Geral

**Decisão de contexto:** Forneci apenas títulos, metadados (versão, responsável, classificação) e resumos de 1-2 linhas dos 5 documentos, sem conteúdo completo. O objetivo era obter uma primeira leitura de temas e riscos "de fora para dentro", sem viés de detalhes que só apareceriam num segundo momento.

**Prompt utilizado:**

~~~text
Estou fazendo a análise de intent para um projeto de assistente de IA de atendimento ao cliente de uma transportadora (NovaTech). Antes de entrevistar stakeholders, quero uma primeira leitura só com base nos metadados e resumos dos documentos internos — ainda não vou te passar o conteúdo completo.
Aqui estão os 5 documentos-chave:

1. POL-001 — Política de Devolução de Mercadorias (v3.1, normativo, responsável: Diretoria de Operações). Define regras de devolução em até 7 dias úteis após entrega, com exceções para cargas perigosas, refrigeradas com rompimento de cadeia de frio, e cargas com lacre violado.
2. PROC-042 — Procedimento de Cálculo de Frete Especial (v1.0, emitido em 03/03/2023, responsável: Diretoria Comercial). Define fórmula de frete para cargas acima de 500kg, com multiplicadores por região e fator de peso. Documento sem indicação formal de vigência.
3. PROC-042-v2 — Procedimento de Cálculo de Frete Especial (Revisado) (v2.0, emitido em 10/11/2023, mesmo responsável). Mesma numeração e objetivo do documento anterior, mas com multiplicadores e fatores diferentes. Também sem indicação formal de que substitui a v1 — os dois coexistem sem hierarquia clara.
4. SLA-2024 — Tabela de SLA por Tipo de Cliente (v2024.1, contratual, responsável: Diretoria Comercial + Operações). Classifica clientes em 3 tiers (Gold, Silver, Standard) com SLAs de resposta e resolução diferenciados para chamados gerais e incidentes críticos.
5. FAQ-Atendimento — Perguntas Frequentes do Time de Suporte (sem versão controlada, sem responsável formal, documento colaborativo mantido pelo time de atendimento ao longo de 2 anos). Contém 47 perguntas e respostas baseadas na experiência prática dos atendentes, explicitamente não validado contra a documentação oficial.

Com base só nesses metadados e resumos (ainda sem o conteúdo completo), quero que você:

1. Monte um mapa dos principais temas cobertos por essa documentação (ex: devolução, frete, SLA, etc.).
2. Levante hipóteses de gaps ou riscos que você já suspeita, só pela forma como esses documentos estão descritos (preste atenção especial a coisas como: documentos com numeração igual mas versões diferentes sem indicação de vigência, documentos informais vs. normativos, e documentos sem responsável formal).
3. Aponte quais desses 5 documentos você recomendaria priorizar para uma análise mais profunda, e por quê.
~~~

**Output obtido:**

> **Mapa de temas:** quatro famílias de intent identificadas — devolução de mercadorias (POL-001), cálculo de frete especial (PROC-042/v2), SLA/prioridade de atendimento (SLA-2024) e conhecimento tácito do dia a dia (FAQ-Atendimento), sendo esta última a camada transversal com maior volume provável de intents reais, porém menor confiabilidade como fonte de verdade.
>
> **Hipóteses de gaps/riscos:** (1) risco crítico no par PROC-042/PROC-042-v2 — mesma numeração, mesmo responsável, multiplicadores diferentes, sem hierarquia formal declarada, com potencial de gerar valores de frete divergentes para o mesmo cenário; (2) risco alto no FAQ — fonte não validada, provavelmente a mais fiel aos intents reais dos clientes, mas com chance de conter desvios de política naturalizados pelo time e nunca sancionados pela Diretoria; (3) risco médio na SLA-2024 — dupla responsabilidade (Comercial + Operações) como padrão clássico de ambiguidade nas bordas/exceções; (4) risco médio no PROC-042 v1 isolado — ausência de vigência formal como sinal de governança documental frágil; (5) risco baixo-médio na POL-001 — três exceções (carga perigosa, refrigerada com rompimento de cadeia de frio, lacre violado) com critérios de verificação provavelmente distintos entre si, área onde intents de atendimento tendem a se concentrar.
>
> **Priorização recomendada:** 1º PROC-042/v2 (como par, não isoladamente) — único risco capaz de invalidar retroativamente uma resposta do assistente; 2º FAQ-Atendimento — tratar como fonte de intents reais, não de respostas corretas; 3º SLA-2024 — esclarecer com os dois donos onde as bordas de decisão são resolvidas na prática; 4º POL-001 — revisar critérios de verificação das três exceções.

**Análise da qualidade do output:**
- O Claude identificou corretamente o par PROC-042/v2 como prioridade máxima só a partir dos metadados (mesma numeração, mesmo responsável, multiplicadores diferentes, sem indicação de vigência).
- Também sinalizou o FAQ como fonte de "intents reais" mas de baixa confiabilidade — uma distinção sofisticada que não estava explícita no meu resumo.
- Limitação: sem o conteúdo completo, os riscos ficaram no nível de hipótese ("prováveis", "sugere") — correto e esperado nesta etapa.

---

## 3. Etapa 2 — Análise Profunda

**Decisão de contexto:** Com base no mapa da Etapa 1, escolhi aprofundar o par **PROC-042 (v1) e PROC-042-v2**, por ser o único risco capaz de invalidar retroativamente uma resposta do assistente (valores de frete divergentes = dinheiro real e disputa contratual). Forneci o conteúdo integral de ambos.

**Prompt utilizado:**

~~~text
Na etapa anterior, você analisou só os metadados de 5 documentos internos de uma transportadora e apontou o par PROC-042 / PROC-042-v2 como prioridade máxima de análise, pela contradição de multiplicadores sem hierarquia formal declarada.
Agora vou te passar o conteúdo completo dos dois documentos. Quero que você:
Identifique todas as inconsistências entre as duas versões (valores de fórmula, multiplicadores regionais, prazos de entrega, condições especiais) — não só as óbvias.
Aponte qualquer indicação (ou falta de indicação) no próprio texto de qual versão está vigente ou se há regra de transição.
Avalie o risco prático: se um pipeline de RAG indexasse os dois documentos sem tratamento, que tipo de resposta contraditória o assistente poderia gerar para o mesmo cenário (ex: mesma carga, mesmo destino)?
Sugira, do ponto de vista de produto (não técnico), como essa ambiguidade deveria ser resolvida antes de ir para o discovery com o cliente.

Documento 1 — PROC-042 (v1.0, emitido em 03/03/2023):
# PROC-042 — Procedimento de Cálculo de Frete Especial
**Versão:** 1.0
**Data de emissão:** 03/03/2023
**Responsável:** Diretoria Comercial
**Status:** Este documento não possui indicação formal de vigência ou obsolescência no sistema da NovaTech. Coexiste com a versão PROC-042-v2.
## 1. Objetivo
Definir a fórmula e os parâmetros para cálculo de frete especial aplicável a cargas com peso acima de 500kg.
## 2. Fórmula de cálculo
O frete especial é calculado como:
Valor do frete = Valor base × Multiplicador regional × Fator de peso
Onde:
- Valor base = tarifa publicada na tabela mensal de fretes.
- Multiplicador regional = fator aplicado conforme a região de destino (seção 2.1).
- Fator de peso = 1.0 para cargas de 500kg a 1.000kg; 1.2 para cargas de 1.001kg a 3.000kg; 1.5 para cargas acima de 3.000kg.
### 2.1. Multiplicadores regionais
| Região | Multiplicador |
|--------|--------------|
| Sul | 1.2 |
| Sudeste | 1.0 |
| Centro-Oeste | 1.3 |
| Nordeste | 1.4 |
| Norte | 1.6 |
## 3. Prazo de entrega para frete especial
O prazo de entrega para frete especial é calculado como o prazo padrão da rota + 2 dias úteis adicionais para manuseio de carga pesada.
## 4. Condições especiais
- Cargas acima de 5.000kg requerem aprovação prévia do gerente de operações regional.
- Cargas perigosas com peso acima de 500kg seguem tabela específica (PROC-043: Frete de Cargas Perigosas).
- Descontos de volume (mais de 10 fretes especiais/mês para o mesmo cliente) devem ser negociados pelo Comercial e registrados em aditivo contratual.

Documento 2 — PROC-042-v2 (v2.0, emitido em 10/11/2023):
# PROC-042-v2 — Procedimento de Cálculo de Frete Especial (Revisado)
**Versão:** 2.0
**Data de emissão:** 10/11/2023
**Responsável:** Diretoria Comercial
**Status:** Este documento não possui indicação formal de que substitui o PROC-042 v1. Ambos coexistem no SharePoint sem hierarquia clara.
## 1. Objetivo
Definir a fórmula e os parâmetros atualizados para cálculo de frete especial aplicável a cargas com peso acima de 500kg. Os multiplicadores foram revisados para refletir os custos operacionais atualizados de cada região.
## 2. Fórmula de cálculo
O frete especial é calculado como:
Valor do frete = Valor base × Multiplicador regional × Fator de peso
Onde:
- Valor base = tarifa publicada na tabela mensal de fretes.
- Multiplicador regional = fator aplicado conforme a região de destino (seção 2.1).
- Fator de peso = 1.0 para cargas de 500kg a 1.000kg; 1.15 para cargas de 1.001kg a 3.000kg; 1.4 para cargas acima de 3.000kg.
### 2.1. Multiplicadores regionais (atualizados em novembro/2023)
| Região | Multiplicador |
|--------|--------------|
| Sul | 1.3 |
| Sudeste | 1.1 |
| Centro-Oeste | 1.4 |
| Nordeste | 1.5 |
| Norte | 1.8 |
## 3. Prazo de entrega para frete especial
O prazo de entrega para frete especial é calculado como o prazo padrão da rota + 3 dias úteis adicionais para manuseio e roteirização de carga pesada (anteriormente era + 2 dias na versão anterior).
## 4. Condições especiais
- Cargas acima de 5.000kg requerem aprovação prévia do gerente de operações regional.
- Cargas perigosas com peso acima de 500kg seguem tabela específica (PROC-043: Frete de Cargas Perigosas). Nota: a PROC-043 está em processo de revisão pelo Compliance e pode sofrer alterações.
- Descontos de volume: a partir de 8 fretes especiais/mês para o mesmo cliente, aplicar desconto de 5% sobre o multiplicador regional. Acima de 15 fretes/mês, desconto de 10%. Descontos maiores requerem aprovação da Diretoria Comercial.
## 5. Disposições transitórias
Durante o período de transição, chamados abertos antes de 01/12/2023 que ainda estejam em processamento devem usar os multiplicadores da versão anterior (PROC-042 v1). Chamados novos a partir de 01/12/2023 devem usar os multiplicadores desta versão.
~~~

**Output obtido:**

> **Inconsistências identificadas (além das óbvias):**
> - Fator de peso: v1 usa 1.0/1.2/1.5 por faixa de peso; v2 usa 1.0/1.15/1.4 — a v2 **reduz** o fator de peso nas faixas superiores.
> - Multiplicadores regionais: todos sobem na v2 (Sul +8,3%, Sudeste +10%, Centro-Oeste +7,7%, Nordeste +7,1%, Norte +12,5%).
> - Combinação peso×região: como a v2 sobe o regional mas desce o peso, o efeito líquido não é uniforme — uma leitura parcial de qualquer uma das duas tabelas isoladamente pode levar a uma conclusão errada sobre qual versão é "mais cara".
> - Prazo de entrega: +2 dias úteis (v1) vs. +3 dias úteis (v2) — mudança direta, com impacto em promessa de SLA.
> - Desconto de volume: a mudança mais perigosa e menos óbvia. Na v1 não existe desconto automático — é negociação caso a caso formalizada em aditivo. Na v2, existe desconto automático e escalonado (5% a partir de 8 fretes/mês; 10% a partir de 15/mês). Não é uma correção de valor, é uma mudança de modelo de política comercial, escondida em texto corrido e não em tabela.
> - Referência cruzada à PROC-043: v1 cita sem ressalva; v2 adiciona nota de que a PROC-043 está em revisão pelo Compliance — sinal de que a cadeia de documentos referenciados também está instável.
>
> **Indicação de vigência:** nenhum dos dois cabeçalhos de "Status" indica vigência formal. Porém, a Seção 5 (Disposições Transitórias) da v2 contém uma regra de transição real por data de chamado (antes de 01/12/2023 usa v1; depois usa v2) — que contradiz o próprio campo de Status da v2, nunca atualizado para refletir isso. Como qualquer chamado atual é posterior a 01/12/2023, a v2 já deveria ser a única vigente na prática, mas isso nunca foi comunicado como decisão fechada. Além disso, o escopo da expressão "multiplicadores da versão anterior" na Seção 5 é ambíguo (cobre só a tabela regional, ou também peso/prazo/desconto?), e a regra de transição só existe na v2 — a v1 não a documenta.
>
> **Risco prático em RAG sem tratamento:** um pipeline que indexe os dois documentos por chunk pode compor uma resposta com multiplicador regional de uma versão e fator de peso de outra (valor que não corresponde a nenhum dos dois documentos oficiais); pode negar um desconto automático já existente na v2 usando só a v1; pode aplicar a v2 a um caso que ainda estivesse na janela de transição; ou pode dar respostas diferentes para a mesma pergunta em sessões distintas por variação de retrieval — risco especialmente grave em atendimento, pois o cliente pode comparar protocolos e contestar cobrança.
>
> **Sugestão de resolução (produto, não técnica):** tratar isso como achado bloqueante a resolver **antes** do discovery, não como pauta de discovery; levar à Diretoria Comercial uma pergunta direta pedindo confirmação formal de que a Seção 5 da v2 substitui a v1 a partir de 01/12/2023; esclarecer o escopo exato da transição; enquanto isso não é decidido, não indexar a v1 no corpus (ou indexar com tag "histórico/não vigente") e usar a v2 como única fonte para cálculo de frete; aproveitar o caso para justificar a exigência de metadado estruturado de vigência (`vigente_desde`/`vigente_até`/`substituido_por`) em qualquer documento normativo que alimente o assistente.

**Análise da qualidade do output:**
- O Claude conseguiu listar todas as inconsistências pedidas — fator de peso, multiplicadores regionais, prazo de entrega e condições de desconto de volume — e ainda identificou uma que eu não tinha nomeado explicitamente no prompt: a contradição entre o campo "Status" da v2 e o conteúdo real da sua própria Seção 5 (disposições transitórias). Também apontou que o efeito líquido da mudança regional×peso não é uniforme, o que evita uma leitura simplista de "v2 é sempre mais cara".
- A sugestão de resolução foi acionável do ponto de vista de produto: propôs uma pergunta formal específica para levar à Diretoria Comercial, uma medida de mitigação de curto prazo (não indexar a v1 até decisão) e uma recomendação estrutural (metadado de vigência), sem recorrer a jargão técnico de RAG na comunicação com stakeholders.

---

## 4. Etapa 3 — Cruzamento com o FAQ

**Decisão de contexto:** Forneci o output consolidado das etapas 1 e 2 junto com o FAQ-Atendimento completo, pedindo ao Claude que cruzasse especificamente o Item 8 (frete especial) com a contradição já mapeada.

**Prompt utilizado:**

~~~text
Nas etapas anteriores, você analisou os metadados dos 5 documentos e depois fez uma análise profunda comparando PROC-042 e PROC-042-v2. Aqui está um resumo do que você encontrou na etapa anterior:
[Cole aqui a resposta que o Claude te deu na Etapa 2 — as inconsistências identificadas, a avaliação de risco, e a sugestão de resolução]
Agora vou te passar o conteúdo completo do FAQ-Atendimento, o documento informal mantido pelo time de suporte. Quero que você cruze especificamente o Item 8 desse FAQ (que fala sobre frete especial) com as inconsistências que identificamos entre PROC-042 e PROC-042-v2. Responda:
O FAQ resolve a ambiguidade entre as duas versões, confirma que ela existe sem resolver, ou introduz uma prática paralela que não está em nenhum dos dois documentos formais?
Se o FAQ sugere uma regra prática (ex: "na dúvida, use a mais recente"), essa regra é consistente com a seção 5 (Disposições Transitórias) do PROC-042-v2, ou contradiz ela?
Que risco isso cria se o assistente de IA aprender a resolver ambiguidades "copiando" o comportamento informal do FAQ em vez de seguir uma regra de negócio formalmente definida?
Considerando os 47 itens do FAQ (dos quais só alguns foram selecionados aqui), que tipo de instrução deveríamos dar ao pipeline de RAG sobre como tratar o FAQ como fonte — nunca usar, usar só como último recurso, usar com aviso ao atendente?

FAQ-Atendimento :
# FAQ-Atendimento — Perguntas Frequentes do Time de Suporte
**Versão:** Não controlada
**Última atualização:** Diversas (documento colaborativo)
**Responsável:** Nenhum responsável formal — mantido informalmente pelo time de atendimento
**Classificação:** Documento informal — NÃO validado por Compliance ou Operações. Representa o conhecimento prático do time, mas pode conter informações desatualizadas ou imprecisas.
Aviso interno: Este FAQ foi criado organicamente pelo time de atendimento ao longo de 2 anos. As respostas refletem a experiência prática dos atendentes, mas NÃO foram validadas contra os documentos oficiais (POL, PROC, SLA). Use com cautela e sempre confirme informações críticas na documentação normativa.
## Perguntas selecionadas (das 47 do documento original)
### Item 3 — "Cliente perguntou se pode devolver carga perigosa. O que respondo?"
Na prática, a gente orienta o cliente a ligar no ramal 4500 (Gestão de Riscos). Oficialmente não pode pelo processo padrão, mas já tiveram casos em que o pessoal de Riscos autorizou exceção. Então não diga que é impossível — diga que precisa de tratamento especial.
### Item 8 — "Como funciona o frete especial?"
Acima de 500kg, aplica a tabela de multiplicadores por região. Cuidado: existem duas versões da PROC-042. A mais recente tem multiplicadores mais altos. Na dúvida, use a mais recente (v2), mas se o cliente reclamar do valor, pode ser que o contrato dele ainda esteja na tabela antiga.
### Item 15 — "Cliente diz que é Platinum. Existe esse tier?"
Não existe tier Platinum na NovaTech. Às vezes o cliente confunde com outra transportadora ou com o programa de fidelidade antigo que foi descontinuado em 2022. Oriente que nossos tiers são Gold, Silver e Standard e peça o número do contrato para verificar.
### Item 22 — "Cliente quer saber sobre seguro de carga. O que falar?"
A NovaTech oferece seguro de carga como adicional. O valor é 0,3% do valor declarado da mercadoria para cargas padrão e 0,8% para cargas perigosas. Detalhe: isso vale para contratos a partir de 2023. Contratos mais antigos podem ter percentuais diferentes — confirme com o Comercial.
### Item 27 — "O tracking mostra 'em trânsito' há 5 dias. O que faço?"
Depende da rota. Rotas para o Norte podem levar até 10 dias úteis. Para Sul/Sudeste, mais de 3 dias parado é estranho. Abra um chamado de rastreamento e classifique como prioridade alta se for Gold ou se o valor da carga for acima de R$ 50.000.
### Item 32 — "Pode enviar carga perigosa com frete expresso?"
Sim, mas precisa de autorização do Compliance e a documentação ANTT tem que estar atualizada. Na prática, demora uns 2 dias para conseguir a autorização, então o 'expresso' acaba não sendo tão expresso. Avise o cliente sobre isso.
### Item 38 — "Cliente quer saber a política para carga que chegou danificada."
Carga danificada em trânsito tem processo diferente de devolução. O cliente precisa registrar a ocorrência em até 48h após o recebimento, com fotos e laudo se possível. A NovaTech investiga e, se comprovada responsabilidade nossa, reembolsa integralmente. Mas isso passa pelo Jurídico, não pelo atendimento normal — encaminhe para o e-mail sinistros@novatech.com.br.
### Item 41 — "Qual a diferença entre SLA de resposta e SLA de resolução?"
Resposta é quando a gente dá o primeiro retorno ao cliente (mesmo que seja 'estamos verificando'). Resolução é quando o problema é efetivamente resolvido. O Gold tem 2h de resposta e 24h de resolução. Silver é 4h e 48h. Standard é 8h e 72h. Para incidentes críticos, os prazos são menores — veja a tabela SLA-2024.
### Item 45 — "O cliente quer desconto no frete. Posso dar?"
Atendente não tem autonomia para dar desconto. Para clientes com mais de 10 fretes especiais por mês, existe desconto automático na tabela (veja PROC-042). Para outros casos, encaminhe ao Comercial com justificativa.
~~~

*Nota: no lugar do placeholder "[Cole aqui a resposta que o Claude te deu na Etapa 2...]", foi colado o texto integral do output da Etapa 2, reproduzido na Seção 3 deste documento.*

**Output obtido:**

> **O FAQ resolve, confirma ou introduz prática paralela?** As três coisas ao mesmo tempo. Confirma a ambiguidade (menciona explicitamente que existem duas versões da PROC-042). Tenta resolver via heurística própria de recência ("na dúvida, use a mais recente"). E introduz uma prática paralela não documentada em nenhum PROC formal: a ideia de que o multiplicador aplicável pode depender do **contrato do cliente** ("se o cliente reclamar do valor, pode ser que o contrato dele ainda esteja na tabela antiga") — critério que não existe na Seção 5 da v2, que usa exclusivamente data do chamado.
>
> **Consistência com a Seção 5 da v2:** contradiz, de forma sutil. A Seção 5 usa data do chamado como critério objetivo; o FAQ usa "use a mais recente" (que hoje dá o mesmo resultado prático, mas por um motivo diferente) e depois abre uma exceção por contrato do cliente que não tem respaldo em documento nenhum — uma terceira regra não formalizada, que parece consistente na superfície mas não é.
>
> **Risco se a IA "copiar" o comportamento do FAQ:** (1) herdaria uma exceção por contrato sem critério verificável, podendo especular sobre o contrato do cliente sem acesso ao sistema que confirma isso; (2) herdaria "use a mais recente" como regra de negócio no lugar da regra formal real (data do chamado) — inofensivo hoje porque coincidem, mas seria a regra errada no primeiro caso de exceção real (ex: chamado antigo ainda em processamento, ou uma futura v3); (3) efeito de "autoridade emprestada" — ao sintetizar a informação numa resposta ao cliente, a IA perde o aviso de cautela que o FAQ carrega no documento original, entregando uma afirmação com a mesma confiança de algo vindo da POL-001.
>
> **Tratamento recomendado do FAQ como fonte para o pipeline de RAG:** nem "nunca usar" nem "usar livremente" — uma terceira categoria. (a) Nunca usar como fonte de valores numéricos, regras de decisão ou políticas formais numa resposta ao cliente (cobre justamente o Item 8, além do 22 e 41). (b) Usar apenas como sinal de intent / padrão de pergunta frequente, nunca como fonte de resposta direta (ex: Item 15 sobre tier "Platinum" é ótimo para reconhecer o padrão de confusão do cliente, não para responder). (c) Tratar como fonte de descoberta de gaps documentais para o discovery — vários itens (3, 27, 32, 38, 45) revelam processos que não existem em nenhum dos 5 documentos normativos mapeados. (d) Se usado como fallback de última instância, a resposta deve sempre transferir para humano, nunca afirmar como fato.
>
> **Achado adicional (não pedido, mas relevante):** o próprio FAQ se contradiz entre itens — o Item 45 cita o threshold de 10 fretes/mês (regra da v1, e ainda descrita incorretamente como "automática", quando na v1 é negociação por aditivo), enquanto o Item 8 recomenda usar a v2 "na dúvida". Ou seja, nem o próprio time de atendimento é internamente consistente sobre qual versão vale — reforçando que o FAQ não pode ser fonte de regra de negócio.

**Análise da qualidade do output:**
- O cruzamento revelou algo que as etapas 1 e 2 isoladas não mostrariam: o FAQ-Item 8 não só confirma a ambiguidade entre v1/v2 e sugere uma heurística de recência sem respaldo formal na Seção 5 — ele também introduz uma variável (contrato do cliente) que não existe em nenhum documento oficial, e ainda se contradiz internamente com o próprio Item 45 do FAQ (que cita o threshold da v1 como se fosse regra vigente). Essa contradição interna do FAQ é um achado que só emerge ao colocar dois itens do mesmo documento informal lado a lado — reforça, com evidência concreta, o argumento de governança já levantado na Etapa 2.

---

## 5. Riscos Identificados para o Discovery Humano

| # | Risco | Como levar ao discovery |
|---|-------|--------------------------|
| 1 | PROC-042 e PROC-042-v2 coexistem sem hierarquia formal — o assistente pode dar valores de frete divergentes para o mesmo cenário. | Pergunta direta à Diretoria Comercial: a v2 revoga a v1? Coexistem para contextos diferentes? Existe algum documento de revogação que não foi listado? A Seção 5 (disposições transitórias) deve ser considerada a confirmação formal de substituição? |
| 2 | O FAQ-Atendimento reflete práticas informais (ex: "na dúvida, use a mais recente") que podem contradizer a regra formal de transição do PROC-042-v2 (seção 5). | Validar com o time de atendimento e com Comercial se essas práticas informais são sancionadas ou são "gambiarras" naturalizadas que a Diretoria nunca aprovou. |
| 3 | O FAQ introduz um critério não documentado (variação de multiplicador por contrato individual do cliente, Item 8) e se contradiz internamente entre itens (Item 8 vs. Item 45 sobre threshold de desconto) — sinal de que existe conhecimento tácito de exceções contratuais legadas que não está formalizado em nenhum PROC. | Perguntar ao Comercial se existem contratos legados com multiplicadores "congelados" da v1, e se sim, pedir a lista/critério formal — esse conhecimento precisa ser explicitado antes de o assistente poder tratar exceções por contrato com segurança. |

---

## 6. Reflexão — Progressive Disclosure vs. "Tudo de Uma Vez"

**O que teria acontecido se os 5 documentos completos fossem colados de uma vez no primeiro prompt:**

Com todo o conteúdo disponível desde o início, o modelo provavelmente:
- Diluiria a atenção entre documentos de peso muito diferente (um normativo como POL-001 receberia o mesmo "espaço de atenção" que o FAQ informal), dificultando a priorização automática do risco mais crítico.
- Poderia não destacar com a mesma clareza a contradição PROC-042/v2, por ela estar "enterrada" em meio a um volume muito maior de texto (efeito relacionado a *lost in the middle* e *context rot*).
- Geraria uma resposta mais genérica e menos acionável, por falta de foco progressivo — a abordagem em etapas força o modelo (e o analista) a decidir o que é prioritário antes de avançar, em vez de esperar que o modelo faça essa triagem sozinho dentro de um contexto saturado.
- Muito provavelmente não teria revelado a contradição interna do FAQ entre o Item 8 e o Item 45: esse achado só emergiu porque, na Etapa 3, o modelo já tinha em memória de trabalho a análise fina da Seção 5 da v2 (Etapa 2) e pôde comparar cada item do FAQ contra ela com precisão — algo que compete diretamente por atenção com dezenas de outros detalhes se tudo fosse apresentado de uma vez.

A abordagem progressiva, por outro lado, permitiu que cada etapa build em cima da anterior com foco controlado — o orçamento de atenção do modelo foi usado deliberadamente, e não desperdiçado.

---
