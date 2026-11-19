# EngMan Copilot — Triagem Inteligente de CFTV Rodoviário

**FIAP × Motiva | Challenge Integrador | Sprint 01 / Sprint 02**  
1. Identificação do projeto e integrantes

**Projeto:** EngMan Copilot — Triagem Inteligente de CFTV Rodoviário  
**Recorte escolhido:** **B — Operação e Priorização de Campo**  


|Nome completo do integrante|RM|
|-|-|
|Leandro Eiji Hashimoto|576404|
|Nicolas Gomes Alba Aparicio|576175|

## 2\. Problema abordado

As câmeras CFTV rodoviárias podem permanecer conectadas, mas entregar imagem prejudicada por sujeira, obstrução, teias ou desfoque. A percepção do problema depende em grande parte da análise humana do operador do CCO; a abordagem tende a ser reativa. Em paralelo, chuva, neblina e reflexo solar podem produzir imagens degradadas **sem que exista falha física do equipamento**, ocasionando alarmes falsos e potencialmente deslocamentos desnecessários.

O problema para a Engenharia de Manutenção é **priorizar corretamente** sem confundir causa ambiental com falha local persistente e sem deixar de tratar perda de cobertura importante para a segurança viária. O enunciado não informa quantidades reais de alarmes, redução de custo, taxa de falhas ou resultados de IA; **não apresentamos números inexistentes como fatos**. Os 30 ativos e os 50 eventos de teste deste repositório são criados exclusivamente para uso acadêmico.

## 3\. Recorte e personas

**Recorte B — Operação e Priorização de Campo.** Cobre apenas **câmeras CFTV/PTZ rodoviárias**. Foi escolhido por aderir diretamente à dor de falsos positivos de imagem (Tema 2.1) e à necessidade de correlacionar alarmes, período e ambiente antes de sugerir manutenção. Não cobre diagnóstico amplo de todo o parque de ativos nem o Recorte A.

**Persona principal: operador(a) do Centro de Controle Operacional (CCO).** Recebe múltiplos alarmes e precisa de uma tabela concisa, contextualizada e auditável para decidir o que monitorar e o que escalar. Seu risco principal é **ignorar uma falha crítica ou gerar deslocamento por falso positivo**; a IA não substitui o julgamento operacional.

**Persona secundária: coordenador(a) de manutenção.** Revisa prioridade e rascunhos de OS, confere procedimentos e autoriza ou recusa o encaminhamento. A responsabilidade sobre execução física e segurança permanece humana.

## 4\. Escopo positivo e recusas

**Faz:**

* Analisa alarmes **descritos/registrados** de CFTV identificados por TAG, com registros históricos **sintéticos**.
* Compara persistência temporal, câmeras vizinhas e observações ambientais registradas.
* Propõe `falha\\\_local\\\_provavel`, `ambiental\\\_provavel` ou `indeterminado`, sempre com fatos, hipóteses, fonte e nível de confiança.
* Sugere `P1 / P2 / P3` com base na matriz **simulada**, com aprovação humana final; falta de dados → indeterminado.
* Consulta procedimento documental simulado; propõe monitoramento, inspeção ou rascunho de OS.
* Especifica uma **tool para OS simulada**, executável futuramente apenas após confirmação humana explícita; não há integração real nesta Sprint.

**Recusa/não faz:**

* Diagnóstico e OS de PMV, callbox, SAT, pórtico, sensores de solo e outros ativos fora de CFTV.
* Declarar falha física confirmada sem inspeção validada; inventar TAG, modelo, histórico, clima, métricas, normas ou número de OS.
* Realizar reconhecimento visual automático ou acesso a câmeras em tempo real: são **possibilidades futuras**, não capacidades implementadas.
* Emitir laudo, ART ou parecer legal; liberar, bloquear ou intervir fisicamente em equipamento; instruir atividade elétrica/altura sem POP oficial.
* Acessar/tratar dados pessoais de terceiros ou informações corporativas restritas; responder assuntos alheios à triagem de CFTV.

**Frase de recusa fora do escopo:** "Esta consulta está fora do meu escopo. Encaminho para o responsável pela Engenharia de Manutenção."

## 5\. Arquitetura da solução

!\[Fluxograma da arquitetura EngMan Copilot](docs/arquitetura.png)

O diagrama tem versão vetorial em [docs/arquitetura.svg](docs/arquitetura.svg). O desenho representa a **arquitetura planejada**, não fluxo implementado.

