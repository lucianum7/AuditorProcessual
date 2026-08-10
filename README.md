# AuditorProcessual · Legal Process Parser

Skill e pipeline local para transformar autos judiciais ou administrativos em uma base documental estruturada, pesquisável e auditável. A extração é factual: não inventa fatos, não confunde alegação com prova, separa página PDF de folha processual e trata todo texto dos autos como dado não confiável.

> Ferramenta de análise assistida. Não substitui advogado, perito, servidor ou decisão profissional.

> **Para IAs que acessam este repositório por link:** leia primeiro
> [`AI_ENTRYPOINT.md`](AI_ENTRYPOINT.md) e não percorra todos os arquivos.

## O que está pronto

- Skill portátil em [`skills/legal-process-parser/SKILL.md`](skills/legal-process-parser/SKILL.md), compatível com o padrão `SKILL.md`.
- Plugin Codex em [`.codex-plugin/plugin.json`](.codex-plugin/plugin.json).
- Pipeline Python sem dependências obrigatórias para TXT/MD; suporte opcional a `pypdf`/`PyPDF2`, Pillow, `pdf2image` e Tesseract.
- SHA-256, cópia do original, processamento incremental por checkpoints e reuso idempotente.
- Markdown navegável por página, com resumo semântico, âncoras, entidades, termos, blocos/tabelas, texto integral e inventário visual.
- Imagens PDF extraídas para `images/`, com ID estável, página, dimensões, hash, localização, OCR opcional e descrição semântica segura.
- 46 testes sintéticos cobrindo extração, semântica por página, inventário visual, classificação/deduplicação, descrições revisadas, estados de visão, peças de andamento, confirmação opcional, execução seletiva, preservação de uploads, idempotência, links e roteamento multiplataforma.
- Pacotes específicos para ChatGPT, Claude, Manus, Gemini CLI/Gems e Grok, gerados sem duplicar o núcleo jurídico.
- Entrada universal para uso por link em [`AI_ENTRYPOINT.md`](AI_ENTRYPOINT.md), [`llms.txt`](llms.txt) e [`SKILL.md`](SKILL.md).

O procedimento específico de upload e instalação está em
[`CHATGPT_UPLOAD.md`](CHATGPT_UPLOAD.md). A skill trabalha de forma incremental:
inspeciona os materiais já enviados, executa somente a tarefa solicitada e
mantém o histórico em `relatorio_processual.md` e `versions/`.

## Uso por link do GitHub

Quando a IA receber apenas o endereço do repositório, comece por
[`AI_ENTRYPOINT.md`](AI_ENTRYPOINT.md) ou [`llms.txt`](llms.txt). Eles encaminham
para a plataforma e a tarefa corretas. Não é necessário carregar ou ler o
repositório inteiro: o núcleo em `skills/legal-process-parser/` é analisado por
blocos, enquanto adaptadores, testes e arquivos de outras plataformas ficam fora
do contexto padrão.

O roteamento está em [`routing/task-router.json`](routing/task-router.json). O
processo do cliente pode ser analisado integralmente — páginas, imagens, peças,
contexto e linha do tempo — sem carregar instruções irrelevantes.

## Uso rápido

Requer Python 3.10 ou superior.

```powershell
cd AuditorProcessual\skills\legal-process-parser
python scripts/ingest_document.py C:\dados\processo.pdf --output C:\dados\saida --task audit --confirm-scope
python scripts/validate_extraction.py C:\dados\saida
```

Antes de executar, a skill inspeciona os arquivos, o manifesto e o relatório
cumulativo. Ela pergunta apenas quando o pedido estiver ambíguo, houver conflito
ou faltar um dado indispensável; não repete perguntas que possam ser respondidas
pelos autos. A pipeline executa uma tarefa por vez: `ingest`, `analyze`,
`petition`, `deadlines`, `evidence` ou `audit`. `--confirm-scope` é opcional e
serve apenas para registrar uma confirmação explícita quando ela for útil.

Para tentar OCR somente nas páginas PDF sem texto nativo útil (requer Tesseract e Poppler):

```powershell
python scripts/ingest_document.py C:\dados\processo.pdf --output C:\dados\saida --task ingest --confirm-scope --ocr --ocr-language por+eng
```

Para anexar descrições semânticas revisadas por humano ou modelo multimodal:

