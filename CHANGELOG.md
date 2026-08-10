# Changelog

## 0.8.0 — 2026-08-12

- Gate explícito de visão semântica (`required`, `best_effort`, `off`) com providers `sidecar` e `agent_review`; `--require-semantic-vision` falha cedo quando a revisão não existe.
- Inventário visual com classes `visual_asset`, `technical_artifact`, `qr_code`, `logo`, `document_scan`, `photo` e `unknown`, deduplicação por SHA-256, ocorrências por página e zoom de imagens pequenas relevantes.
- Entrega portátil com `processo_completo.md`, `images/index.json`, `paginas_problematicas.md`, `assets/pages/`, manifesto de codificação, ZIP completo e validador de links/assets.
- Contrato `vision_review.schema.json`, opção `--encoding utf-8-sig` e 46 testes sintéticos cobrindo o fluxo visual corrigido.

## 0.7.0 — 2026-08-11

- Entrada universal `AI_ENTRYPOINT.md`, `llms.txt` e roteador JSON para uso por link do GitHub sem leitura integral do repositório.
- Adaptadores mínimos para Manus, Gemini CLI/Gems, Grok, ChatGPT/GPT e Claude.
- Extensão Gemini CLI com `gemini-extension.json` e `GEMINI.md`; instruções prontas para Gems e Grok.
- Empacotamento específico por plataforma, mantendo um único núcleo jurídico e execução de uma tarefa por vez.
- Suíte ampliada para 41 testes com validação do roteamento e da separação de adaptadores.

## 0.6.0 — 2026-08-11

- Gate adaptativo: inspeciona os materiais primeiro e pergunta somente quando houver ambiguidade, conflito ou dado indispensável ausente.
- Execução seletiva por tarefa (`ingest`, `analyze`, `petition`, `deadlines`, `evidence` ou `audit`).
- `--confirm-scope` opcional, relatório cumulativo `relatorio_processual.md` e preservação de uploads anteriores em `versions/`.
- Perguntas determinísticas em scripts/scope_questions.py e regras em references/request_intake_rules.md.

## 0.5.0 — 2026-08-11

- Pacote compatível com upload de Skills no ChatGPT, com SKILL.md na raiz do arquivo .skill.
- Fluxo especializado para PETITION_DRAFT e PROCEDURAL_ANALYSIS.
- Regras e templates para petições, peças, análise processual, citações internas e revisão antes do protocolo.
- Suíte ampliada para 32 testes, incluindo os modos PETITION_DRAFT e PROCEDURAL_ANALYSIS.
- Execução seletiva por tarefa, sem geração automática de relatórios não solicitados.
- Suíte ampliada para 36 testes.

## 0.4.0 — 2026-08-11

- Aplicadas as peças de andamento processual: contrato JSON, relatório de andamento, prazos para revisão, matriz documental, mapa de provas, checklist, minuta de trabalho e conformidade.
- Adicionada proveniência por página, fila de tarefas e bloqueio explícito contra cálculo automático de prazos ou protocolo de minutas.

## 0.3.0 — 2026-08-10

- Correção arquitetural para separar renderização, inspeção técnica, OCR, visão semântica e consolidação.
- Conversão integral agora exige renderização e visão semântica aplicáveis; ausência de provider gera `PARTIAL`.
- Manifesto `1.1` com cobertura física, textual, de renderização e semântica independentes.
- Adicionados `--vision-mode`, `--render-dpi`, sidecar de descrições por página e testes de regressão contra falso `COMPLETE`.
- Melhorias visuais e de cobertura consolidadas na versão 0.4.0.

## 0.2.1 — 2026-08-10

- Créditos de autoria e links oficiais de Lucianum adicionados à documentação e aos metadados.

## 0.2.0 — 2026-08-10

- Markdown estruturado por página com resumo semântico, entidades, termos-chave e blocos/tabelas.
- Inventário de imagens PDF e scans, cópias derivadas, hash, dimensões, localização e OCR opcional.
- Campo `--image-descriptions` para descrições semânticas revisadas por humano ou modelo multimodal autorizado.
- `image_inventory.json`, schemas de página/visual e cobertura visual no manifesto e relatório.
- README, SKILL, template, protocolo visual e validador atualizados.
- Suíte ampliada para 22 testes.

## Créditos

Projeto desenvolvido e mantido por **Lucianum (lucianum7)** — [Instagram](https://www.instagram.com/lucianum/).
