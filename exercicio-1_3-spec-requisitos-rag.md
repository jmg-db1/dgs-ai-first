# Exercício 1.3 — Especificação de Requisitos de RAG do Ponto de Vista do Produto

**Papel:** Product Specialist (BA/UX)
**Cenário:** NovaTech — Assistente de IA para atendimento ao cliente
**Autor:** Julia
**Data:** 02/07/2026

---

## 1. Contexto

Especificação de requisitos de produto para o pipeline de RAG do assistente de atendimento da NovaTech. O objetivo é definir, de forma **não técnica mas precisa e testável**, o comportamento esperado do assistente quanto a fontes de dados, tratamento de contradições, ausência de resposta, atualização e rastreabilidade.

A especificação está ancorada em gaps reais identificados nos exercícios anteriores: a contradição PROC-042 v1/v2 sem hierarquia formal, e o FAQ-Atendimento informal e não validado.

Este documento registra os **3 estágios de iteração** exigidos pelo exercício: versão inicial → revisão crítica do Claude → versão final refinada.

---

## 2. Estágio 1 — Especificação Inicial (v1)

**Prompt utilizado:**

~~~text
Sou Product Specialist de um projeto de assistente de IA (RAG) para atendimento ao cliente de uma transportadora (NovaTech). O pipeline funciona assim: documentos são divididos em chunks, transformados em embeddings, armazenados num banco vetorial, e recuperados por similaridade quando o atendente faz uma pergunta; o LLM gera a resposta usando os chunks recuperados como contexto.

Já identificamos gaps reais na documentação-fonte: duas versões de um mesmo procedimento (PROC-042 v1/v2) coexistindo sem hierarquia formal, e um FAQ interno informal que não deve ser citado como fonte de valores operacionais.

Ajude-me a escrever uma especificação de requisitos de produto — não técnica, mas precisa e testável pelo QA — cobrindo:

1. Fontes de dados: quais tipos de documento devem ser indexados, e como tratar documentos obsoletos ou sem indicação formal de vigência (excluir da base? marcar com aviso? bloquear até formalização?).
2. Documentos contraditórios: comportamento esperado quando duas fontes oficiais sobre o mesmo tema dão respostas diferentes (não deve escolher uma versão sozinho — deve expor o conflito).
3. Ausência de resposta: comportamento quando nenhuma fonte cobre a pergunta (nunca completar com conhecimento geral do modelo; sempre declarar explicitamente que não encontrou).
4. Atualização: prazo máximo aceitável entre a publicação/revisão de um documento e sua disponibilidade nas respostas do assistente.
5. Rastreabilidade: toda resposta deve citar o documento-fonte, a versão e, se possível, o trecho exato usado — defina o nível mínimo de detalhe exigido.

Escreva cada requisito no formato "O assistente deve/não deve..." de forma que um QA consiga verificar objetivamente se foi atendido.
~~~

**Saída obtida:**