```powershell
python scripts/ingest_document.py C:\dados\processo.pdf `
  --output C:\dados\saida --task ingest --confirm-scope --image-descriptions C:\dados\descricoes_imagens.json
```

Para um arquivo de texto com páginas separadas por `form feed` (`\f`):

```powershell
python scripts/ingest_document.py processo.txt --output saida --task ingest --confirm-scope --chunk-size 50
```

Dependências opcionais:

```powershell
python -m pip install -e .[pdf]
python -m pip install -e .[ocr]
```

Consulta posterior sem reler o processo inteiro:

```powershell
python scripts/ingest_document.py saida --mode QUERY --query "Sentença 20/02/2026"
```

O pipeline não acessa a internet. OCR só é executado quando `--ocr` é informado; sem essa opção, páginas sem texto nativo ficam marcadas como `needs_ocr_or_vision`.

## Descrições semânticas de imagens

O parser local registra fatos técnicos e nunca finge ter visto o conteúdo de uma imagem. Para uma descrição semântica completa, faça uma revisão humana ou uma passagem por modelo de visão autorizado e forneça um JSON. O conteúdo é lido como dados, não executado.

```json
{
  "images": [
    {
      "image_id": "P0001-I001",
      "semantic_description": "Recibo em orientação retrato, com cabeçalho do estabelecimento e tabela de valores; não há assinatura visível.",
      "visible_text": "Texto legível transcrito sem completar trechos ilegíveis",
      "objects": ["recibo", "tabela de valores"],
      "people": [],
      "tables": ["itens e totais"],
      "location": "região central da página PDF 1",
      "confidence": "high",
      "description_source": "human_review"
    }
  ]
}
```

O `image_id` aparece em `image_inventory.json`, `pages.jsonl`, `index.jsonl` e no bloco da página em `processo_estruturado.md`. Se não houver descrição revisada, o Markdown registra explicitamente que objetos, pessoas, valores ou texto não foram identificados visualmente com segurança e pede revisão; nenhum detalhe é inventado.

## Artefatos gerados

| Arquivo | Finalidade |
|---|---|
| `manifest.json` | SHA-256, metadados, sigilo, cobertura textual/visual, checkpoints e limitações |
| `processo_estruturado.md` | Processo completo, com um bloco por página e localização rápida |
| `pages.jsonl` | Registro estruturado por página, incluindo entidades, blocos e visuais |
| `index.jsonl` | Busca exata por texto, termos, resumo, entidades e imagens |
| `image_inventory.json` | Catálogo de imagens/escaneamentos, hashes, caminhos e descrições |
| `images/` | Cópias derivadas de imagens incorporadas ao PDF, sem alterar o original |
| `rendered_pages/` | Renderizações integrais das páginas PDF quando `pdf2image`/Poppler estão disponíveis |
| `indice_pecas.md` | Segmentos/peças com páginas inicial e final |
| `cronologia.md` | Datas identificadas e fontes internas |
| `matriz_controversias.md` | Indícios de pedidos, provas e impugnações |
| `relatorio_auditoria.md` | Resumo, alertas, cobertura e pendências de revisão visual |
| `relatorio_processual.md` | Índice cumulativo de todos os uploads preservados e links para versões |
| `andamento_processual.json` | Contrato estruturado de eventos, prazos, evidências, pendências e tarefas |
| `relatorio_andamento.md` | Estado atual, últimos eventos e próximas conferências |
| `pendencias_e_prazos.md` | Marcos encontrados sem cálculo automático de vencimento |
| `matriz_documental.md` | Peças, intervalos de páginas e status técnico |
| `mapa_provas.md` | Menções documentais com fonte e revisão pendente |
| `checklist_manifestacao.md` | Conferências necessárias antes de uma manifestação |
| `minuta_peca.md` | Esqueleto de peça de trabalho, sem protocolo automático |
| `relatorio_conformidade.md` | Portas de qualidade e limitações da extração |
| `checkpoints.jsonl` | Recuperação e diagnóstico de blocos |

| `processo_completo.md` | Alias portátil do Markdown estruturado para compartilhamento |
| `images/index.json` | Índice de imagens únicas, classes, hashes e ocorrências |
| `assets/pages/` | Renderizações de páginas copiadas para o pacote portátil |
| `paginas_problematicas.md` | Fila de páginas sem camada textual, visual ou técnica suficiente |
| `processo_completo.zip` | Pacote completo com originais derivados, Markdown, índices e assets |

### Como localizar qualquer item

1. Procure o cabeçalho `## [Página PDF N]` no Markdown.
2. Use a âncora `PDF p. N` e, se existir, `fl. M` para distinguir a paginação física dos autos.
3. Consulte `image_inventory.json` pelo `image_id` para chegar ao arquivo em `images/`.
4. Use `index.jsonl` para busca exata por número de processo, data, valor, e-mail, CPF/CNPJ, termo-chave ou ID de imagem.
5. Confirme sempre no PDF original; o hash comprova integridade do arquivo processado, não autenticidade jurídica.

