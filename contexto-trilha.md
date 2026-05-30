# PRD — Trilha Interativa de Evento ao Vivo
**Produto:** Sistema de trilha para eventos e mentorias online  
**Tema atual:** 🌊 Mergulhando na IA  
**Repositório:** `anabeperes/Passo-a-passo-eventos-` (GitHub)  
**Deploy:** Netlify (branch `main`, automático)

---

## 1. Visão Geral

Sistema composto por dois arquivos HTML independentes (sem framework, sem build, sem backend próprio) que se comunicam via Firebase Realtime Database em modo REST puro. O apresentador controla o evento em tempo real pelo painel admin; os participantes acompanham a trilha que atualiza automaticamente a cada 3 segundos.

**Premissa:** qualquer pessoa com o link acessa pelo navegador, sem instalar nada. Para editar o código, basta ser colaborador no repositório.

---

## 2. Arquivos do Sistema

### 2.1 `trilha-admin.html` — Painel do Apresentador

**Acesso:** protegido por senha (`123`)

**Aba ⚙ CONFIGURAÇÃO** (pré-evento):
- Lista de etapas com campos: número de ordem, tipo (dropdown), nome, texto descritivo
- Botões de reordenação (↑ ↓) e exclusão (✕) por linha
- Botão `+ ADICIONAR ETAPA` no final da lista
- Contador de etapas no topo
- Barra fixa no rodapé com botão **SALVAR →** — ao salvar, reseta `step: 0` e publica tudo no Firebase
- Proteção contra saída acidental (`beforeunload`) se houver alterações não salvas

**Aba 🎮 CONTROLE** (durante o evento ao vivo):
- **Contador grande:** `X / total` etapas liberadas
- **Barra de progresso** com percentual
- **Preview:** etapa atual (✅ ATUAL) e próxima etapa (⏭ PRÓXIMA) com badge de tipo colorido
- **Botões de controle:** `◀ VOLTAR` e `LIBERAR ▶` — cada clique faz PUT no Firebase e atualiza a tela dos participantes em até 3s
- **Seção 🎁 OFERTA ESPECIAL:** botões para liberar/fechar overlay de oferta em tela cheia para os participantes
- **Lista de etapas:** todas as etapas com status (liberada = verde / bloqueada = cinza), badge de tipo, ícone 📋 se tiver checkpoint preenchido, ícone 🔒 se bloqueada
- **Edição de título inline:** ao clicar no ✏ de qualquer etapa, transforma o nome em campo de input inline; Enter ou ✓ para salvar e sincronizar
- **Edição de checkpoint:** ao clicar em uma etapa da lista, carrega campos de edição: nome do checkpoint, passo a passo (textarea), link opcional. Botão **SALVAR CHECKPOINT** faz PUT imediato no Firebase

**Indicador de sync:** badge AO VIVO / OFFLINE no topo — verde quando a última operação com Firebase teve sucesso

---

### 2.2 `trilha-participante.html` — Tela do Participante

Somente leitura. Atualiza automaticamente via polling a cada 3 segundos.

**Estrutura visual (de cima para baixo):**

1. **Header fixo (sticky)**
   - Título do evento (ex: `🌊 MERGULHANDO NA IA — DIA 03`)
   - Ponto de sincronização (verde pulsante = conectado, cinza = offline)
   - Barra de progresso com gradiente teal → laranja
   - Texto `X / Y etapas liberadas`
   - Saudação personalizada (ex: `🌊 OI, ANA!`) após preenchimento do nome

2. **Trilha SVG serpentina**
   - Caminho em forma de S com curvas suaves (viewBox `0 0 360 1300`)
   - Três layers de path sobrepostos: borda externa (56px), corpo da estrada (50px), linha tracejada central (2px, dashed)
   - Marcadores posicionados dinamicamente via `getPointAtLength()`
   - Posições: INÍCIO → parada₁ → checkpoint₁ → parada₂ → checkpoint₂ → ... → CHEGADA
   - Total de pontos: `N = STEPS.length × 2 + 1`

3. **Personagem Leandro Sereia**
   - Pixel art 18×36px, renderizado em 45×90px
   - Label "LEANDRO" em teal acima da cabeça (não cobre o rosto)
   - Animação de bobbing (flutuar suavemente) em loop
   - Efeito pop (escala + salto) ao avançar de posição
   - Glow teal animado ao redor
   - Clicável: abre bottom sheet com a etapa atual
   - Avança de posição conforme `unlocked` aumenta

4. **Bolhas decorativas**
   - 12 bolhas com `position: fixed`, tamanho entre 4–16px
   - Animação de subida com escala e fade, duração e delay aleatórios
   - Contribuem para a atmosfera subaquática sem interferir na interação

**Elementos interativos:**

