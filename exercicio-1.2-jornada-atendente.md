# Exercício 1.2 — Design de Jornada com Componente de IA

**Papel:** Product Specialist (BA/UX)
**Cenário:** NovaTech — Assistente de IA para atendimento ao cliente
**Autor:** Julia
**Data:** 02/07/2026

---

## 1. Contexto e Insumos do Discovery

- Atendentes hoje abrem em média **4 fontes diferentes** por chamado.
- Distribuição das dúvidas mais comuns: prazos de entrega (35%), regras de frete (25%), política de devolução (20%), outros (20%).
- Em **15%** dos casos, o atendente não encontra resposta e escala para o supervisor.
- Riscos já mapeados na documentação-fonte (Exercício 1.1): contradição entre PROC-042 v1/v2, e FAQ-Atendimento informal e não validado contra os documentos oficiais.

Esses dois últimos pontos ancoram os guardrails e o fluxo de fallback abaixo — não são hipotéticos, são gaps reais identificados na análise anterior.

---

## 2. Fluxo Principal (Caminho Feliz)

| # | Passo | O que o atendente vê |
|---|-------|----------------------|
| 1 | Cliente faz uma pergunta ao atendente | — |
| 2 | Atendente digita a pergunta em linguagem natural no assistente (integrado ao Teams) | Campo de busca conversacional |
| 3 | Assistente busca na base indexada (RAG) e gera a resposta | Indicador de "buscando..." |
| 4 | Assistente retorna a resposta com **citação explícita da fonte** | Resposta em texto + card da fonte (documento, versão, data, trecho usado) |
| 5 | Atendente avalia se a resposta serve para o caso do cliente | Botões: "Usar esta resposta" / "Não resolveu" (→ Fluxo de Fallback) / "Reportar problema" (→ Fluxo de Feedback) |
| 6 | Atendente repassa a resposta ao cliente | — |
| 7 | Atendente marca o chamado como resolvido | Registro automático: pergunta, resposta, fonte, aceite ou não |

**Ponto de decisão:** no passo 5, quem decide se a resposta está adequada é sempre o atendente — o assistente nunca "auto-aprova" a própria resposta.

---

## 3. Fluxo de Fallback

### 3.1. Baixa confiança detectada pelo sistema (antes de responder)

| # | Passo | Quem decide |
|---|-------|-------------|
| 1 | Assistente recupera chunks para a pergunta | — |
| 2 | Sistema avalia confiança: documentos conflitantes OU nenhum chunk relevante acima do threshold | Decisão automática, baseada em regra determinística |
| 3a | Se conflito entre fontes oficiais → assistente **não escolhe sozinho**; expõe ambas as versões e recomenda confirmação | Atendente vê o conflito explícito e a recomendação de escalar |
| 3b | Se nenhuma fonte relevante → assistente diz que não encontrou informação (nunca preenche com conhecimento geral) | Atendente vê sugestão de escalar |
| 4 | Atendente decide: aceitar a limitação e escalar, ou reformular a pergunta | Decisão do atendente, conforme criticidade percebida |

### 3.2. Atendente discorda de resposta com confiança alta

| # | Passo | Quem decide |
|---|-------|-------------|
| 1 | Assistente responde com confiança alta, mas o atendente suspeita de erro/desatualização | — |
| 2 | Atendente clica "Não resolveu" / "Discordo" | Decisão do atendente, com base em conhecimento tácito |
| 3 | Sistema pergunta o motivo (categorias rápidas) | Alimenta o Fluxo de Feedback |
| 4 | Atendente decide resolver sozinho ou escalar | Critério: envolve valor financeiro/prazo contratual/exceção de política → escalar |
| 5 | Caso de discordância é encaminhado para revisão da base de conhecimento | Não fica restrito ao chamado individual |

---

## 4. Fluxo de Feedback e Melhoria Contínua

| # | Passo | Detalhe |
|---|-------|---------|
| 1 | Atendente reporta problema (botão disponível em toda resposta) | Categorias: Incorreta / Desatualizada / Incompleta / Fonte errada |
| 2 | Sistema captura pergunta, resposta, fonte, categoria e comentário livre | Registro estruturado |
| 3 | Feedback entra em fila de triagem humana obrigatória | Decisor: dono da documentação-fonte + Product Specialist/curador da base |
| 4 | Triagem classifica a causa: erro de retrieval, erro de geração, gap real, ou documentação-fonte errada na origem | Define onde a correção deve acontecer |
| 5a | Erro de retrieval/geração → ajuste no pipeline/prompt | Ciclo rápido, técnico |
| 5b | Gap ou erro na fonte → demanda para a área dona do documento corrigir/formalizar antes de reindexar | Ciclo mais lento, mas resolve a causa raiz |
| 6 | Documento corrigido → reindexação → teste de regressão do caso original | Fecha o loop |
| 7 | Métrica acumulada por categoria e por documento-fonte | Prioriza qual parte da documentação revisar primeiro |

**Ponto de decisão central:** feedback do atendente nunca corrige a base automaticamente — abre um processo de triagem humana.

---

## 5. Guardrails de Comportamento do Assistente (versão refinada)

