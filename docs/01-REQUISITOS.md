# 01 — Requisitos

## Contexto

A House Imob (unidade Caucaia) coleta dados operacionais dos corretores — quantidade de ligações, visitas, captações etc. — por meio de **vários Google Forms separados**. Hoje cada form tem um link próprio, distribuído por WhatsApp, e os corretores se perdem entre os links.

**Objetivo**: um único site (1 link) onde o corretor escolhe sua equipe e preenche todos os forms da equipe, um após o outro, sem sair da página.

## Usuários

- **Corretor**: acessa pelo **celular** (caso dominante — assume-se >90% mobile), escolhe a equipe, preenche os forms. Não faz login no site.
- **Gestor (Ryan/diretoria)**: mantém o `forms.config.json` atualizado quando um form muda ou uma equipe é criada/removida. Não é desenvolvedor — a edição precisa ser trivial.

## Requisitos funcionais

| # | Requisito | Prioridade |
|---|-----------|------------|
| RF1 | Página inicial com seleção de equipe (cards/botões grandes) | Obrigatório |
| RF2 | Ao escolher equipe, listar todos os forms daquela equipe | Obrigatório |
| RF3 | Cada form abre **embutido na própria página** (iframe), sem redirecionar | Obrigatório |
| RF4 | Navegação entre forms da equipe (abas ou lista accordion) com indicação visual de qual está aberto | Obrigatório |
| RF5 | Equipes e forms vêm 100% do `forms.config.json` — adicionar/remover form não exige mexer no HTML | Obrigatório |
| RF6 | Lembrar a equipe escolhida no dispositivo (localStorage) para visitas futuras caírem direto na equipe certa | Obrigatório |
| RF7 | Botão "trocar de equipe" sempre visível | Obrigatório |
| RF8 | Marcação manual de "já preenchi" por form no dia (checkbox local, reseta diariamente) para o corretor se orientar | Desejável |
| RF9 | Forms marcados como `comum: true` no config aparecem para todas as equipes | Desejável |
| RF10 | Link de fallback "abrir no Google Forms" em cada form (caso o iframe falhe) | Obrigatório |

## Requisitos não-funcionais

- **RNF1 — Mobile-first**: layout pensado para tela de celular; desktop é secundário.
- **RNF2 — Zero backend**: site 100% estático. As respostas continuam indo para os Google Forms/Sheets existentes — este projeto **não** toca no destino dos dados.
- **RNF3 — Zero build**: um único `index.html` com CSS e JS inline + `forms.config.json` carregado via `fetch`. Sem npm, sem framework, sem bundler.
- **RNF4 — Manutenção por não-dev**: instruções de edição dentro do próprio config.
- **RNF5 — Idioma**: toda a UI em português (pt-BR).

## Fora de escopo (não implementar)

- Autenticação/login de corretores.
- Substituir os Google Forms por formulários próprios (avaliado e rejeitado — ver `02-ARQUITETURA.md`).
- Dashboard de respostas (já existe pipeline de gamificação separado).
- Notificações/lembretes (já cobertos por workflows n8n existentes).

## Dados necessários (pendência do usuário)

Para cada equipe de Caucaia:
- Nome da equipe (ex.: "Equipe Fênix").
- Lista de forms: **título curto** + **URL do Google Form** (o link de compartilhar, formato `https://docs.google.com/forms/d/e/<ID>/viewform` ou `https://forms.gle/<código>`).

> Nota técnica: links `forms.gle` funcionam, mas o site converte para embed usando `?embedded=true`, que só funciona no formato `docs.google.com/forms/d/e/<ID>/viewform`. Links `forms.gle` redirecionam corretamente dentro do iframe na prática, mas **prefira colar o link longo** (abrir o forms.gle no navegador e copiar a URL final) para garantir.
