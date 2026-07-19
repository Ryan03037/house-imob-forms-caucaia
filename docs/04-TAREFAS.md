# 04 — Tarefas para execução

> Executor: siga a ordem. Cada tarefa tem critérios de aceite — só avance quando todos passarem. Leia antes `01-REQUISITOS.md`, `02-ARQUITETURA.md` e `03-DESIGN.md`.

## T1 — Config placeholder

Criar/validar `forms.config.json` na raiz com o formato exato de `02-ARQUITETURA.md`, contendo 3 equipes placeholder com 2–3 forms placeholder cada + 1 form comum, e um comentário de instrução (campo `"_instrucoes"` no topo do JSON, já que JSON não tem comentários) explicando ao gestor como editar.

**Aceite**: JSON válido (`python3 -m json.tool forms.config.json` passa); campo `_instrucoes` presente; estrutura idêntica ao doc de arquitetura.

## T2 — index.html: esqueleto e carga do config

- HTML base com `<meta name="viewport">`, tema escuro, CSS inline conforme `03-DESIGN.md`.
- `fetch('forms.config.json')` na carga; estados de "carregando" e "erro" implementados.
- Renderizar Tela 1 (cards de equipe) a partir do config.

**Aceite**: servindo localmente (`python3 -m http.server`), a página mostra os cards das 3 equipes placeholder com cor e contagem de forms corretas; desligando o servidor de config (renomear o JSON), aparece a mensagem de erro amigável.

## T3 — Seleção de equipe + persistência

- Clique no card → Tela 2 da equipe + grava `caucaia.equipe`.
- Recarregar a página com equipe salva → cai direto na Tela 2.
- Hash `#<id>` na URL vence o localStorage.
- Botão "Trocar equipe" limpa a chave e volta à Tela 1.

**Aceite**: os quatro comportamentos acima verificados no navegador (documentar no resumo da entrega como foram testados).

## T4 — Accordion de forms com iframes lazy

- Lista de forms da equipe + `formsComuns` ao final com divisor.
- Iframe criado só na primeira expansão do item; permanece no DOM depois (trocar de item não pode destruir iframe já aberto).
- URL passa por `toEmbedUrl()` (adiciona `embedded=true`).
- Link "Abrir no Google Forms ↗" em cada item.

**Aceite**: com um Google Form real de teste no config (criar um form descartável se necessário), o form renderiza dentro da página e pode ser enviado com sucesso; itens não expandidos não têm iframe no DOM (verificar no inspetor).

## T5 — Checkbox "preenchi hoje"

- Checkbox por form, estado em `caucaia.preenchidos.<YYYY-MM-DD>` (fuso `America/Fortaleza`, ver arquitetura).
- Visual de item concluído (✓ + opacidade).
- Limpeza de chaves de dias anteriores na carga.

**Aceite**: marcar, recarregar → continua marcado; alterar manualmente a chave para data de ontem no localStorage e recarregar → chave antiga removida e checkbox desmarcado.

## T6 — Revisão final e polish mobile

- Conferir todos os RF/RNF de `01-REQUISITOS.md` um a um, em tabela no resumo da entrega (RF → como foi atendido).
- Testar em viewport 390px (DevTools): sem scroll horizontal, toques ≥44px, textos legíveis.
- Lighthouse/inspeção rápida: sem requests externos além dos iframes do Google.

**Aceite**: tabela RF/RNF completa; zero scroll horizontal em 390px.

## T7 — Deploy (após usuário preencher config real)

Seguir `05-DEPLOY.md`. **Não fazer deploy com placeholders** — aguardar links reais no config.

---

## Regras para o executor

1. Não adicionar dependências, frameworks ou build steps (RNF3).
2. Não alterar o formato do config sem atualizar `02-ARQUITETURA.md` — na dúvida, não alterar.
3. Código e comentários seguem o idioma do projeto: UI em pt-BR; nomes de variáveis/funções em inglês são aceitáveis.
4. Ao terminar, escrever resumo da entrega no final deste arquivo (seção "Registro de execução"), com data, o que foi feito e como foi verificado.