> ## Especificação de Requisitos — Governança de Fontes no Pipeline de RAG
> ### Assistente de Atendimento ao Cliente — NovaTech (v1.0)
>
> **Tipo de documento:** Especificação de requisitos de produto (não técnica), testável por QA
>
> ### 0. Propósito e Escopo
>
> Esta especificação define o comportamento esperado do assistente de IA (RAG) de atendimento ao cliente da NovaTech em relação à **governança das fontes documentais** que alimentam suas respostas. Não descreve arquitetura técnica (chunking, embeddings, banco vetorial) — descreve **o que o assistente deve e não deve fazer**, de forma verificável por um QA sem conhecimento de engenharia de IA.
>
> ### 1. Fontes de Dados e Documentos Obsoletos
>
> **RF-01** — O assistente deve indexar apenas documentos que possuam classificação formal de tipo (Política, Procedimento, Tabela de SLA) e responsável formal identificado no cabeçalho.
> *Critério de aceite:* Dado um documento sem campo "Responsável" preenchido, o assistente não deve retorná-lo como fonte em nenhuma resposta.
>
> **RF-02** — O assistente não deve indexar documentos classificados como "informais" ou "colaborativos" (ex: FAQs mantidos por equipe, sem validação formal) na mesma base usada para gerar respostas factuais ao cliente ou ao atendente.
> *Critério de aceite:* Perguntando por um valor operacional (prazo, multiplicador, tarifa) que só exista em um documento informal, o assistente deve responder que não encontrou fonte oficial — nunca deve citar o documento informal como base do valor.
>
> **RF-03** — Quando um documento não possuir indicação formal de vigência (nem "vigente", nem "obsoleto", nem data de revogação), o assistente deve tratá-lo como bloqueado para uso em respostas que envolvam valores numéricos, prazos ou condições comerciais, até que a vigência seja formalizada.
> *Critério de aceite:* Um documento sem campo de vigência preenchido não deve aparecer como fonte única de uma resposta com valor numérico.
>
> **RF-04** — O assistente não deve inferir vigência a partir de sinais indiretos no corpo do texto (ex: uma seção de "disposições transitórias" que sugere que um documento é mais atual) quando o campo formal de status do próprio documento contradiz ou não confirma essa inferência.
> *Critério de aceite:* Se o campo "Status" de um documento diz que ele coexiste sem hierarquia com outro, o assistente não deve tratar esse documento como vigente só porque uma cláusula interna sugere isso.
>
> ### 2. Documentos Contraditórios
>
> **RF-05** — O assistente deve, antes de responder, verificar se mais de uma fonte oficial indexada cobre o mesmo tema com valores ou regras diferentes.
> *Critério de aceite:* Testável via conjunto de perguntas com respostas conhecidas em múltiplas fontes conflitantes — o assistente deve detectar a divergência em 100% dos casos do conjunto de teste antes de formular a resposta final.
>
> **RF-06** — Quando houver conflito detectado entre fontes oficiais sobre o mesmo tema, o assistente não deve escolher uma versão para responder ao atendente. Deve expor explicitamente ao atendente que existe divergência entre fontes, citando ambas.
> *Critério de aceite:* A resposta deve conter, no mínimo: (a) menção explícita de que há mais de uma fonte com valores diferentes, (b) identificação de cada fonte e seu valor, (c) instrução clara ao atendente de que uma confirmação formal é necessária antes de informar o valor ao cliente.
>
> **RF-07** — O assistente não deve usar frequência de menção, recência de data de emissão, ou qualquer heurística própria para decidir qual fonte prevalece, exceto quando essa regra de prevalência estiver formalmente documentada e sem ambiguidade em uma fonte oficial.
> *Critério de aceite:* Em teste com dois documentos conflitantes onde nenhuma regra de precedência está formalmente clara, o assistente deve tratar como conflito não resolvido, não como decisão automática por recência.
>
> **RF-08** — Quando uma fonte oficial contiver uma regra de transição/precedência, mas o campo de status/vigência da própria fonte não confirmar essa regra como decisão institucional fechada, o assistente deve tratar isso como conflito interno da própria fonte, sujeito ao mesmo tratamento do RF-06.
> *Critério de aceite:* Caso espelhando PROC-042-v2 Seção 5: o assistente não deve declarar "a v2 é a vigente porque a Seção 5 diz que chamados atuais a usam" como fato consolidado.
>
> ### 3. Ausência de Resposta
>
> **RF-09** — Quando nenhuma fonte indexada cobrir a pergunta feita, o assistente não deve complementar a resposta com conhecimento geral do modelo, inferências plausíveis, ou extrapolações a partir de temas relacionados.
> *Critério de aceite:* Em um conjunto de teste com perguntas deliberadamente fora da cobertura documental, 0% das respostas podem conter valores, prazos ou regras não literalmente rastreáveis a um chunk recuperado.
>
> **RF-10** — O assistente deve declarar explicitamente, em linguagem clara para o atendente, que não encontrou fonte oficial para a pergunta, sempre que esse for o caso.
> *Critério de aceite:* A resposta deve conter uma frase inequívoca de ausência de cobertura — não deve haver respostas vagas ou que pareçam responder sem de fato citar fonte.
>
> ### 4. Atualização e Prazo de Disponibilidade
>
> **RF-11** — O assistente não deve responder com base em uma versão de documento anterior à última revisão publicada e formalmente vigente, uma vez que essa nova versão tenha completado o prazo de disponibilização definido pela operação.
> *Critério de aceite:* o prazo máximo entre a publicação/revisão formal de um documento e sua disponibilidade nas respostas do assistente é de [X dias úteis — parâmetro a ser confirmado; sugestão inicial: 5 dias úteis].
>
> **RF-12** — Quando um documento antigo for formalmente substituído (com vigência confirmada), o assistente não deve continuar citando a versão antiga como fonte válida após o prazo definido em RF-11 — mesmo que ela permaneça indexada para fins de auditoria/histórico.
> *Critério de aceite:* Após o prazo de RF-11 e confirmação formal de substituição, testes repetindo a mesma pergunta não devem retornar a versão antiga como fonte da resposta ativa.
>
> ### 5. Rastreabilidade
>
> **RF-13** — Toda resposta que utilize conteúdo de uma fonte documental deve citar explicitamente: (a) o nome/código do documento, (b) a versão do documento, (c) a seção ou item específico usado.
> *Critério de aceite mínimo:* citação no formato `[Documento — Versão — Seção/Item]` (ex: `PROC-042-v2 — Versão 2.0 — Seção 2.1`).
>
> **RF-14** — O assistente não deve apresentar uma resposta como baseada em documentação oficial sem citação rastreável, mesmo quando o conteúdo estiver correto.
> *Critério de aceite:* Em um conjunto de N respostas corretas, 100% devem conter citação no formato do RF-13. Correção de conteúdo não compensa ausência de citação.
>
> **RF-15** — Quando a resposta envolver valores numéricos ou condições comerciais, o assistente deve reproduzir o trecho ou valor exato do documento-fonte, não uma paráfrase que possa alterar a magnitude ou condição.
> *Critério de aceite:* Comparação byte-a-byte entre o valor citado na resposta e o valor no documento-fonte. Divergência de valor numérico = falha automática no teste.
>
> *(A v1 incluía ainda uma matriz-resumo para QA e uma seção de pendências para discovery, apontando o prazo de RF-11, a decisão formal v1/v2 e a política de uso do FAQ como itens em aberto.)*