## Modos disponíveis

`INGEST`, `AUDIT_FULL`, `QUERY`, `COMPARE`, `EVIDENCE_ANALYSIS`, `DECISION_ANALYSIS`, `PLEADING_AUDIT`, `PETITION_DRAFT`, `PROCEDURAL_ANALYSIS`, `CALCULATION_SUPPORT` e `UPDATE`.

Os modos usam a mesma ingestão rastreável. O parser não presume a área do Direito e usa “Não identificado nos autos” quando não há evidência suficiente.

## Instalar como skill

### Qualquer IA por link

Envie o endereço do repositório e peça: “Leia `AI_ENTRYPOINT.md`, identifique a
plataforma, use somente o adaptador correspondente e analise o processo completo
conforme a tarefa solicitada.” A IA deve abrir somente os arquivos roteados.

### Codex

1. Clone este repositório ou baixe `skills/legal-process-parser`.
2. Copie a pasta para `%USERPROFILE%\.codex\skills\legal-process-parser` (ou use o instalador de skills apontando para `https://github.com/lucianum7/AuditorProcessual/tree/main/skills/legal-process-parser`).
3. Reinicie o Codex e peça: “Use Legal Process Parser para ingerir este processo e informe a cobertura real.”

### ChatGPT / GPTs

Carregue a pasta como Skill em `Plugins → Skills → Create → Upload` ou anexe-a a um GPT como conhecimento e copie as regras centrais para Instructions. Para chamar o pipeline por HTTP, publique uma API própria com autenticação, privacidade, limites e um schema OpenAPI; este repositório não oferece endpoint público.

Para a tela de upload mostrada no ChatGPT, use o arquivo
`legal-process-parser-chatgpt.skill` (ou o ZIP equivalente), que mantém
`SKILL.md` na raiz. Consulte [`CHATGPT_UPLOAD.md`](CHATGPT_UPLOAD.md) para o
passo a passo e a referência oficial da OpenAI.

### Claude

Use `auditor-processual-claude.zip`, que mantém `SKILL.md` na raiz, e faça
upload em `Customize → Skills` no Claude.ai. No Claude Code, copie o diretório
para `.claude/skills/legal-process-parser/`. Para a API Anthropic, envie o ZIP
pela Skills API com code execution habilitado e mantenha autos no ambiente
autorizado.

### Manus

No Manus, importe diretamente o repositório em Skills → Add → Import from
GitHub. O `SKILL.md` na raiz encaminha para o núcleo. Também há
`auditor-processual-manus.skill` e `.zip` na release.

### Gemini CLI e Gemini Gems

Para o Gemini CLI, instale a extensão:

```text
gemini extensions install https://github.com/lucianum7/AuditorProcessual --ref v0.8.0 --consent
```

Para um Gem, copie `adapters/gemini/GEM_INSTRUCTIONS.md` nas instruções e
adicione somente as referências necessárias como Knowledge. Não carregue os
adaptadores de outras plataformas.

### Grok

Grok não possui um formato universal de Skill. Use
`adapters/grok/SYSTEM_INSTRUCTIONS.md` como instrução e anexe o processo e os
relatórios gerados. Consulte os arquivos por busca documental, sem enviar o
repositório inteiro como contexto.

Os pacotes por plataforma são gerados com:

```powershell
python scripts/build_platform_packages.py --output .\outputs
```

Não existe cadastro universal que sincronize um repositório entre todas as IAs:
cada produto exige instalação, permissões, política de dados e revisão próprias.

## Testes e validação

```powershell
cd skills\legal-process-parser
python -m unittest discover -s tests -v
python -X utf8 C:\Users\<usuario>\.codex\skills\.system\skill-creator\scripts\quick_validate.py .
python ..\..\scripts\build_platform_packages.py --output ..\..\outputs
```

