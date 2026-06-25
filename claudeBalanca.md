# claudeBalanca.md — Balança de Equações

App educativo interativo em HTML single-file para ensino de equações de 1º grau (7º ano).

## Arquivos
- `index.html` — versão ALUNO / PRODUÇÃO (6 acertos desbloqueiam o próximo; persistência; estrelas; auras; **responsivo**)
- `equacaoprof.html` — versão PROFESSOR (tudo liberado, sem persistência)
- `teste.html` — index + tema estético experimental (REPROVADO pelo Alexis; não usar de base)
- `versaoteste.html` — cópia do index com `ACERTOS_PARA_PASSAR=1` + botão ⏭ TESTE (testar progressão rápido)
- `Audio/` — efeitos/intro/APT embutidos; **músicas grandes (Romantic/Soda/Sigma) referenciadas** `src="Audio/..."` (subir a pasta inteira no Netlify)

## Balança: posição dinâmica (não tapar o título)
- `ajustarMargemBalanca()` mede o topo real dos pratos (`#pan-left/right-assembly`, via getBoundingClientRect — já reflete itens + inclinação) e ajusta `#scale-wrap` margin-top p/ ficar logo abaixo do `#status`. Chamada no fim do render + `setTimeout 1450` (pós-inclinação). Transição suave.
- Mobile: corrige `zoom` (delta/z) e inclui `#pivot-animal` na medição. ⚠️ margin-top mobile SEM `!important` (senão trava o JS); margin-bottom mobile COM `!important`.

## Telas de intro (tela1/2/3)
- tela1/tela2: `<img object-fit:contain>` + char-zones (%); tela3: `<img id="t3-*">` por personagem — ⚠️ tinham `object-fit:cover` INLINE (cortava); forçado `contain !important` global.
- Scroll travado enquanto telas abertas: `<html class="telas-lock">`, `startGame()` remove a classe.

## Responsividade (só index, camada @media no fim do CSS)
- Desktop >1100px: idêntico ao original (regras não aplicam)
- ≤1100px: painéis fixos viram `position:static` e empilham na ordem do DOM; balança encolhe com `zoom`; `overflow-x:hidden`
- ⚠️ `pointer-events:auto` num filho de elemento `pointer-events:none` RE-ATIVA o clique do filho — cuidado com `#char-cards` (cobre a tela) sobre as telas de intro

## Sons
- Intro (Old School): toca ao abrir, 30%, começa aos 7s; troca para APT (fade) ao entrar na balança
- Efeitos sintetizados (Web Audio): toc/peso, fiu/balão, ding/equilíbrio, fanfarra/vitória, buzz/erro, swoosh/faca, arpejo/desbloqueio
- Gritos dos animais: arquivos embutidos (Griffin.mp3, Dragon.wav, Phoenix.m4a, Centaur.mp3), 70%, com ducking da música
- ⚠️ Centaur era .m4a e dava `PIPELINE_ERROR_DECODE` no Chrome (codec AAC) — trocado por Centaur.mp3 (MIME `audio/mpeg`). Preferir MP3/WAV para máxima compatibilidade.
- ⚠️ módulo sfx PRECISA ficar no topo do script (newGame inicial chama render→sfx)

### Playlist por progressão (`MUSICAS[]`, campo `destrava`)
- APT (base, sempre), Actually Romantic (nível 3), Soda Pop (nível 5), Sigma Boy (Livre)
- Ao desbloquear, toca automático em loop com fade (`tocarMusicaJogo`). Gancho: `showUnlockToast(lv)`→`desbloquearMusica(lv)` E `pularNivelTeste` (TESTE não passa por showUnlockToast)
- Botão 🎶⏭ (`#btn-music-next`, `proximaMusica()`) aparece com 2+ músicas; `atualizarBotaoTrocar()` roda em `refreshLevelButtons()`
- ⚠️ As 3 músicas grandes são REFERENCIADAS (`src="Audio/...mp3"` preload="none"), NÃO embutidas — HTML 18MB em vez de 33MB. Não renomear/mover os arquivos. Subir pasta inteira no Netlify.

### Mute separado (2 botões, canto sup. direito)
- `#btn-mute` (🎵/🔇) → só música (`musicaMutada`, muta todas as `<audio>` de música)
- `#btn-mute-sfx` (🔔/🔕) → só efeitos/animações (`efeitosMutados`); gates em sfxTone/sfxNoise/sfx.animal
- `visibilitychange` pausa/retoma todas as músicas quando aba oculta

## Vitória
- Confetes + overlay com imagem win do animal + estrelas (1–3 por eficiência: solveLog.length vs window._parPassos) — sai com clique
- Pivô anima: festeja (vitória) / nega (erro)

## Persistência (só index)
- localStorage `balanca_save_<animal>` {lp: levelProgress, sc: score} — UMA CHAVE POR ANIMAL (`chaveSave()`)
- `salvarProgresso()` grava; `temProgresso()` retorna true só se houver vitória/derrota real (sc.w>0 || sc.l>0)
- Na tela 3, se há progresso: prompt "Voltar de onde parou?" (`#resume-box`) — Sim restaura / Recomeçar faz `removeItem`
- ⚠️ chamar `salvarProgresso()` DEPOIS de `levelProgress[level]++` (não antes)
- Botão ⏭ TESTE (`pularNivelTeste()`) só seta levelProgress, NÃO conta vitória — por isso não dispara o resume-box

