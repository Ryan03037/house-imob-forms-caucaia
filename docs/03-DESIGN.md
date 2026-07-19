# 03 — Design / UX

> Mobile-first. Testar mentalmente tudo em viewport de 390px de largura antes de pensar em desktop.

## Identidade

- Tema escuro por padrão (corretores usam à noite para fechar o dia): fundo `#0F1115`, cards `#1A1D24`, texto `#F5F5F5`, texto secundário `#9CA3AF`.
- Cor de destaque por equipe vinda do `forms.config.json` (`cor`).
- Fonte: system stack (`-apple-system, 'Segoe UI', Roboto, sans-serif`). Sem webfonts (RNF3: zero dependência externa).
- Título do site vindo do config (`titulo`).
- Tom dos textos: direto e informal-profissional, pt-BR ("Escolha sua equipe", "Marcar como preenchido").

## Telas

### Tela 1 — Seleção de equipe

- Cabeçalho: título + subtítulo curto ("Todos os formulários da sua equipe em um só lugar").
- Cards de equipe empilhados (1 coluna no mobile, grid 2–3 colunas ≥768px), altura mínima 72px, borda esquerda de 4px na cor da equipe, nome grande, contagem de forms ("5 formulários").
- Tocar no card → Tela 2 + salva `caucaia.equipe`.
- Se já houver equipe salva, pular direto para a Tela 2 (com botão de trocar visível — RF7).

### Tela 2 — Forms da equipe

- Cabeçalho fixo (sticky): nome da equipe (na cor dela) + botão "Trocar equipe" (volta à Tela 1 e limpa a chave salva).
- **Padrão de navegação: accordion** (lista vertical de forms, um expande por vez visualmente, mas iframes já abertos permanecem montados — ver `02-ARQUITETURA.md`). Accordion escolhido em vez de abas: com 5+ forms, abas estouram a largura do celular.
- Cada item do accordion:
  - Linha do cabeçalho: título do form + checkbox "preenchi hoje" (RF8) + chevron.
  - Expandido: iframe (80vh) + link "Abrir no Google Forms ↗" logo acima do iframe.
  - Item marcado como preenchido: título com opacidade reduzida + ✓ verde.
- Forms comuns (`formsComuns`) ao final, sob divisor "Todas as equipes".
- Rodapé discreto: "House Imob Caucaia".

## Estados e detalhes

- **Carregando config**: spinner/texto simples "Carregando…".
- **Erro no fetch do config**: mensagem "Não foi possível carregar os formulários. Abra pelo link oficial ou avise o gestor." — nada de tela branca.
- **Equipe do hash**: se a URL vier com `#<id-da-equipe>` válido, ela vence o localStorage (permite mandar link direto da equipe no grupo de WhatsApp de cada equipe).
- Toque: áreas clicáveis ≥44px de altura.
- Sem animações complexas; transição de accordion simples (max-height ou grid-rows) é suficiente.
- Acessibilidade mínima: botões reais (`<button>`), `aria-expanded` no accordion, contraste AA no tema escuro.