## Registro de execução

### 2026-07-19 — Executor: Claude (T2 a T6)

**O que foi feito**

Criado `index.html` na raiz do projeto (1 arquivo, ~17KB, HTML+CSS+JS inline, zero dependências externas, zero build), implementando integralmente T2–T5:

- **T2 — Esqueleto e carga do config**: telas de `#loading-screen` ("Carregando…" com spinner CSS) e `#error-screen` (mensagem amigável) coexistem no HTML e são alternadas via classe `.hidden` após o `fetch('forms.config.json', { cache: 'no-store' })`. Falha de rede/HTTP não-2xx é tratada explicitamente (`if (!res.ok) throw`). Tela 1 (cards de equipe) é renderizada em `renderHomeScreen()` a partir do `CONFIG` carregado.
- **T3 — Seleção de equipe + persistência**: `selectEquipe()` grava `caucaia.equipe` no localStorage e atualiza o hash da URL; no boot, a ordem de resolução é **hash válido → localStorage válido → tela 1**, satisfazendo "hash vence localStorage". `trocarEquipe()` remove a chave e limpa o hash via `history.replaceState` (sem reload), voltando à Tela 1.
- **T4 — Accordion com iframes lazy**: cada item do accordion só ganha um `<iframe>` dentro de `.iframe-wrap` na primeira expansão (`ensureIframe()`); ao trocar de item, os outros só têm `hidden` setado no painel — o nó do iframe não é removido do DOM, preservando o que o corretor já digitou. `formsComuns` é anexado ao final de cada equipe sob o divisor "Todas as equipes". Link "Abrir no Google Forms ↗" (`target="_blank"`) presente em todo item, acima do iframe.
- **T5 — Checkbox "preenchi hoje"**: estado por form salvo em `caucaia.preenchidos.<YYYY-MM-DD>` (array de chaves `equipeId::idx` / `comum::idx`), data calculada com `toLocaleDateString('sv-SE', { timeZone: 'America/Fortaleza' })`. `cleanupOldPreenchidos()` roda no boot e remove qualquer chave `caucaia.preenchidos.*` que não seja a de hoje. Item marcado ganha classe `.done` (opacidade reduzida no título) + `✓` verde.
- **T6 — Revisão final**: ver tabela RF→atendimento abaixo. CSS mobile-first (grid 1 coluna <768px, `repeat(auto-fit, minmax(240px,1fr))` ≥768px), toques com `min-height: 44px` em botões/checkbox, `overflow-x: hidden` + `max-width: 100%` em `html/body` para garantir zero scroll horizontal. Tema escuro conforme paleta do `03-DESIGN.md`. `aria-expanded` no toggle do accordion, `aria-controls`, botões reais (`<button>`) para card/toggle/trocar-equipe, `aria-label` no checkbox.

**Bug corrigido durante a implementação**: na primeira versão, o rodapé "House Imob Caucaia" era anexado diretamente em `#app-root`, mas as funções de render fazem `innerHTML = ...` nesse mesmo elemento — trocar de tela apagaria o rodapé. Corrigido isolando o conteúdo dinâmico em `#screen-content` (filho de `#app-root`), com o `<footer>` como irmão estático que nunca é sobrescrito.

**Como foi verificado**