## Sistema de auras (aplicarAura)
- `aplicarAura()` conta níveis completados (≥ ACERTOS_PARA_PASSAR) e aplica classe ao pivô + sprite do placar
- Mapa: 1→aura-1, 2→aura-2, 3→aura-4, 4→aura-5, 5→aura-6
- **Regra de camadas**: div com `background-image:none`; AURA no `::before` (atrás); SPRITE no `::after` (frente). O `::after` sempre pinta na frente do `::before`.
- aura-1/2/3 = glow (drop-shadow)
- **TODOS os 4 animais têm sistema DEDICADO** (pastas `Imagens/Dragon|Phoenix|Griffin|Centaur/`). Vars por animal: `--d-*` (dragão), `--p-*` (phoenix), `--g-*` (griffin), `--c-*` (centaur).
  - aura-4 (3 níveis): sprite normal (spritesheet) + `<animal> aura 2` estática
  - aura-5 (4 níveis): sprite "2" + aura 2 estática (phoenix usa `--fin-phoenix`)
  - aura-6 (Livre): sprite "3" + `aura 3a↔3b` alternando (`@keyframes dragonFogo/phoenixFogo/griffinRaio/centaurAura`, steps(1) 0.45s)
- **Destaque do sprite** (paleta confundia com aura de fogo): `::after` com contorno preto 4 camadas + contrast/saturate/brightness; maior via `inset` negativo
  - Dragão Livre clareado (brightness 1.42, halo escuro reduzido); Centauro Livre `scaleX(1.25)` + aura mais baixa
- **Clique no animal** (`toggleRaios()`, aura-4/5/6): classe `.sem-raios` esconde a aura (`::before`), sprite continua via `display:block !important` no `::after`
- ⚠️ ao injetar data URIs no `:root`, NUNCA usar regex que pare em `;` (quebra `data:...;base64`). Usar `IndexOf`/`Insert` com âncoras.
- Vars `--fin-*`/`--aura-*` antigas removidas (só `--fin-phoenix` sobrou, em uso). `@keyframes auraPisca` órfã.
- Typo de arquivo: `Grififn aura 3a.webp` (griffin) — manter como está no disco.

## Layout (fixed nas bordas, cabe em 1 tela)
- `#level-bar`: coluna vertical fixa no topo-esquerda (`left:0.8rem; top:0.8rem`)
- `#mission`: fixa topo-esquerda ao lado dos níveis. `box-sizing:border-box; max-width:215px` para terminar ANTES da zona central (380px) e não tapar o título (importante no nível 4/5, texto longo)
- `#controls` (painéis de botões): fixa `left:1rem; top:17rem` (abaixo da coluna de níveis). `.painel-divisoria` = linha dourada entre Lado Esquerdo/Direito
- `#steps-panel`: fixa à direita. `body padding: 14px 240px 14px 380px` (esquerda 380 empurra miolo p/ direita; direita 240 reserva os passos)
- ⚠️ screenshots do Alexis em device px com escala Windows ~150%: 1rem(16css) ≈ 24 device px

## O que o app faz
Balança visual onde o aluno resolve equações colocando/removendo pesos e balões nos pratos. A balança inclina conforme o peso de cada lado. Objetivo: encontrar o valor de `x` equilibrando a balança.

## Estrutura do estado
```javascript
state = { left:{x, n5, n}, right:{x, n5, n} }
// x = caixas mistério, n5 = pesos de 5, n = pesos de 1
// Negativo = balões (leveza/subtração)
// weight(side) = state[side].x * xValue + state[side].n5*5 + state[side].n
```

## Modos
- **Níveis 1–5**: equações geradas automaticamente, dificuldade crescente
- **Livre (Nível 6)**: aluno digita equação (ex: `3x+2=11`), balança monta sozinha
- **Arrastar**: toggle drag & drop (botão na barra de níveis)

## Arquitetura visual
- Viga dourada SVG, gira com CSS transform; pratos ficam ACIMA (sem cordas)
- Base SVG cinza escura com 6 furos retangulares + pilar dourado fino
- Painel de botões fixo à ESQUERDA, painel de passos fixo à DIREITA
- `body { padding: 20px 240px }` evita sobreposição dos painéis fixos
- Ângulo máximo 15°; inclinação 1.4s cubic-bezier(0.34,1.56,0.64,1)

## Passos algébricos
- `logActive` / `solveLog[]`: checkpoints só quando balança equilibrada
- `describeStep(prev, curr)`: detecta ÷N, ±Nx, ±N entre dois estados
- Painel aparece após vitória; descrição na mesma linha da equação que descreve (offset +1)

## Parser (modo Livre)
- `parseEqString(str)`: aceita `3x+2=11`, `4x=2x+8`, etc.
- Valida: tem `=`, solução inteira, solução ≠ 0, x não some dos dois lados

## Drag & Drop
- `dragPayload = {palette, type, amt}` ou `{side, type}`
- Paleta: Pesos (+x/+5/+1) | Balões (−x/−5/−1) | Lixeira 🗑️
- Drop entre pratos = move; Drop na lixeira = remove

## Cores
- x: roxo `#7c3aed` com X vermelho | 5: cinza `#475569` | 1: azul `#0369a1`
- Balões: mesmas cores dos pesos | Viga/pratos: dourado | Base: `#2e3340`
