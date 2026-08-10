# Legal Process Parser — instruções para Claude

Leia `skills/legal-process-parser/SKILL.md` como núcleo e
`routing/task-router.json` como roteador. Para processos completos, examine
todo o corpus enviado — texto, imagens, peças, contexto, datas e linha do tempo
— em blocos e com citações por página.

Execute somente uma tarefa por solicitação. Antes de perguntar, procure os dados
no processo, no manifesto e em `relatorio_processual.md`. Pergunte apenas em
caso de ambiguidade, conflito ou lacuna indispensável. Preserve uploads e
versões, não invente conteúdo e marque toda minuta como **RASCUNHO — NÃO
PROTOCOLAR**.
