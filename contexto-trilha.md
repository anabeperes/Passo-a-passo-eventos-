# PRD — Trilha Interativa de Evento ao Vivo
### Tema: Mergulhando na IA

---

## O que é

Sistema de trilha interativa para mentorias/eventos online ao vivo. Dois arquivos HTML independentes, sem backend próprio — usam Firebase Realtime Database via REST. Identidade visual temática **oceano/mergulho**.

---

## Arquivos

### `trilha-admin.html`
Painel protegido por senha (`123`). Duas abas:

- **⚙ CONFIGURAÇÃO** — monta as etapas da trilha: tipo (parada/checkpoint), nome, texto, ordem. Botão SALVAR grava tudo no Firebase.
- **🎮 CONTROLE** — usado durante o evento ao vivo: libera etapas uma a uma (LIBERAR / VOLTAR), edita título de qualquer etapa inline (✏), edita checkpoints (nome + descrição + link).

### `trilha-participante.html`
Tela somente leitura para os participantes. Lê o Firebase a cada 3 segundos. Mostra:
- Estrada serpentina em SVG com visual oceânico (azul profundo + linha tracejada teal)
- Personagem **Leandro Sereia** pixel art que avança conforme etapas são liberadas
- Emojis temáticos clicáveis nas paradas (abre bottom sheet com texto da etapa)
- Quadrados de checkpoint clicáveis (abre bottom sheet com nome + desc + link)
- Botão fixo **🐠 ME PERDI, E AGORA?** com FAQ accordion
- Bolhas flutuantes animadas de fundo
- Barra de progresso com gradiente teal → laranja

---

## Firebase

- **Projeto:** `passo-a-passo-eventos`
- **URL:** `https://passo-a-passo-eventos-default-rtdb.firebaseio.com`
- **Path:** `/trilha.json`
- **Modo:** Teste (regras abertas)
- Acesso via REST puro (sem SDK)

### Estrutura de dados no Firebase
```json
{
  "steps": [{ "id": "s1", "type": "PDF|Aula|Doc|Site|Skill|Planilha|Prompt", "title": "Nome", "text": "Texto opcional" }],
  "step": 0,
  "cps": { "s2": { "name": "Nome CP", "desc": "Descrição", "link": "https://..." } },
  "titles": { "s1": "Título override" },
  "oferta": false
}
```

---

## Lógica principal

- `N = STEPS.length * 2 + 1` — número de pontos na trilha (dinâmico)
- `STEPS` é carregado do Firebase em todo pull (não hardcoded)
- `getTitle(id)` — retorna override de `TITLES` ou título padrão do step
- `pullState()` — lê Firebase a cada 3s, reconstrói STEPS/CPS/TITLES, chama `buildTrail()` + `updateCharacter()`
- `buildTrail()` retorna "🌊 AGUARDANDO CONFIGURAÇÃO..." se `STEPS.length === 0`
- `data.oferta = true` no Firebase ativa overlay de oferta em tela cheia

---

## Identidade Visual

### Tema
**Mergulhando na IA** — ambiente submarino profundo com bioluminescência.

### Paleta CSS

| Variável | Valor | Uso |
|---|---|---|
| `--bg` | `#020D14` | Fundo principal (oceano profundo) |
| `--bg2` | `#041824` | Fundo de cards e painéis |
| `--road` | `#052840` | Cor interna da estrada |
| `--edge` | `#083858` | Borda externa da estrada |
| `--dash` | `#00C8D4` | Linha tracejada central da estrada |
| `--bdr` | `#0A3D5C` | Bordas de elementos |
| `--text` | `#C8E8F4` | Texto principal |
| `--muted` | `#2A6080` | Texto secundário / desabilitado |
| `--acc` | `#00C8D4` | Destaque teal/ciano |
| `--glow` | `#FF7A20` | Laranja bioluminescente (hover, gradiente) |

Fonte: **Press Start 2P** (Google Fonts) — estética pixel art.

### Personagem
**Leandro Sereia** — pixel art 18×36px com:
- Cabelo cacheado escuro fluindo nas laterais
- Pele clara (`#E8B08A`)
- Expressão surpresa (sobrancelhas levantadas, boca aberta)
- Top de conchas lilás/pérola
- Barriga exposta
- Cauda azul-turquesa (`#0AB8B8`) com padrão de escamas
- Nadadeira dupla na base
- Label "LEANDRO" em teal, posicionado acima da cabeça
- Animação de bobbing (flutuar) + efeito pop ao avançar
- Glow teal animado ao redor

### Emojis por tipo de etapa
| Tipo | Emoji |
|---|---|
| PDF | 🐚 |
| Aula | 🐬 |
| Doc | 🐠 |
| Site | 🦑 |
| Skill | 🦀 |
| Planilha | 🐙 |
| Prompt | ⚓ |

---

## Interações do Participante

| Elemento | Ação | Resultado |
|---|---|---|
| Emoji de parada (liberado) | Clique | Bottom sheet com conteúdo da etapa |
| Emoji de parada (bloqueado) | Clique | "🔒 ZONA BLOQUEADA — Ainda não chegamos aqui" |
| Quadrado de checkpoint | Clique | Bottom sheet com nome + descrição + link |
| Leandro Sereia | Clique | Bottom sheet com a etapa atual |
| INÍCIO | Clique | "Já começamos, procure o Leandro Sereio pelo caminho e acompanhe a gente!" |
| CHEGADA (não concluído) | — | Opaco, ícone 🎯 |
| CHEGADA (concluído) | — | Visível, ícone 🏆, texto "MISSÃO CUMPRIDA!" |

---

## FAQ do Participante (3 itens fixos)

| Pergunta | Resposta |
|---|---|
| Me perdi, onde ele está agora? | "Clique na **SEREIA**" — palavra clicável que rola a tela até o personagem |
| Onde está o link dos materiais? | Link clicável para `vtsd.com.br/materiais-ia` |
| A trilha não atualiza, o que faço? | "Atualize a página." |

---

## Efeitos visuais

- **Bolhas animadas** — 12 bolhas `position: fixed` com tamanho/velocidade/delay aleatórios, sobem da base da tela em loop
- **Gradiente de progresso** — barra de progresso com gradiente teal → laranja
- **Glow no sync dot** — ponto verde brilhante quando conectado ao Firebase
- **Sombra no header** — `box-shadow: 0 2px 20px rgba(0,200,212,.12)`
- **Backdrop blur** no overlay do bottom sheet

---

## Deploy

- Repositório: `anabeperes/Passo-a-passo-eventos-` (GitHub)
- Deploy automático via **Netlify** (branch `main`)
- Qualquer pessoa com o link público acessa os painéis pelo navegador
- Para editar o código: colaborador no repositório GitHub ou via Claude Code

---

## Configuração rápida por evento

Para reutilizar em outro evento, alterar apenas:

1. `CFG.title` — título exibido no header (ex: `"🌊 MERGULHANDO NA IA — DIA 03"`)
2. `CFG.firebase.url` — URL do banco Firebase do evento
3. `label.textContent` — nome do apresentador no personagem (atualmente `'LEANDRO'`)
4. `FAQ_ITEMS[1].a` — link dos materiais do evento