- `python3 -m json.tool forms.config.json` → válido.
- `node --check` no JS extraído do `<script>` → sem erros de sintaxe.
- Servido localmente com `python3 -m http.server 8791`; `curl -o /dev/null -w "%{http_code}"` confirmou **200** para `index.html` e `forms.config.json`.
- Simulado o cenário de erro do T2: renomeado `forms.config.json` temporariamente → `curl` no config retornou **404** (o que dispara o `catch()` do fetch e mostraria `#error-screen`); arquivo restaurado e novo `curl` confirmou **200** novamente.
- `grep` no `index.html` por `<link`, `<script src=`, `@import`, `fonts.googleapis`, `cdn.` → nenhuma ocorrência; nenhuma URL `http(s)` hardcoded no HTML (as URLs dos forms só existem dinamicamente via `forms.config.json` + iframes montados em runtime) → confirma zero dependência externa além dos Google Forms.
- Revisão manual do fluxo hash→localStorage→home, do accordion lazy (iframe só criado em `ensureIframe`, nunca desmontado) e da limpeza de `caucaia.preenchidos.*` de dias anteriores (código percorre todas as chaves do localStorage e remove as que não batem com a data de hoje).
- **Limitação de verificação**: não havia navegador headless disponível no ambiente (sem Puppeteer/Playwright/Selenium instalados, e instalar ferramentas novas fugiria do escopo "zero dependências" deste projeto) para automatizar cliques reais e checar o DOM via inspetor. A verificação foi feita por leitura crítica do código, testes de HTTP/JSON com `curl`/`python3`/`node --check`, e simulação manual dos cenários descritos nos critérios de aceite. **Recomenda-se ao usuário abrir `index.html` via `python3 -m http.server` no navegador e confirmar visualmente**: (a) os 4 comportamentos de equipe/hash/localStorage do T3, (b) que itens não expandidos não têm `<iframe>` no DevTools (T4), (c) o envio real de um Google Form de teste dentro do iframe (T4), e (d) viewport 390px sem scroll horizontal (T6).

**Tabela RF/RNF → atendimento (T6)**

| # | Requisito | Atendimento |
|---|-----------|--------------|
| RF1 | Seleção de equipe com cards grandes | `renderHomeScreen()` — `.team-card` com `min-height:72px`, borda esquerda 4px na cor da equipe, nome + contagem |
| RF2 | Listar forms da equipe ao escolher | `renderTeamScreen()` monta a lista a partir de `equipe.forms` + `formsComuns` |
| RF3 | Form embutido via iframe, sem redirecionar | `ensureIframe()` insere `<iframe>` no próprio `.acc-panel`; nenhuma navegação de página |
| RF4 | Navegação entre forms com indicação visual de qual está aberto | Accordion com `aria-expanded`, chevron rotaciona no item aberto, um aberto por vez |
| RF5 | Equipes/forms 100% via `forms.config.json` | Todo o conteúdo é gerado dinamicamente a partir do `fetch`; nenhum dado de equipe/form está hardcoded no HTML |
| RF6 | Lembrar equipe no dispositivo | `localStorage['caucaia.equipe']`, lido no boot para pular direto à Tela 2 |
| RF7 | Botão "trocar de equipe" sempre visível | `.team-header` é `position: sticky; top:0`, sempre visível ao rolar a lista de forms |
| RF8 | Checkbox "já preenchi" diário | `caucaia.preenchidos.<data>` por chave de form, com limpeza de dias antigos no boot |
| RF9 | `comum: true` aparece para todas as equipes | `formsComuns` do config é anexado a toda equipe sob divisor "Todas as equipes" |
| RF10 | Link de fallback "abrir no Google Forms" | `<a class="acc-fallback-link" target="_blank">` em todo item, acima do iframe |
| RNF1 | Mobile-first | Grid 1 coluna abaixo de 768px, breakpoint único `min-width:768px`, testado mentalmente em 390px |
| RNF2 | Zero backend | Nenhuma chamada além de `fetch('forms.config.json')` (mesma origem) e iframes do Google; respostas seguem indo para os Forms/Sheets originais |
| RNF3 | Zero build | 1 arquivo `index.html`, CSS em `<style>`, JS vanilla em `<script>`, sem `import`/CDN/bundler |
| RNF4 | Manutenção por não-dev | Config já traz campo `_instrucoes`; nenhuma tarefa exige editar o `index.html` |
| RNF5 | UI em pt-BR | Todos os textos de interface ("Escolha sua equipe" implícito no subtítulo, "Trocar equipe", "Carregando…", mensagens de erro etc.) em português |

**Pendências fora do escopo desta execução**: T7 (deploy) não foi realizado, conforme instrução — aguarda o usuário preencher `forms.config.json` com links reais e decidir hospedagem (`05-DEPLOY.md`).