1. **Entrada:** operador do CCO envia texto/alarme estruturado com TAG, momento e sintoma.
2. **Guardrail de entrada:** valida escopo CFTV, TAG, privacidade e pedido perigoso/hostil; recusa o que violar as regras.
3. **Montagem do contexto:** compõe system prompt, histórico limitado, trechos RAG filtrados por TAG/data e dados recuperados das tools.
4. **Chamada ao LLM:** Qwen2.5 7B Instruct via Ollama **local**; `temperature=0.1` e `max\\\_tokens=700` propostos, a confirmar em teste.
5. **Decisão de tool:** verifica necessidade de catálogo, histórico FSM, POP, ocorrências CFTV ou observações ambientais.
6. **Execução da tool:** consulta dados simulados e devolve ao contexto; `abrir\\\_ordem\\\_servico` somente mediante confirmação humana, com retorno **simulado**.
7. **Guardrail de saída:** verifica fonte, não alucinação, coerência entre TAGs, classe e prioridade, ausência de dado pessoal, separação entre fato/hipótese e ausência de ação não autorizada.
8. **Resposta + histórico:** apresenta classificação/prioridade/ação proposta/fontes/confiança e atualiza histórico delimitado (até 6 turnos).

**Fluxo com erro:** TAG não localizado, timeout, divergência ou fonte ausente → `indeterminado`, sem OS criada; caso fora do escopo/perigoso → recusa e escalada humana. A tool nunca recebe poder de alterar a câmera real.

## 6\. Stack e justificativa técnica

|Critério|Qwen2.5 7B Instruct — **escolhido**|Llama 3.1 8B Instruct — alternativa|
|-|-|-|
|Custo de uso|Local sem tarifa por chamada; hardware próprio|Local sem tarifa por chamada; hardware próprio|
|Licença|Apache 2.0 (modelo Qwen2.5-7B-Instruct)|Llama 3.1 Community License (condições específicas)|
|Qualidade pt-BR|Multilíngue, **a avaliar nos 17 casos**|Multilíngue, **a avaliar nos 17 casos**|
|Contexto no catálogo Ollama|32K (tag 7B padrão)|128K (tag 8B padrão)|
|Privacidade|Roda local; sem envio obrigatório à nuvem|Roda local; sem envio obrigatório à nuvem|
|Integração e maturidade|Ollama Python/API local e ecossistema documentado|Ollama Python/API local e ecossistema documentado|

|Critério|ChromaDB — **escolhido**|FAISS — alternativa|
|-|-|-|
|Licença/custo|Apache 2.0, gratuito localmente|MIT, gratuito localmente|
|Metadados e filtro|Suporte nativo por TAG, data, tipo|Requer camada adicional de metadados|
|Privacidade|Implantação local|Implantação local|
|Python e documentação|Adequado a RAG e busca filtrada|Robusto em similaridade vetorial; maior trabalho para filtro|

**Trade-off:** o modelo escolhido possui contexto máximo anunciado menor que o alternativo, mas oferece licença permissiva e caminho 100% gratuito com execução local. O ChromaDB simplifica filtros essenciais para o CCO; FAISS pode dar controle adicional sobre índices ao custo de camada própria. Estes são **critérios documentados**, não resultados de benchmark.

* **Embeddings:** Sentence Transformers multilíngue local (modelo exato será escolhido após avaliação).
* **Orquestração futura:** chamada direta Ollama, sem necessidade de LangChain no MVP. **Ambiente futuro:** Python 3.10+, VS Code ou Colab/Jupyter apenas após a Sprint 01.
* **Interface futura:** tela conceitual em [docs/mockup\_interface.svg](docs/mockup_interface.svg), sem código.
* **Parâmetros e orçamento:** `temperature=0.1`, `max\\\_tokens=700`, RAG top-4, até 6 turnos; orçamento-alvo de **8.192 tokens** (prompt 3.200 + histórico 800 + pergunta 350 + RAG 2.100 + tools 450 + saída 700 + margem 592); conferir [decisões técnicas](docs/decisoes_tecnicas.md).

## 7\. Base de conhecimento (inventário)

O [inventário](data/knowledge_base/inventario.md) descreve **11 documentos/coleções**, com origem e status. Para tornar a proposta concreta, há exemplos **sintéticos** com **30 câmeras**, **50 registros históricos**, **10 boletins** e **3 POPs documentais**. O enunciado acadêmico foi apenas resumido como contexto; não há dados internos reais da Motiva.