---

## 3. Estágio 2 — Revisão Crítica do Claude

**Prompt utilizado:**

~~~text
Considere a especificação de requisitos que produzimos no passo anterior. Agora quero que você atue como revisor crítico, não como coautor. Analise essa especificação e aponte, especificamente:

1. Ambiguidades: algum requisito está escrito de um jeito que duas pessoas poderiam interpretar de forma diferente? Aponte a frase exata e explique a ambiguidade.
2. Não-testabilidade: algum requisito não é verificável objetivamente por um QA (ex: usa termos vagos como "rapidamente", "adequado", "quando possível" sem critério numérico ou binário)? Reescreva esses requisitos de forma testável.
3. Gaps: existe algum cenário relevante do domínio de logística/atendimento que a especificação não cobre? Por exemplo: e se o documento contraditório tiver 3 versões, não só 2? E se a base de dados ficar temporariamente indisponível? E se um documento for corrigido mas ainda estiver "em revisão pelo Compliance" (como a PROC-043 citada dentro do PROC-042-v2)?
4. Excesso de rigor / overengineering: existe algum requisito que impõe um nível de detalhe ou rigidez desnecessária, que custaria caro implementar sem agregar valor real ao atendente?
5. Conflito entre requisitos: algum requisito entra em tensão com outro (ex: o requisito de atualização rápida vs. o requisito de nunca responder sem confirmação formal de vigência)?

Não reescreva a especificação inteira agora — apenas liste os problemas encontrados, organizados por categoria (1 a 5), para eu decidir o que incorporar na versão final.
~~~

O Claude atuou como **revisor crítico** (não coautor), apontando problemas organizados em 5 categorias. Resumo dos principais achados:

**Ambiguidades:**
- RF-01: o "E" entre classificação de tipo e responsável formal pode ser lido como conjunção obrigatória ou lista de critérios independentes — resultados de teste diferentes para o mesmo documento incompleto.
- RF-02: "na mesma base" é ambíguo entre banco vetorial e escopo lógico de recuperação; se o FAQ estiver numa collection separada mas ainda acessível em tempo de resposta, o requisito foi cumprido ou violado?
- RF-03: "respostas que envolvam valores numéricos, prazos ou condições comerciais" não decide se uma pergunta procedimental (sem número) no mesmo documento sem vigência é bloqueada ou não.
- RF-05: "mesmo tema" não tem definição operacional de granularidade — dois documentos sobre frete especial para produtos diferentes são "mesmo tema"?
- RF-06: "instrução clara ao atendente" é subjetivo, sem critério que separe uma instrução "clara" de uma "vaga".
- RF-15: "trecho ou valor exato" trata reprodução de valor (trivial de verificar) e reprodução de trecho de texto (que colide com política de paráfrase/direitos autorais) como se fossem a mesma coisa.

**Não-testabilidade:**
- RF-05: "verificar" é um processo interno não observável externamente — o QA só observa o output final; requisito redundante com RF-06.
- RF-07: "sem ambiguidade" é justamente o tipo de termo vago que a spec deveria eliminar; reintroduz julgamento humano.
- RF-11: "[X dias]" é placeholder, não critério — nenhum QA consegue aprovar/reprovar nada contra ele.
- Padrão geral: RF-05, RF-06 (parte final) e RF-07 descrevem *processo mental* esperado em vez de *saída observável*.

