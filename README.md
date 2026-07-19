# Central de Formulários — House Imob Caucaia

Página única (site estático) que reúne todos os Google Forms usados pelos corretores, organizados por equipe. O corretor acessa **1 link**, escolhe sua equipe e preenche todos os formulários dali mesmo, sem caçar links no WhatsApp.

## Estado do projeto

| Fase | Status |
|------|--------|
| Planejamento / arquitetura | ✅ concluído (este conjunto de docs) |
| Coleta dos links dos forms | ⏳ **pendente — preencher `forms.config.json`** |
| Implementação do site | ⏳ aguardando execução |
| Deploy | ⏳ aguardando implementação |

## Como este projeto funciona

Este repositório usa um fluxo **orquestrador → executor**:

- O **orquestrador** (Claude Fable) produziu os documentos em `docs/` com decisões de arquitetura, requisitos e tarefas.
- Os **executores** (outros modelos/sessões) leem os docs e implementam. **Nenhum executor deve tomar decisões de arquitetura por conta própria** — se algo estiver ambíguo, está documentado como decisão em `docs/02-ARQUITETURA.md`; se não estiver, registre a dúvida e siga o padrão mais simples.

### Ordem de leitura para executores

1. `docs/01-REQUISITOS.md` — o que o site precisa fazer
2. `docs/02-ARQUITETURA.md` — como fazer (decisões fechadas, não reabrir)
3. `docs/03-DESIGN.md` — aparência e UX (mobile-first)
4. `docs/04-TAREFAS.md` — tarefas numeradas com critérios de aceite
5. `forms.config.json` — fonte única de dados (equipes + forms)

## Estrutura de arquivos (alvo final)

```
JUNTAR TODOS OS FORMS - CAUCAIA/
├── README.md              ← este arquivo
├── forms.config.json      ← equipes e links dos forms (editável sem tocar em código)
├── index.html             ← site completo (HTML + CSS + JS inline, sem build)
└── docs/
    ├── 01-REQUISITOS.md
    ├── 02-ARQUITETURA.md
    ├── 03-DESIGN.md
    ├── 04-TAREFAS.md
    └── 05-DEPLOY.md
```

## Pendências do usuário (Ryan)

1. **Preencher `forms.config.json`** com os links reais dos Google Forms de cada equipe (instruções dentro do próprio arquivo e em `docs/01-REQUISITOS.md`).
2. Confirmar nomes das equipes de Caucaia (o config traz placeholders).
3. Escolher onde hospedar (recomendação: GitHub Pages — ver `docs/05-DEPLOY.md`).
