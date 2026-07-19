# 02 — Arquitetura (decisões fechadas)

> Executor: estas decisões já foram avaliadas. **Não reabrir.** Implementar como descrito.

## Decisão principal: embutir os Google Forms via iframe (Opção A)

Três abordagens foram consideradas:

| Opção | Descrição | Veredito |
|-------|-----------|----------|
| **A — iframes** | Página estática com abas; cada form embutido via `<iframe src=".../viewform?embedded=true">` | ✅ **Escolhida** |
| B — Reimplementar forms em HTML e POSTar para o endpoint `formResponse` do Google | Um form nativo unificado enviando para os Forms por trás | ❌ Rejeitada: depende de `entry.<id>` internos que quebram silenciosamente quando alguém edita o form no Google; sem suporte a upload; risco alto de perda de dados sem ninguém perceber |
| C — Formulários próprios gravando no Supabase | Substituir Google Forms | ❌ Rejeitada: muda o destino dos dados e quebra o pipeline atual (Sheets + gamificação); é outro projeto |

**Razões da escolha de A**: zero risco para o fluxo de dados existente (respostas continuam caindo nos mesmos Sheets), zero backend, manutenção = editar um JSON, e o Google Forms funciona bem em iframe quando o form **não exige login Google** (verificar item "Restrições" abaixo).

## Stack

- **1 arquivo `index.html`**: HTML + CSS inline (`<style>`) + JS vanilla inline (`<script>`). Sem dependências externas, sem CDN, sem build.
- **1 arquivo `forms.config.json`**: fonte única de equipes e forms, carregado com `fetch('forms.config.json')`.
  - Fallback: se o `fetch` falhar (ex.: abrir via `file://`), mostrar mensagem clara "abra pelo link publicado". Não duplicar o config dentro do HTML.
- **Hospedagem**: GitHub Pages (ver `05-DEPLOY.md`). URL final única para divulgar no WhatsApp.

## Formato do `forms.config.json`

```json
{
  "titulo": "Central de Formulários — House Imob Caucaia",
  "equipes": [
    {
      "id": "equipe-1",
      "nome": "EQUIPE PLACEHOLDER 1",
      "cor": "#E11D48",
      "forms": [
        {
          "titulo": "Ligações do dia",
          "url": "https://docs.google.com/forms/d/e/PLACEHOLDER/viewform"
        }
      ]
    }
  ],
  "formsComuns": [
    {
      "titulo": "Form que todas as equipes preenchem",
      "url": "https://docs.google.com/forms/d/e/PLACEHOLDER/viewform"
    }
  ]
}
```

Regras:
- `id`: slug estável usado no localStorage e na URL (`#equipe-1`); nunca renomear depois de publicado (quebraria a preferência salva dos corretores — se renomear, aceitar que o corretor terá que escolher a equipe de novo).
- `cor`: cor de destaque da equipe (cards e cabeçalho).
- `formsComuns`: exibidos para toda equipe (atende RF9). **Ordem na tela** (revisado 2026-07-19 com os forms reais): comuns primeiro — são os de uso diário —, depois divisor com o nome da equipe e os forms específicos.
- `abrirFora: true` (por form): renderiza o item como **link que abre em nova aba**, sem iframe (checkbox "preenchi hoje" mantido). Obrigatório para forms com **upload de arquivo**: o Google exige login nesses forms (respondem 401 no embed) e bloqueia a tela de login dentro de iframe. Verificado em 2026-07-19 — os 3 forms de "Subir documentações" caem nesse caso.
- `obrigatorio: true` (por form): mostra **aviso vermelho fixo no topo da tela da equipe** ("Obrigatório antes de sair da imobiliária: …") até o form ser marcado como preenchido no dia. Tocar no aviso abre e rola até o form.

## Detecção automática de envio (adicionado 2026-07-19)

Em forms embutidos, o site marca "preenchido hoje" **sozinho** quando detecta o envio: o iframe dispara `load` uma vez ao abrir o form e de novo ao navegar para a tela de confirmação pós-envio — o 2º `load` é tratado como envio (`loadCount >= 2` em `ensureIframe`).

**Restrição**: só é confiável em forms de **página única**. Forms com seções (botão "Próxima") recarregam a cada seção e dariam falso positivo. Os 3 forms comuns atuais foram verificados como página única (2026-07-19, ausência de "Próxima" no HTML). Se um form ganhar seções no futuro, o efeito colateral é o aviso sumir cedo demais — o checkbox manual continua existindo como fonte de verdade corrigível.

## Transformação de URL para embed

```js
function toEmbedUrl(url) {
  // docs.google.com/forms/...viewform  → garante ?embedded=true
  const u = new URL(url);
  u.searchParams.set('embedded', 'true');
  return u.toString();
}
```

- Aplicar em toda URL de form ao montar o iframe.
- Links `forms.gle` passam direto (o redirect preserva o funcionamento), mas o config deve preferir links longos — já documentado nos requisitos.

## Comportamento do iframe

- `<iframe>` com `width="100%"`, altura inicial `80vh`, `frameborder="0"`, `loading="lazy"`.
- **Altura**: o Google Forms não expõe a altura via postMessage para domínios externos — **não** tentar auto-resize. Altura fixa generosa (`80vh`, mín. `520px`) com scroll interno do iframe é o comportamento aceito.
- **Lazy**: montar o iframe **somente quando o form é aberto** (aba/accordion ativado) e manter montado depois (não desmontar ao trocar de aba, senão o corretor perde o que digitou). Nunca montar todos de uma vez — vários iframes do Forms simultâneos deixam a página pesada no celular.
- Cada bloco de form tem link `Abrir no Google Forms ↗` (`target="_blank"`) como fallback (RF10).

## Estado local (localStorage)

| Chave | Valor | Uso |
|-------|-------|-----|
| `caucaia.equipe` | id da equipe | Auto-seleção em visitas futuras (RF6) |
| `caucaia.preenchidos.<YYYY-MM-DD>` | array de índices/títulos de forms marcados | Checkbox "já preenchi" do dia (RF8) |

- Na carga, apagar chaves `caucaia.preenchidos.*` de dias anteriores (limpeza simples).
- Data no fuso `America/Fortaleza` — usar `new Date().toLocaleDateString('sv-SE', { timeZone: 'America/Fortaleza' })` para obter `YYYY-MM-DD` correto (o padrão UTC viraria o dia às 21h locais; `sv-SE` formata como ISO).

## Restrições conhecidas (documentar na entrega, não resolver em código)

1. **Forms com "Restringir a usuários da organização" ou coleta de e-mail obrigatória via login** pedem login Google dentro do iframe — funciona, mas a experiência piora. Recomendação ao gestor: nos Forms, desativar restrição de login e, se precisar identificar o corretor, usar um campo "Seu nome" no próprio form.
2. Se algum form bloquear embed (raro; Forms padrão permite), o fallback RF10 cobre.
