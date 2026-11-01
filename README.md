# VINI MASTER AI — Alpha 0.2

**Seu escritório isométrico de agentes virtuais na Cloudflare.** A interface foi desenvolvida para funcionar no navegador do celular ou computador. Não exige que seu computador pessoal fique ligado quando estiver publicada.

## O que já está implementado

- Painel visual, responsivo, com **seis estações isométricas** clicáveis, indicadores reais e linha do tempo.
- Chat com supervisor usando **Cloudflare Workers AI (GLM-4.7-Flash)** ou **NVIDIA NIM (Nemotron 3 Ultra)**.
- Seleção automática: tenta Cloudflare primeiro e recorre à NVIDIA quando disponível; também é possível escolher somente um provedor.
- **Missões duráveis via Cloudflare Workflows**: um supervisor planeja, cinco especialistas fazem análises e o supervisor consolida o relatório. A página pode ser fechada durante a execução.
- **Memória D1**: grava todas as mensagens enviadas e respostas recebidas, indexa o texto com SQLite FTS5, oferece busca e guarda memórias fixadas. Ao escrever "lembre que ...", o conteúdo também é fixado.
- Histórico de missões, resultados por agente e atividades; exportação e exclusão de conversas.
- Chave privada obrigatória para a API, armazenada apenas na sessão atual do navegador. Nenhum token é publicado no repositório.
- Limite inicial: **3 novas missões por dia**, para reduzir risco de consumir a cota gratuita inadvertidamente.

**Importante:** na Alpha 0.2 os agentes geram **análises, relatórios, ideias e código textual**; eles **não** editam arquivos externos, não executam comandos arbitrários, não dão merge e não publicam sites. A visualização reflete execuções registradas no D1, não animações falsas de trabalho. A memória preserva os registros, mas a IA recupera apenas o histórico recente, memórias fixadas e trechos antigos pertinentes; nenhuma IA consegue considerar todas as mensagens simultaneamente.

## O que falta para ficar online

Este projeto contém o **código-fonte completo, preparado para publicação via GitHub integrado à Cloudflare**. **Não foi publicado na conta Cloudflare**; é necessário conectar a conta; o banco D1 será criado automaticamente no primeiro deploy. Leia `DEPLOY-CLOUDFLARE.md` para a implantação sem usar computador local.

## Organização

- `public/index.html`, `styles.css`, `app.js`, `office.svg` — interface responsiva.
- `src/index.js` — Worker e proteções HTTP.
- `src/api.js` — APIs do painel, conversa, memória e missões.
- `src/ai.js` — conexão com modelos NVIDIA e Cloudflare.
- `src/workflow.js` — tarefas duráveis dos seis agentes.
- `schema.sql` — tabelas D1 e índice pesquisável FTS5.
- `src/bootstrap.js` — cria tabelas D1 na primeira requisição autenticada, sem executar SQL no painel.
- `wrangler.jsonc` — bindings Cloudflare.
- `tests` — testes de modelos, autenticação e banco SQLite.

## Segurança e privacidade

- Crie a variável secreta `MASTER_TOKEN` (mínimo 24 caracteres; recomendado 40+ aleatórios). **Não** coloque a chave em `wrangler.jsonc` ou no GitHub.
- O segredo opcional `NVIDIA_API_KEY` deve ser configurado pela Cloudflare. A integração com NVIDIA só funciona depois disso.
- Conversas e memórias são privadas **somente se a chave for mantida secreta**. Para uso em equipe, adicionar Cloudflare Access e contas separadas antes de produção.
- Evite salvar senhas ou dados especialmente sensíveis nas conversas. Quando NVIDIA for utilizada, o contexto relevante será enviado também ao provedor NVIDIA para gerar a resposta.
- O histórico cresce no D1 conforme o uso; limites e retenção do plano Cloudflare continuam valendo. Não se trata de armazenamento absolutamente ilimitado.

## Testes executados nesta entrega

- Testes Node.js: `node --test tests/ai.test.mjs`.
- Testes SQL com SQLite: `python -m unittest discover -s tests -p 'test_*.py' -v`.
- Verificação sintática de JS com `node --check`.

Ainda é necessário um **teste de implantação real na Cloudflare**, incluindo Workers AI, Workflows e o banco D1 remoto, antes de declarar a aplicação pronta para produção.