**Gaps:**
- Múltiplas versões (3+): toda a Seção 2 assume cardinalidade 2 ("citando ambas"); não cobre v3 nem discordância entre documentos de tipos diferentes.
- Indisponibilidade técnica da base vetorial (timeout, banco fora do ar) ≠ ausência de cobertura documental — do ponto de vista do atendente parecem iguais, mas são categorias de erro diferentes.
- Documento vigente **sob revisão ativa** (caso PROC-043, "em revisão pelo Compliance"): terceiro estado, distinto de "vigente" e de "sem indicação de vigência" — não previsto por RF-03.
- Correção parcial/seccional de documento: RF-11/RF-12 tratam substituição como evento binário e único.
- Escalonamento humano: a spec diz o que declarar, mas não define se conflito exposto deve gerar ação de governança (chamado, notificação, métrica).
- Nível de citação para cliente final vs. atendente: a Seção 5 assume atendente; falta declarar exposição ao cliente como fora de escopo.

**Overengineering:**
- RF-05 redundante com RF-06 (o comportamento observável já está em RF-06).
- RF-15: exigir reprodução literal de *trecho de texto* é caro e de valor questionável para o atendente, que quer o número certo; reprodução exata de *valor* é razoável e barata.
- RF-04: como escrito, pode bloquear até leituras diretas seguras (ex: "REVOGADO" no título) só porque não estão num campo estruturado.

**Conflitos entre requisitos:**
- RF-03/04 (bloqueio até formalização) vs. RF-11/12 (SLA de atualização): se uma nova versão é publicada mas ninguém formaliza a vigência — o padrão real da NovaTech com v1/v2 — o sistema trava permanentemente.
- RF-02 (FAQ proibido como fonte) vs. RF-06 (expor conflito citando as fontes): e se a única pista de que o conflito existe está no FAQ (como o Item 8 revelou a prática informal)?
- RF-09/10 (ausência total) vs. RF-06 (cobertura conflitante): não modela o meio-termo — cobertura parcial com conflito parcial.

---

## 4. Decisões de Produto Tomadas sobre a Crítica

| Ponto levantado | Decisão |
|-----------------|---------|
| Bloqueio permanente vs. válvula de escape | Adotar válvula de escape: documento sem vigência formal fica bloqueado como fonte, mas dispara escalonamento automático ao dono do documento após prazo definido, registrado como pendência de governança |
| FAQ como pista de conflito | FAQ permanece proibido como fonte de resposta, mas pode alimentar a fila de triagem como sinal interno de investigação. Separar "fonte de resposta" de "sinal de descoberta" |
| Múltiplas versões (3+) e fontes distintas | Generalizar a Seção 2 para cardinalidade N e cobrir discordância entre documentos de tipos diferentes |
| Documento sob revisão ativa (PROC-043) | Nova categoria "vigente sob revisão ativa": citar normalmente, mas com aviso de que o valor pode mudar |
| Falha técnica de retrieval | Separar "não encontrei informação" de "não consigo consultar agora", com mensagens distintas |
| Cobertura parcial com conflito parcial | Responder a parte coberta e expor o conflito só na parte conflitante |
| RF-15 (reprodução literal) | Exigir reprodução exata só de valores; para texto, citar documento + versão + seção |
| RF-05 | Rebaixar a nota de design (comportamento observável já está em RF-06) |
| RF-04 | Permitir leitura direta de declarações explícitas; bloquear só inferências sobre sinais indiretos |
| Testabilidade geral | Reescrever "processos mentais" como saídas observáveis; substituir termos vagos por critérios numéricos/binários |

---

## 5. Estágio 3 — Especificação Final Refinada (v2)

**Prompt utilizado:**

~~~text
Decisões sobre os conflitos e gaps:

- Bloqueio permanente vs. válvula de escape (RF-03/04 vs. RF-11/12): não quero que o sistema trave para sempre quando a governança documental falhar. Adote a válvula de escape: um documento sem vigência formal fica bloqueado como fonte de resposta, mas dispara escalonamento automático para o dono do documento após um prazo definido, registrando isso como pendência de governança. Defina esse prazo como um requisito próprio e testável.
- FAQ como pista de conflito (RF-02 vs. RF-06): o FAQ continua proibido como fonte de resposta, mas pode ser usado como sinal interno para investigação. Ele nunca aparece citado ao atendente como fonte, mas pode alimentar a fila de triagem. Separe explicitamente "fonte de resposta" de "sinal de descoberta".
- Múltiplas versões (3+) e fontes distintas: generalize a Seção 2 para cardinalidade N e cubra também o caso de dois documentos de tipos diferentes que discordem sobre o mesmo dado.
- Documento em revisão por terceiro órgão (caso PROC-043): crie a categoria "vigente sob revisão ativa" — a fonte é citada normalmente, mas com aviso de que está sob revisão e o valor pode mudar.
- Falha técnica de retrieval: separe "não encontrei informação na base" (ausência de cobertura) de "não consigo consultar a base agora" (falha técnica), com mensagens distintas.
- Cobertura parcial com conflito parcial: defina a precedência — quando parte da pergunta é coberta e há conflito só numa parte, o assistente responde a parte coberta e expõe o conflito na parte conflitante.

