# 05 — Deploy

## Recomendação: GitHub Pages

Gratuito, URL estável, publica direto do repositório — combina com o fluxo já usado nos outros projetos da House Imob.

### Passos

1. Criar repositório no GitHub (ex.: `house-imob-forms-caucaia`), privado não funciona com Pages no plano free — usar **público** (o site não contém segredo algum: só títulos e links de forms que já são públicos).
2. Commitar `index.html` + `forms.config.json` na branch `main` (os `docs/` podem ir junto, não atrapalham).
3. Settings → Pages → Source: `Deploy from a branch`, branch `main`, pasta `/ (root)`.
4. URL final: `https://<usuario>.github.io/house-imob-forms-caucaia/`.
5. Testar a URL no celular antes de divulgar.

### Atualizar forms depois

Editar `forms.config.json` direto na interface web do GitHub (botão de lápis) → commit → Pages republica em ~1 min. Não precisa mexer no `index.html`.

## Alternativas (se Pages não servir)

- **Netlify Drop** (arrastar a pasta em app.netlify.com/drop): mais rápido para testar, mas atualizar exige re-arrastar — pior para manutenção contínua.
- **Vercel**: equivalente ao Pages; só vale se já houver conta.

## Divulgação

- Mandar o link único nos grupos de WhatsApp.
- Opcional por equipe: link com hash direto (`.../#equipe-1`) no grupo daquela equipe — abre já na equipe certa (ver `03-DESIGN.md`, "Equipe do hash").
- Sugestão: fixar a mensagem com o link no grupo.