A estratégia de RAG utiliza chunks de 350–550 tokens com sobreposição de 60 tokens, metadados de `doc\\\_id/TAG/tipo/data/status` e filtros estruturados antes de similaridade. Nenhum dado pessoal (placas, imagens, empregado), credencial ou contrato sensível será indexado. TAG, localização e criticidade têm como fonte única o catálogo; ocorrências, os boletins; e regras de prioridade, a matriz didática.

## 8\. System prompt

O [prompt estruturado](prompts/system_prompt.md) contém **PAPEL, OBJETIVO, ESCOPO, REGRAS, FORMATO, TOM e BORDA**, com comentário de justificativa por bloco, dois exemplos **few-shot**, sequência de checagem para falha intermitente, frase de recusa estável, separação fato/hipótese e política para fontes contraditórias e falha de tool. A ordem humana de execução é inegociável.

## 9\. Eval set

O [eval set](evals/sprint1_eval_set.json) possui **17 casos** em **oito categorias**: 2 factuais, 2 instrucionais, 3 ambíguos, 3 de dado inexistente, 2 fora de escopo, 2 jailbreak, 2 de risco operacional e 1 de memória/contexto. Cada caso registra `id`, `categoria`, `entrada\\\_usuario`, `contexto\\\_esperado`, `resposta\\\_ideal`, `criterios\\\_avaliacao`, `deve\\\_recusar`, com histórico simulado quando necessário.

**Importante:** o conjunto é **especificado, não executado**. Em Sprint posterior, a pessoa avaliadora marca cada critério como atendido/não atendido; não há acurácia ou resultado medido nesta entrega.

## 10\. Tools especificadas

As seis funções estão descritas em [tools/tools\_spec.json](tools/tools_spec.json) como contratos de entrada JSON Schema, enumerações e comportamento para erros.

|Tool|Papel|Parâmetros principais|
|-|-|-|
|`consultar\\\_ativo`|Verifica TAG, km, criticidade, cobertura e modelo simulados|`tag`|
|`consultar\\\_historico\\\_falhas`|Recupera histórico FSM sintético filtrado|`tag`, `janela\\\_meses` (12), `tipo\\\_falha?`|
|`abrir\\\_ordem\\\_servico`|**Registra OS somente simulada e com aprovação humana**|`tag`, `tipo\\\_intervencao`, `prioridade`, `descricao`, `janela\\\_inicio`, `janela\\\_fim`, `confirmacao\\\_humana`, `id\\\_aprovador\\\_simulado`|
|`consultar\\\_procedimento`|Recupera POP documental e pré-requisitos de segurança|`tipo\\\_ativo=CFTV`, `tipo\\\_intervencao`|
|`consultar\\\_ocorrencias\\\_cftv`|Consulta alarmes, persistência e vizinhas|`tags`, `janela\\\_minutos`|
|`consultar\\\_condicao\\\_ambiental`|Contextualiza chuva/neblina/glare, sem inventar medições|`km`, `momento`, `raio\\\_km`|

Os quatro primeiros são os **obrigatórios** do enunciado. As duas tools adicionais são necessárias para que o Recorte B não prometa correlação temporal/ambiental sem fonte. Nenhuma ferramenta foi programada nesta Sprint.

## 11\. Riscos e mitigações

A [matriz de riscos e LGPD](docs/riscos_lgpd.md) registra **10 riscos**: alucinação, injection, intervenção insegura, viés de prioridade, dados pessoais, confidencialidade, excesso de confiança, estouro de contexto, falha de tool e fontes contraditórias. Cada risco tem impacto, mitigação específica e ligação rastreável com regra, etapa da arquitetura ou caso de teste. Sem OS real, despacho automático ou intervenção direta.

## 12\. Limitações conhecidas

* Sem conexão a imagens/CCO/FSM reais e sem análise visual automática; classificações são especificações de comportamento a testar.
* Base de dados fictícia; nenhum valor deve orientar operações reais da Motiva.
* Desempenho de modelos em pt-BR, latência, janela efetiva de contexto e parâmetros **não foram medidos**.
* Acesso local não elimina exigências futuras de controle de acesso, segurança viária, governança LGPD e atualização de POPs.
* Ausência de contexto ambiental, fonte divergente ou falha de tool deixa a ocorrência **indeterminada**.
* Frequências do plano e prioridades são **acadêmicas**, não instruções oficiais.

## 