Decisões sobre rigor:

- RF-15 (reprodução literal): exija reprodução exata apenas de valores. Para texto, basta citar documento + versão + seção, sem reprodução verbatim.
- RF-05: rebaixe de requisito testável para nota de design.
- RF-04: permita leitura direta de declarações textuais explícitas (ex: título "REVOGADO"), bloqueando apenas inferências sobre sinais indiretos/ambíguos.

Decisões sobre testabilidade:

- Reescreva todos os requisitos que descreviam "processo mental" como saídas observáveis.
- Substitua todos os termos vagos por critérios numéricos ou binários.

Agora reescreva a especificação completa, versão final, com cada requisito no formato "O assistente deve/não deve...", numerados, com um critério de aceitação testável logo abaixo de cada um. Marque com um identificador os requisitos que são novos ou que mudaram em relação à versão anterior.
~~~

**Especificação final:**

> ## Especificação de Requisitos — Governança de Fontes no Pipeline de RAG
> ### Assistente de Atendimento ao Cliente — NovaTech — Versão 2.0 (final)
>
> **Legenda de rastreabilidade de versão:** **[MANTIDO]** (sem mudança material) · **[ALTERADO]** (existia na v1.0, modificado por decisão de revisão) · **[NOVO]** (introduzido nesta versão)
>
> ### 0. Propósito e Escopo
>
> Descreve o que o assistente/sistema deve e não deve fazer quanto à governança das fontes documentais, de forma verificável por um QA sem conhecimento de engenharia de IA.
>
> **Fora de escopo (declarado explicitamente):** o nível de detalhe de citação da Seção 6 assume que o destinatário é o **atendente interno**. Exposição direta ao cliente final exige especificação própria.
>
> **Distinção fundamental usada em todo o documento:**
> - **Fonte de resposta** — documento cujo conteúdo pode ser usado para formular a resposta e ser citado ao atendente.
> - **Sinal de descoberta** — documento/item que não pode ser citado como fonte, mas pode disparar alertas internos de governança. Nunca aparece citado ao atendente.
>
> ### 1. Fontes de Dados e Documentos Obsoletos
>
> **RF-01 — [ALTERADO]** O sistema deve indexar como fonte de resposta apenas documentos que atendam simultaneamente: (a) campo de classificação de tipo preenchido **e** (b) campo de responsável formal preenchido no cabeçalho.
> *Critério de aceite:* Dado um documento com **qualquer um** dos dois campos vazio, o assistente não deve retorná-lo como fonte citada. Teste: 3 documentos (um sem tipo, um sem responsável, um sem ambos) — nenhum pode aparecer como fonte.
>
> **RF-02 — [ALTERADO]** O assistente não deve usar documentos informais/colaborativos como **fonte de resposta**, para nenhum tipo de conteúdo. Podem ser usados como **sinal de descoberta** (RF-19).
> *Critério de aceite:* Para pergunta cuja única cobertura esteja em documento informal, a resposta não cita esse documento nem reproduz seus valores; cai em ausência de cobertura (RF-11). Teste: perguntar por valor que só exista no FAQ → declaração de ausência, sem menção ao FAQ.
>
> **RF-03 — [ALTERADO]** Quando um documento não possuir indicação formal de vigência, o sistema deve bloqueá-lo como fonte de resposta para **toda a resposta** — não apenas a parte numérica — até formalização.
> *Critério de aceite:* Documento sem vigência não aparece como fonte em nenhuma resposta. Teste: perguntar por procedimento não-numérico cuja fonte não tem vigência → resposta não cita esse documento.
>
> **RF-04 — [ALTERADO]** O assistente não deve inferir vigência a partir de sinais **indiretos ou ambíguos** no corpo do texto. Pode reconhecer **declarações textuais explícitas e inequívocas** de status (ex: "REVOGADO" no título, "Este documento substitui X") como indicação formal válida.
> *Critério de aceite (binário):* documento com cláusula que só *sugere* atualidade → conflito (RF-06); documento com declaração explícita → tratado conforme a declaração. Teste par a par: (a) estilo PROC-042-v2 → não vigente por inferência; (b) "REVOGADO" no título → obsoleto sem metadado adicional.
>
> **RF-05 — [NOVO]** O sistema deve disparar escalonamento automático para o responsável formal quando um documento estiver bloqueado por ausência de vigência (RF-03) por mais de **10 dias corridos** a partir da indexação, registrando pendência de governança em fila auditável.
> *Critério de aceite (numérico/binário):* em D+9 sem escalonamento; em D+10 há registro na fila identificando documento e dono, e o documento segue bloqueado. Teste com relógio simulado.
>
> ### 2. Documentos Contraditórios (cardinalidade N)
>
> **Nota de design (não testável) — [ALTERADO]** *Para que esta seção seja possível, o sistema precisa comparar as fontes recuperadas antes de responder. Esse comportamento interno não é testado isoladamente; o QA verifica a saída observável de RF-06 em diante. (Substitui o antigo RF-05 da v1, rebaixado de requisito para nota de design.)*
>
> **RF-06 — [ALTERADO]** Quando duas ou mais fontes de resposta (**N versões do mesmo documento, ou N documentos de tipos diferentes**) cobrirem o mesmo dado com valores/regras divergentes, o assistente não deve escolher uma. Deve expor a divergência listando **todas** as fontes.
> *Critério de aceite:* para **cada** fonte (N ≥ 2): (a) documento e versão, (b) valor/regra de cada uma, (c) frase de bloqueio (RF-07). Teste com N=3 e teste com dois tipos distintos (PROC vs. Política discordando de prazo) — todas listadas, nenhuma eleita.
>
> **RF-07 — [ALTERADO]** Em conflito (RF-06), o assistente deve emitir a instrução de bloqueio usando o texto padronizado exato: **"Há mais de uma fonte oficial com valores diferentes para esta pergunta. Confirme com [responsável] antes de informar ao cliente."**
> *Critério de aceite (binário):* a resposta contém a string padronizada. Presença = passa; ausência ou paráfrase = falha.
>
> **RF-08 — [ALTERADO]** O assistente não deve usar recência, número de versão, frequência ou heurística própria para eleger fonte. Precedência só se aplica se **declarada em campo formal** de um documento oficial (não inferida do corpo).
> *Critério de aceite (binário):* sem campo formal de precedência → conflito (RF-06), independentemente de recência; com campo "este documento substitui X" → precedência aplicada. Teste PROC-042-v2: Seção 5 no corpo **não** conta como campo formal → permanece conflito.
>
> **RF-09 — [ALTERADO]** Quando uma cláusula interna sugerir vigência/precedência mas o campo formal de status não confirmar, o assistente deve tratar como conflito interno (RF-06/RF-07) e não deve declarar a leitura como fato consolidado.
> *Critério de aceite (binário):* caso PROC-042-v2 Seção 5 — a resposta não afirma "a v2 é a vigente"; expõe como leitura não confirmada. Afirmação de vigência consolidada = falha.
>
> ### 3. Vigência Sob Revisão Ativa
>
> **RF-10 — [NOVO]** Quando um documento estiver formalmente vigente **mas** contiver ou for referenciado com declaração explícita de revisão por órgão competente (ex: "em revisão pelo Compliance, pode sofrer alterações"), o assistente deve citá-lo normalmente, acrescentando o aviso padronizado: **"Atenção: esta fonte está sob revisão ativa e o valor pode mudar."**
> *Critério de aceite (binário):* caso PROC-043 — resposta que use PROC-043 contém a string de aviso. Uso sem aviso = falha; recusa total de usar (ela é vigente) = falha. Teste: perguntar sobre frete de carga perigosa → cita PROC-043 + aviso.
>
> ### 4. Ausência de Cobertura vs. Falha Técnica
>
> **RF-11 — [ALTERADO]** Quando nenhuma fonte de resposta cobrir a pergunta, o assistente não deve complementar com conhecimento geral/inferência, e deve declarar ausência com o texto padronizado: **"Não encontrei essa informação na documentação oficial disponível."**
> *Critério de aceite:* em conjunto fora de cobertura, 0% das respostas contêm valores não rastreáveis a chunk, e 100% contêm a string. Teste: "Qual o procedimento para frete internacional?" → string exata, sem aproximação.
>
> **RF-12 — [NOVO]** Quando o retrieval **falhar tecnicamente** (timeout, indisponibilidade, erro de embedding), o assistente não deve emitir a mensagem de ausência (RF-11) nem responder por conhecimento próprio. Deve emitir texto distinto: **"Não consigo consultar a base de conhecimento neste momento. Tente novamente em instantes."**
> *Critério de aceite (binário):* simular falha → resposta contém a string de falha técnica, **diferente** da de RF-11. Emitir RF-11 durante falha técnica = falha.
>
> **RF-13 — [NOVO]** Quando a pergunta for **parcialmente coberta**, o assistente deve responder a parte coberta citando a fonte **e** aplicar na mesma resposta o tratamento da parte problemática (RF-06/RF-07 se conflito; RF-11 se ausência). Não deve tratar tudo como ausência total nem como resposta consolidada.
> *Critério de aceite:* pergunta composta (parte A coberta, parte B em conflito) → resposta contém resposta+citação de A e instrução de conflito de B. Teste: "SLA de resolução para cliente Gold e o multiplicador de frete Norte?" → responde A, expõe conflito v1/v2 em B. Omitir A ou o conflito de B = falha.
>
> ### 5. Atualização e Prazo de Disponibilidade
>
> **RF-14 — [ALTERADO]** O sistema deve refletir uma revisão formalmente vigente nas respostas em no máximo **5 dias úteis** após a publicação/vigência formal. O assistente não deve citar versão anterior após esse prazo.
> *Critério de aceite (numérico):* publicar revisão de teste com vigência; dentro de 5 dias úteis as respostas passam a citar a nova versão; em D+6 dias úteis nenhuma resposta ativa cita a anterior. (Documentos sem vigência caem em RF-03/RF-05.)
>
> **RF-15 — [MANTIDO]** Quando um documento for formalmente substituído, o assistente não deve citar a versão antiga como fonte ativa após o prazo de RF-14, ainda que ela siga indexada para auditoria.
> *Critério de aceite:* após substituição formal e prazo de RF-14, a mesma pergunta não retorna a versão antiga, exceto se o atendente pedir o histórico.
>
> ### 6. Rastreabilidade
>
> **RF-16 — [ALTERADO]** Toda resposta que use uma fonte deve citar: (a) nome/código, (b) versão, (c) seção/item, no formato `[Documento — Versão — Seção/Item]`.
> *Critério de aceite (binário):* amostragem de N respostas — 100% no formato exato (ex: `PROC-042-v2 — Versão 2.0 — Seção 2.1`). Falta de qualquer campo = falha.
>
> **RF-17 — [MANTIDO]** O assistente não deve apresentar resposta como baseada em documentação oficial sem citação rastreável, mesmo que o conteúdo esteja correto.
> *Critério de aceite:* em N respostas corretas, 100% contêm citação (RF-16). Correção não compensa ausência de citação.
>
> **RF-18 — [ALTERADO]** Quando a resposta envolver um **valor** (número, prazo, percentual), o assistente deve reproduzir o valor **exato** do documento-fonte. Para **texto/regra descritiva**, basta a citação de RF-16, sem reprodução verbatim.
> *Critério de aceite:* valores — comparação exata; qualquer divergência numérica = falha automática. Texto — presença de citação válida é suficiente. Teste: "multiplicador Norte na v2" → exatamente "1.8"; "condições da devolução" → basta citar POL-001 com seção.
>
> ### 7. Sinais de Descoberta (uso interno, nunca citado ao atendente)
>
> **RF-19 — [NOVO]** O sistema pode usar documentos informais (ex: FAQ-Atendimento) e padrões neles detectados como **sinal de descoberta** para a fila de triagem de governança (ex: alerta quando um item informal contradiz uma fonte oficial). Esses sinais nunca devem aparecer citados ao atendente como fonte.
> *Critério de aceite (binário):* positivo — item de FAQ que contradiz fonte oficial gera registro na fila identificando a divergência; negativo — em nenhuma resposta ao atendente o FAQ aparece citado. Teste duplo: alerta presente na fila interna **e** ausência de citação do FAQ na resposta.
>
> ### 8. Pendências para Discovery
>
> 1. Valor de RF-14 (5 dias úteis) — fechado e testável, mas sujeito a validação com o SLA real de republicação da NovaTech.
> 2. Valor de RF-05 (10 dias corridos) — parâmetro proposto; confirmar com Governança de Conteúdo.
> 3. Decisão formal PROC-042 v1 vs. v2 — até confirmação escrita da Diretoria Comercial, o par cai sob RF-06/RF-07/RF-09.
> 4. Destino dos sinais de descoberta (RF-19) — quem consome a fila e com que SLA não está definido.
> 5. Exposição ao cliente final — fora de escopo; se ocorrer no futuro, a Seção 6 precisa de revisão.
>
> ### 9. Matriz-Resumo (para uso rápido em QA)
>
> | Situação | O assistente NÃO deve | O assistente DEVE | Requisito |
> |---|---|---|---|
> | Doc sem tipo ou sem responsável | Citar como fonte | Bloquear como fonte | RF-01 |
> | Doc informal (FAQ) | Citar como fonte de resposta | Usar só como sinal interno | RF-02, RF-19 |
> | Doc sem vigência formal | Usar em qualquer resposta | Bloquear + escalonar após 10 dias | RF-03, RF-05 |
> | Sinal indireto de vigência | Inferir vigência | Tratar como conflito | RF-04, RF-09 |
> | Declaração explícita ("REVOGADO") | Ignorar | Reconhecer como status válido | RF-04 |
> | Conflito entre N fontes | Eleger uma | Expor todas + frase padrão | RF-06, RF-07, RF-08 |
> | Fonte vigente sob revisão | Recusar ou omitir aviso | Citar + aviso padrão | RF-10 |
> | Nenhuma cobertura | Completar com conhecimento geral | Frase padrão de ausência | RF-11 |
> | Falha técnica de retrieval | Emitir "não encontrei" | Frase padrão distinta | RF-12 |
> | Cobertura parcial + conflito parcial | Tratar como ausência total | Responder coberto + expor conflito | RF-13 |
> | Revisão publicada | Citar versão antiga após prazo | Refletir nova versão em ≤5 dias úteis | RF-14, RF-15 |
> | Toda resposta factual | Responder sem citação | Citar doc + versão + seção | RF-16, RF-17 |
> | Valores numéricos | Parafrasear/alterar valor | Reproduzir valor exato | RF-18 |