| Elemento | Estado | Ação | Resultado |
|---|---|---|---|
| Emoji de parada | Liberado | Clique | Bottom sheet com tipo + título + texto da etapa |
| Emoji de parada | Bloqueado | Clique | "🔒 ZONA BLOQUEADA — Ainda não chegamos aqui." |
| Quadrado de checkpoint | Com conteúdo e liberado | Clique | Bottom sheet com nome + passo a passo + link |
| Quadrado de checkpoint | Bloqueado | — | Invisível (opacity 0.15, sem pointer-events) |
| Leandro Sereia | — | Clique | Bottom sheet com etapa atual |
| INÍCIO | — | Clique | "Já começamos, procure o Leandro Sereio pelo caminho e acompanhe a gente!" |
| CHEGADA | Não concluído | — | Opaco (30%), ícone 🎯, texto "CHEGADA" |
| CHEGADA | Concluído | — | Visível, ícone 🏆, texto "MISSÃO CUMPRIDA!" |
| Oferta ativada | — | Automático | Overlay de oferta em tela cheia |

**Bottom sheet:**
- Desliza da base da tela, ocupa até 72vh com scroll interno
- Fundo com `backdrop-filter: blur(2px)`
- Borda superior em teal
- Exibe: label do conteúdo + descrição + botão de link (quando houver)
- Fecha ao clicar no backdrop ou no ✕

**Botão FAQ fixo (🐠 ME PERDI, E AGORA?):**
- Posição fixa no canto superior direito
- Abre painel accordion com 3 perguntas fixas

**FAQ — 3 perguntas fixas:**
| Pergunta | Resposta |
|---|---|
| Me perdi, onde ele está agora? | "Clique na **SEREIA**" — palavra clicável que fecha o FAQ e faz scroll suave até o personagem |
| Onde está o link dos materiais? | Link clicável para `vtsd.com.br/materiais-ia` |
| A trilha não atualiza, o que faço? | "Atualize a página." |

**Tela de nome (primeiro acesso):**
- Overlay em tela cheia sobre a trilha
- Campo de texto: "👋 OLÁ! QUAL É O SEU NOME?"
- Botão "MERGULHAR →" (ou Enter)
- Nome salvo em `localStorage` como `trilha_name`
- Saudação aparece no header e persiste entre sessões

**Som de avanço:**
- Toca automaticamente quando `step` aumenta
- Sequência de 3 notas (523 Hz → 659 Hz → 784 Hz) via Web Audio API
- Sem dependência externa

---

## 3. Firebase

| Campo | Valor |
|---|---|
| Projeto | `passo-a-passo-eventos` |
| URL | `https://passo-a-passo-eventos-default-rtdb.firebaseio.com` |
| Path | `/trilha.json` |
| Modo de acesso | REST puro (sem SDK) |
| Regras | Teste (abertas) |
| Método de leitura | GET com `?_=timestamp` para evitar cache |
| Método de escrita | PUT (sobrescreve o JSON completo) |

### Estrutura do JSON no Firebase

```json
{
  "steps": [
    { "id": 1, "type": "Aula", "title": "Introdução", "text": "Texto opcional para o participante" }
  ],
  "step": 3,
  "cps": {
    "2": { "name": "Como acessar", "desc": "Passo 1: ...\nPasso 2: ...", "link": "https://..." }
  },
  "titles": {
    "1": "Título customizado ao vivo"
  },
  "oferta": false
}
```

| Campo | Tipo | Descrição |
|---|---|---|
| `steps` | Array | Lista de etapas configuradas (recriada a cada SALVAR) |
| `step` | Number | Quantidade de etapas liberadas (0 = nenhuma) |
| `cps` | Object | Conteúdo dos checkpoints indexado por `id` da etapa |
| `titles` | Object | Overrides de título editados ao vivo (só os diferentes do padrão) |
| `oferta` | Boolean | `true` ativa o overlay de oferta na tela do participante |

---

## 4. Identidade Visual

### Tema
**Mergulhando na IA** — ambiente submarino profundo com bioluminescência teal e laranja.
Inspirado em imagens de Leandro como personagem submarino (sereia, caranguejo).

### Paleta CSS — Participante

| Variável | Valor | Uso |
|---|---|---|
| `--bg` | `#020D14` | Fundo principal (oceano profundo) |
| `--bg2` | `#041824` | Fundo de cards e painéis |
| `--road` | `#052840` | Interior da estrada |
| `--edge` | `#083858` | Borda externa da estrada |
| `--dash` | `#00C8D4` | Linha tracejada central |
| `--bdr` | `#0A3D5C` | Bordas de elementos |
| `--text` | `#C8E8F4` | Texto principal |
| `--muted` | `#2A6080` | Texto secundário / desabilitado |
| `--acc` | `#00C8D4` | Destaque teal/ciano (botões, bordas ativas, glows) |
| `--glow` | `#FF7A20` | Laranja bioluminescente (hover, gradiente da barra) |

### Paleta CSS — Admin
Admin mantém tema original neutro (GitHub dark): `--bg:#0D1117`, `--acc:#E8B500`, fundo cinza escuro.

### Tipografia
**Press Start 2P** (Google Fonts) — estética pixel art em todos os títulos, labels, badges e botões.
Fallback: `monospace`.
Corpo de texto: `-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`.

### Personagem — Leandro Sereia

Pixel art SVG inline, viewBox `0 0 18 36`, renderizado em 45×90px:

| Parte | Detalhes |
|---|---|
| Cabelo | Cacheado escuro (`#2E1A00`), fluindo nas laterais (cols 0–2 e 15–17) |
| Rosto | Pele clara (`#E8B08A`), expressão surpresa |
| Sobrancelhas | Levantadas e inclinadas (preocupado/surpreso) |
| Olhos | Arregalados, branco + pupila preta |
| Boca | Aberta (surpreso) |
| Sem barba | Pele exposta do queixo ao pescoço |
| Top | Conchas lilás/pérola (`#C8A0D8`) com destaque nacarado |
| Braços | Pele clara, estendidos lateralmente |
| Barriga | Exposta, pele clara (ref. à imagem original) |
| Cauda | Teal iridescente (`#0AB8B8`) com padrão de escamas alternadas |
| Nadadeira | Dupla, na base, em teal escuro |
| Label | "LEANDRO" — tag em teal (`#00C8D4`), posicionada em `top: -108px` (acima da cabeça) |
| Animação | `char-bob`: flutua 6px para cima/baixo em 1.4s infinito |
| Efeito pop | Escala 1.3 + salto de 12px ao avançar |
| Glow | `drop-shadow(0 0 8px #00C8D4)` + halo difuso |

### Emojis por tipo de etapa

| Tipo | Emoji | Cor do badge |
|---|---|---|
| PDF | 🐚 | `#FF7B72` |
| Aula | 🐬 | `#00C8D4` |
| Doc | 🐠 | `#56D364` |
| Site | 🦑 | `#7ECFFF` |
| Skill | 🦀 | `#FF9A50` |
| Planilha | 🐙 | `#39D4C5` |
| Prompt | ⚓ | `#F778BA` |

---

## 5. Lógica de Negócio

### Participante — `pullState()`
Executado na inicialização e repetido a cada **3 segundos**:
1. GET no Firebase com cache-buster
2. Se `data.steps` mudou → reconstrói `STEPS` e `CPS`
3. Se `data.step` mudou e aumentou → toca som de avanço
4. Atualiza `unlocked`, `CPS`, `TITLES`
5. Se `data.oferta === true` → ativa overlay de oferta
6. Se houve mudança → `buildTrail()` + `updateCharacter(true)`
7. Atualiza ponto de sync (verde/cinza)

### Admin — fluxo de escrita
- Toda escrita é um **PUT** com o JSON completo (não PATCH parcial)
- `buildPayload()` compila o estado atual de `STEPS`, `unlocked`, `CPS`, `TITLES`, `oferta`
- `saveAll()` (aba Configuração) sempre reseta `step: 0` e limpa `cps` e `titles`
- `pushState()` (controles ao vivo) preserva todos os campos e apenas atualiza `step`, `cps`, `titles`

### Posicionamento na trilha
```
N = STEPS.length × 2 + 1   (total de pontos)

i=0          → INÍCIO (texto fixo, clicável)
i=1,3,5...   → paradas (stops) com emoji + dot
i=2,4,6...   → checkpoints com quadrado clicável
i=N-1        → CHEGADA (flag + ícone)

Personagem: idx = unlocked === 0 ? 0 : min(2×unlocked - 1, N-2)
```

---

## 6. Deploy e Infraestrutura

| Item | Detalhe |
|---|---|
| Repositório | `anabeperes/Passo-a-passo-eventos-` (GitHub público) |
| Hospedagem | Netlify — deploy automático no push para `main` |
| Branch de desenvolvimento | `claude/modest-maxwell-BRhTe` → merge em `main` para publicar |
| Domínio | URL gerada pelo Netlify (compartilhar com participantes) |
| Firebase | Regras abertas — qualquer pessoa pode ler/escrever (suficiente para eventos fechados com URL não pública) |
| Dependências externas | Apenas Google Fonts (Press Start 2P) — sem npm, sem build |

---

## 7. Configuração Rápida por Evento

Para reutilizar em outro evento, alterar somente:

| O quê | Onde | Exemplo |
|---|---|---|
| Título do header | `CFG.title` no participante | `"🌊 MERGULHANDO NA IA — DIA 03"` |
| URL do Firebase | `CFG.firebase.url` nos dois arquivos | `"https://meu-projeto.firebaseio.com"` |
| Nome do personagem | `label.textContent` em `initCharacter()` | `'LEANDRO'` |
| Link dos materiais | `FAQ_ITEMS[1].a` | URL do Google Drive / Notion |
| Senha do admin | `CFG.password` no admin | `"123"` |

---

## 8. Limitações Conhecidas

| Limitação | Impacto | Mitigação atual |
|---|---|---|
| Firebase em modo teste (regras abertas) | Qualquer pessoa com a URL pode sobrescrever dados | Evento tem URL não pública; acesso ao admin exige senha |
| Polling a cada 3s (não WebSocket real) | Pequeno atraso na atualização | Aceitável para o formato de evento |
| Senha do admin em texto no HTML | Visível no código-fonte | Senha simples, arquivo não indexado |
| PUT completo a cada ação | Sobrescreve tudo a cada clique | Sem conflito pois há apenas um admin ativo |
| Nomes no localStorage | Persiste entre eventos no mesmo dispositivo | Participante pode limpar manualmente |