### 5.1. Teste de rigor (Passo 2 — refinamento crítico)

Antes de fechar a versão final, cada guardrail foi testado contra um cenário em que segui-lo parece, à primeira vista, pior para o atendimento do que quebrá-lo — e contra a pergunta "o que precisa existir fora do assistente para essa tensão não travar a operação?".

| Guardrail | Cenário de tensão | Por que se mantém mesmo assim | O que precisa existir fora do assistente |
|---|---|---|---|
| **G1** (conflito entre fontes formais) | Cliente com carga já em rota, cobrando o valor do frete na hora, sexta 17h50, supervisor ocupado. O assistente "travar" no conflito PROC-042/v2 parece não resolver nada. | Escolher automaticamente (ex: sempre a versão mais recente) seria o assistente tomando uma decisão de política comercial sem mandato — e a v2 pode nunca ter sido formalmente aprovada. Errar aqui é dinheiro real e disputa contratual. | Canal de escalação com SLA curto (ex: resposta em até 5 min) para bloqueios de conflito de fonte com cliente aguardando, e decisão formal da Diretoria Comercial sobre qual versão está vigente — dívida de governança que o assistente só torna visível, não resolve. |
| **G2** (FAQ não fundamenta valor operacional) | Pergunta recorrente sem fonte formal, mas com resposta consolidada no FAQ há 2 anos sem reclamação. Escalar parece burocracia desnecessária. | O assistente não pode verificar caso a caso se "esse item do FAQ" é o correto — e o próprio FAQ já é internamente contraditório em itens conhecidos. Abrir exceção quando "parece razoável" anula a proteção do guardrail no caso em que o FAQ está errado. | Processo de promoção de conhecimento tácito a documento formal: itens do FAQ usados com frequência e nunca contestados devem ser levados à área dona para formalização — responsabilidade do Product Specialist/curador, alimentada pelas métricas de "perguntas sem fonte formal" do Fluxo de Feedback. |
| **G3** (cadeia de referência instável) | Pergunta sobre carga perigosa, com o caminhão prestes a sair; esperar confirmação do Compliance sobre a PROC-043 (em revisão) parece inviável. | É o domínio de maior custo de erro possível (risco físico/regulatório) — exatamente onde ceder "só essa vez" por pressão de tempo é o padrão clássico de acidente. Responder com confiança plena sobre uma referência que a própria empresa sabe instável empresta autoridade que o assistente não tem. | Canal de resposta rápida do Compliance para bloqueios operacionais de carga perigosa, com SLA diferenciado (ex: 30 min em horário comercial + plantão fora dele) — sem isso, o guardrail evita que o assistente erre, mas não resolve a necessidade operacional real. |

**Padrão comum:** os três guardrails devolvem a decisão a um humano com mandato — mas isso só funciona se existir uma rota humana **rápida o suficiente**. Um guardrail sem SLA de escalação correspondente é, na prática, um convite a ser contornado.

**Gap identificado:** G1 e G3 resolvem a ambiguidade da *fonte* (o que o assistente pode ou não afirmar), mas não a ambiguidade da *ação* — um atendente novo que recebe "⚠️ Encontrei duas versões conflitantes" pode não saber se deve escalar, travar ou decidir por conta própria. Sem uma ação obrigatória associada ao alerta, o risco que G1 evita (decisão errada tomada sem mandato) simplesmente se desloca do assistente para o atendente menos experiente. Isso deu origem ao G4.

### 5.2. Versão final (formato de especificação)

1. **O assistente NUNCA** escolhe sozinho uma versão entre documentos-fonte conflitantes sobre o mesmo parâmetro operacional (frete, prazo, multiplicador — ex: PROC-042 vs. PROC-042-v2). **SEMPRE** expõe as duas fontes conflitantes, com trecho e referência de cada uma, e aciona a decisão explícita do G4.
2. **O assistente NUNCA** cita o FAQ-Atendimento, ou qualquer fonte sem dono formal e sem validação contra a documentação oficial, como fundamento de valores de frete, prazos de SLA ou regras de política. Quando a única fonte disponível for informal, **SEMPRE** trata o caso como "sem fonte formal disponível" e aciona a decisão explícita do G4.
3. **O assistente SEMPRE** sinaliza quando a fonte usada depende de um documento referenciado marcado como em revisão, sem vigência definida, ou sem indicação de status (ex: PROC-043 citado dentro do PROC-042-v2) — mesmo quando a resposta parece resolvida. A marcação de instabilidade não é opcional com base na confiança aparente da resposta.
4. **(Guardrail de processo, não só de conteúdo)** Sempre que G1, G2 ou G3 forem acionados, o assistente **NUNCA** se limita a exibir um aviso — **SEMPRE** apresenta ao atendente uma decisão binária e acionável ("este caso muda o que você diz ao cliente? Sim → escalar, com contexto pré-preenchido / Não → prosseguir por sua conta"). Toda decisão tomada nesse ponto é registrada de forma rastreável, para que padrões problemáticos (ex: atendentes novos escolhendo "prosseguir" em casos que deveriam ser escalados) apareçam nas métricas de qualidade — e não fiquem invisíveis dentro do chamado individual.

---