---

## 6. Rastreabilidade da Evolução

A revisão crítica não produziu ajustes cosméticos: ela mudou a robustez estrutural da especificação. O ganho mais significativo foi resolver o **conflito de bloqueio permanente**. A v1 descrevia, sem perceber, um sistema que travaria para sempre sempre que a governança documental da NovaTech falhasse — exatamente o padrão real observado com PROC-042 v1/v2, nunca formalizado. A v2 introduz a **válvula de escape** (RF-05): o documento continua bloqueado como fonte, mas o bloqueio deixa de ser um beco sem saída e passa a gerar uma pendência ativa de governança com escalonamento ao dono após 10 dias. O sistema deixa de absorver silenciosamente a falha de processo da empresa e passa a forçar sua resolução.

O segundo avanço foi conceitual: a separação explícita entre **"fonte de resposta"** e **"sinal de descoberta"**. Na v1, o FAQ era simplesmente proibido, o que jogava fora um sinal que o próprio exercício 1.1 havia tratado como valioso — foi o Item 8 do FAQ que revelou a prática informal divergente. A v2 (RF-02 + RF-19) preserva a proibição de citá-lo ao atendente, mas recupera seu valor como gatilho interno de investigação. É uma distinção que só emergiu porque a IA foi usada para *criticar* a própria decisão anterior, não para reforçá-la.

