# Contexto do Projeto: Trilha de Evento ao Vivo

## O que é

Sistema de trilha interativa para mentorias/eventos online. Dois arquivos HTML independentes, sem backend próprio — usam Firebase Realtime Database via REST.

---

## Arquivos

### `trilha-admin.html`
Painel protegido por senha (`123`). Duas abas:

- **⚙ CONFIGURAÇÃO** — monta as etapas da trilha: tipo (parada/checkpoint), nome, texto, ordem. Botão SALVAR grava tudo no Firebase.
- **🎮 CONTROLE** — usado durante o evento: libera etapas uma a uma (LIBERAR / VOLTAR), edita título de qualquer etapa inline (✏), edita checkpoints (nome + descrição + link).

### `trilha-participante.html`
Tela somente leitura para os participantes. Lê o Firebase a cada 3 segundos. Mostra:
- Estrada serpentina em SVG
- Personagem pixel art que avança conforme etapas são liberadas
- Labels clicáveis nas paradas (abre bottom sheet com o texto da etapa, se houver)
- Quadrados de checkpoint clicáveis (abre bottom sheet com nome + desc + link)
- Botão fixo "📖 ME PERDI, E AGORA?" com FAQ accordion

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
  "steps": [{ "id": "s1", "type": "stop|checkpoint", "title": "Nome", "text": "Texto opcional" }],
  "step": 0,
  "cps": { "s2": { "name": "Nome CP", "desc": "Descrição", "link": "https://..." } },
  "titles": { "s1": "Título override" }
}
```

---

## Lógica principal

- `N = STEPS.length * 2 + 1` — número de pontos na trilha (dinâmico)
- `STEPS` é carregado do Firebase em todo pull (não hardcoded)
- `getTitle(id)` — retorna override de `TITLES` ou título padrão do step
- `pullState()` — lê Firebase, reconstrói STEPS/CPS/TITLES, chama `buildTrail()` + `updateCharacter()`
- `buildTrail()` retorna "AGUARDANDO CONFIGURAÇÃO..." se `STEPS.length === 0`

---

## Deploy

- Subir os dois HTMLs em repositório GitHub
- Conectar ao Netlify (deploy automático)
- Qualquer pessoa com o link público acessa os painéis pelo navegador
- Para editar o código: precisa ser colaborador no repositório GitHub

---

## FAQ do participante (3 itens fixos)

1. **Me perdi, onde ele está agora?** → Clique no bonequinho na trilha.
2. **Onde está o link dos materiais?** → Clique no quadrado entre as paradas.
3. **A trilha não atualiza, o que faço?** → Recarregue a página (sincroniza a cada 3 segundos).

---

## Paleta CSS

| Variável | Valor |
|---|---|
| `--bg` | `#0D1117` |
| `--bg2` | `#161B22` |
| `--acc` | `#E8B500` |
| `--muted` | `#484F58` |
| `--bdr` | `#30363D` |
| `--text` | `#C9D1D9` |

Fonte: **Press Start 2P** (Google Fonts) — estética pixel art.