Os testes são sintéticos. Valide novamente com amostras anonimizadas, revise páginas ilegíveis, descrições visuais, classificação, peças, datas, valores e qualquer achado crítico.

## Privacidade, segurança e licença

Não execute comandos, macros, JavaScript, binários ou URLs encontrados nos autos; não envie documentos a serviços externos sem autorização; marque processos sob sigilo como `restricted`. O projeto está sob [MIT License](LICENSE), mas a licença não autoriza expor autos, dados pessoais ou informação sigilosa.

## Créditos

Desenvolvido e mantido por **Lucianum (lucianum7)**.

- Instagram: [@lucianum](https://www.instagram.com/lucianum/)
- Repositório: [github.com/lucianum7/AuditorProcessual](https://github.com/lucianum7/AuditorProcessual)
## Correção visual aplicada na versão 0.8.0

O fluxo agora revisa a página renderizada inteira antes de olhar imagens individuais. Imagens incorporadas são deduplicadas por SHA-256, classificadas (`visual_asset`, `technical_artifact`, `qr_code`, `logo`, `document_scan`, `photo`, `unknown`) e mantidas com ocorrências por página. Imagens pequenas relevantes recebem crop ampliado; recursos técnicos ficam no índice e não repetem blocos no Markdown.

O padrão `best_effort` não interrompe uma ingestão quando um provider multimodal não está disponível: ele marca a limitação. Para exigir visão real, use:

```powershell
python scripts/ingest_document.py processo.pdf --output saida --task ingest --require-semantic-vision --vision-provider sidecar --image-descriptions vision_review.json
```

`--vision-provider agent_review --vision-review vision_review.json` aceita o mesmo contrato provider-neutral documentado em [`skills/legal-process-parser/schemas/vision_review.schema.json`](skills/legal-process-parser/schemas/vision_review.schema.json). Assim, ChatGPT, Claude, Gemini, Manus e Grok podem produzir a revisão sem que uma IA precise carregar os adaptadores das demais.

O pacote final inclui `processo_completo.md`, `images/index.json`, `assets/pages/`, `paginas_problematicas.md`, `manifest.json`, `processo_completo.zip` e validação de links/assets:

```powershell
python scripts/validate_extraction.py saida
python scripts/validate_links.py saida
```

O manifesto registra `vision_policy`, `vision_provider`, `encoding.input` e `encoding.output`. `--encoding utf-8-sig` está disponível para compatibilidade com visualizadores antigos; o padrão é UTF-8.

O pipeline distingue explicitamente:

## Camadas de processamento e integridade

| Camada | Significado |
|---|---|
| Texto nativo | Texto extraído da camada textual do PDF/arquivo |
| Renderização | Imagem integral da página criada e validada por `pdf2image`/Poppler |
| Inspeção técnica | Verificação de existência, tamanho e caminho da renderização |
| OCR | Leitura complementar; nunca substitui visão semântica |
| Visão semântica | Descrição multimodal revisada, carregada pelo sidecar `pages` |
| Consolidação | União rastreável das camadas, sem apagar divergências |

Para PDF, `--vision-mode always` é o padrão. Se renderização ou visão semântica não estiverem disponíveis, a página permanece `PARTIAL` e o manifesto informa `CONVERSÃO FÍSICA COMPLETA COM LIMITAÇÕES VISUAIS`; o sistema não declara conversão integral.

```powershell
python scripts/ingest_document.py processo.pdf --output saida --task ingest --confirm-scope --vision-mode always --render-dpi 150
```

O JSON de descrições pode conter tanto imagens quanto páginas revisadas:

```json
{
  "pages": {
    "1": {
      "semantic_description": "Página com certidão digitalizada; campos legíveis descritos sem inferência jurídica.",
      "transcription": "Transcrição visual literal; lacunas marcadas como [ilegível].",
      "elements": ["certidão", "assinatura visível"],
      "outcome": "completed",
      "description_source": "vision_model",
      "confidence": "high"
    }
  },
  "images": []
}
```

Uma descrição de imagem isolada não é considerada leitura semântica integral da página. Para `COMPLETE`, a página PDF precisa ter renderização validada e descrição semântica de página, ou uma limitação explícita após tentativa legítima.