Os demais ajustes seguem a mesma lógica de tornar a spec ao mesmo tempo mais completa e mais testável: a Seção 2 foi generalizada de cardinalidade 2 para N (cobrindo v3+ e discordância entre tipos distintos de documento); criou-se a categoria "vigente sob revisão ativa" (RF-10, caso PROC-043); separou-se falha técnica de ausência de cobertura (RF-12); definiu-se a precedência para cobertura parcial (RF-13); e todos os termos vagos ("instrução clara", "sem ambiguidade", "[X dias]") foram substituídos por strings padronizadas e números verificáveis, enquanto a exigência de reprodução literal foi restrita a valores. A v1 tinha 15 requisitos com várias brechas de interpretação; a v2 tem 19 requisitos, cada um com critério de aceite binário ou numérico — um QA consegue aprovar ou reprovar cada um sem depender de julgamento subjetivo.

---

## 7. Evidências de Uso das Ferramentas

- [ ] Print/export da conversa — Estágio 1 (especificação inicial)
- [ ] Print/export da conversa — Estágio 2 (revisão crítica)
- [ ] Print/export da conversa — Estágio 3 (versão final)
- [x] Este documento consolidado

---

*Documento produzido como parte do Cenário 1 da Trilha de Formação AI First — DGS. Branch: `cenario-1`.*
