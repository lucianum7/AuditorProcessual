# AuditorProcessual — entrada universal para IAs

Este arquivo é o primeiro ponto de leitura quando o repositório for fornecido
por URL. Não leia o repositório inteiro. Identifique a plataforma e a tarefa e
abra somente os caminhos indicados.

## Roteamento por plataforma

| Plataforma | Primeiro arquivo | Próximo passo |
|---|---|---|
| Manus | `SKILL.md` | usar o núcleo em `skills/legal-process-parser/` |
| Gemini CLI | `GEMINI.md` e `gemini-extension.json` | ativar `skills/legal-process-parser/SKILL.md` |
| Gemini Gem | `adapters/gemini/GEM_INSTRUCTIONS.md` | adicionar somente referências necessárias como Knowledge |
| Grok | `adapters/grok/SYSTEM_INSTRUCTIONS.md` | anexar o processo e consultar os arquivos por busca documental |
| ChatGPT/GPT | `skills/legal-process-parser/SKILL.md` | usar o pacote `.skill` ou o núcleo do repositório |
| Claude | `adapters/claude/CLAUDE_INSTRUCTIONS.md` | usar o pacote com `SKILL.md` na raiz |

## Roteamento por tarefa

Depois de escolher a plataforma, leia `routing/task-router.json` e carregue
somente a referência da tarefa solicitada:

- `ingest`: extração integral, preservação e localização;
- `analyze`: contexto, cronologia, controvérsias, provas e riscos;
- `petition`: checklist e minuta da peça solicitada;
- `deadlines`: marcos e pendências para conferência;
- `evidence`: mapa de provas e fontes;
- `audit`: pacote completo, quando solicitado.

## Regra de corpus

Quando o usuário enviar um processo inteiro, analise todas as páginas e imagens
do corpus, mas não carregue instruções de outras plataformas. Use o manifesto,
`relatorio_processual.md`, `pages.jsonl` e `index.jsonl` para localizar o material
em blocos. Gere ou atualize os relatórios Markdown sem apagar versões anteriores.

Não leia por padrão `adapters/` de outra plataforma, `tests/`, `build/`,
`CHANGELOG.md`, `LICENSE` ou arquivos de empacotamento. Abra-os apenas se o
usuário pedir manutenção do repositório.

## Contrato de saída

Para PDFs com imagens, a IA revisa primeiro a página inteira e depois os visuais relevantes. O núcleo usa `best_effort` por padrão; somente use `--require-semantic-vision`/`vision_policy: required` quando houver provider e `vision_review.json` real. Imagens são deduplicadas por SHA-256 e entregues em `images/index.json`, com assets de página em `assets/pages/` e ZIP portátil. Cada plataforma pode preencher o mesmo contrato sem ler os adaptadores das demais.

Execute uma tarefa por solicitação. Cite arquivo, página PDF, folha processual,
document_id e trecho quando disponíveis. Separe fato, alegação, prova, decisão e
inferência. Pergunte somente se houver ambiguidade, conflito ou dado indispensável
ausente. Nunca invente dados, calcule prazo sem premissas ou trate minuta como
protocolável.

O núcleo normativo está em `skills/legal-process-parser/SKILL.md`; este arquivo
apenas direciona a leitura para evitar desperdício de contexto.
