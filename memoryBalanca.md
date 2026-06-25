# memoryBalanca.md — Histórico do Projeto Balança de Equações

## Sessão 1 — Fundação e evolução principal (junho 2026)

### O que foi feito
- HTML single-file criado do zero: balança CSS com divs + transform rotate
- Estado `{left, right}` com x, n5, n; botões +/− para cada tipo de peso
- Modo Livre: parser `parseEqString()` aceita `3x+2=11`, `4x=2x+8`, etc.
- Passos algébricos: sistema `logActive`/`solveLog`, checkpoints só no equilíbrio
- `describeStep()` detecta operação entre dois estados consecutivos
- Painel de passos fixo à direita, aparece após vitória

### Decisões tomadas
- Registrar passos APENAS quando balança equilibrada (não a cada clique)
- Descrição do passo aparece na linha da equação que ela descreve (offset +1 no render)
- Modo Livre: Reset em vez de Próximo Desafio; sem contador; seção Peso 10 adicionada
- Esquema de cores por tipo: roxo (x), azul (1), cinza (5) — balões com mesmas cores dos pesos

---

## Sessão 2 — Redesign visual (junho 2026)

### O que foi feito
- Substituição do design CSS por balança SVG dourada
- Viga: gradiente dourado horizontal, esfera pivô central, hooks nas pontas
- Tentativa 1: pratos pendurados com cordas em V (SVG com preserveAspectRatio=none)
- Problema: cordas não alcançavam o prato quando muitos itens empilhados
- Tentativas de fix: margin-bottom negativo, position:absolute na corda — todas parciais
- Solução definitiva: pratos ACIMA da viga, sem cordas
- Base: SVG cinza escuro com 6 furos retangulares + pilar dourado fino (estilo balança de farmácia)
- Layout 3 colunas: botões fixos à esquerda, balança no centro, passos à direita
- Modo drag & drop: paleta com pesos positivos, balões negativos e lixeira 🗑️

### Decisões tomadas
- Pratos acima da viga: elimina problema de corda, mais simples, visualmente limpo
- `bottom:0` nos pan-assemblies: âncora na viga, itens crescem para cima
- Margem dinâmica `marginTop` cresce conforme itens se acumulam
- Velocidade de inclinação: 0.7s original → 1.4s (50% mais lento, pedido do Alexis)
- Painéis fixos esquerdo/direito: `body { padding: 20px 240px }` garante que conteúdo central não sobreponha

### Problemas encontrados
- Cordas em V: tentamos margin-bottom negativo e position:absolute — nenhum funcionou bem com muitos itens. Solução foi eliminar as cordas mudando o design dos pratos.
- Pilar SVG aparecia muito pequeno: aumentado de 130×78 para 300×112.

---

## Sessão 3 — Personagens, Tela 3 e polimento visual (junho 2026)

### O que foi feito
- Removido fundo branco de `Griffin wins NoBG.png` e fundo vermelho/checkerboard de `Phoenix Wins NoBG.png` via C# BFS/flood-fill inline no PowerShell
- Removido fundo branco de `Ugabuga NoBG.png` via C# BFS
- Verificadas e confirmadas todas imagens em `Imagens/` como WebP
- Tela 1 e Tela 2 atualizadas com imagens WebP embutidas como base64 no HTML
- Adicionada **Tela 3**: overlay fullscreen com wallpaper do personagem escolhido, clique para entrar no jogo
  - Solução: 4 `<img>` pré-embutidos no HTML com `src="data:..."` (não via JS — chrome bloquia data URIs grandes via JS em file://)
  - CSS `display:block !important` para sobrepor `display:none` inline
- Removido badge "CENTAUR/GRIFFIN/etc." do header
- Título centralizado com 2 sprites do animal escolhido flanqueando (um de cada lado, virados para o título)
- Animal do personagem adicionado acima do pivô da balança (104×104px)
- Placar: sprite do personagem no lado ✅, Ugabuga no lado ❌
- Removido botão Projetor
- Score + Tela Cheia movidos para canto fixo superior direito (`position:fixed; right:1rem; top:1rem`)
- Emoji ⚖️ removido do título
- Sprite do pivô dobrado de tamanho (56→104px)

### Arquitetura dos sprites
- `House Animals.webp` 1024×1024 com 4 animais em grid 2×2 (griffin, dragon, phoenix, centaur)
- CSS: `background-position` em `200% 200%` para cada quadrante
- `.flip` = `transform: scaleX(-1)` para espelhar
- Classe `.sm` para versão 36×36 no placar
- `--uga` = Ugabuga.webp para sprite dedicado (36×36, cover)

### Orientação dos sprites por personagem (flanqueando o título)
- griffin: L=normal(→), R=flip(←)
- dragon: L=flip(→), R=normal(←)
- phoenix: L=normal, R=normal (simétrico)
- centaur: L=flip(→), R=normal(←)

### Problema crítico encontrado e resolvido
- Chrome bloqueia data URIs grandes setados via JS em `file://` (falha silenciosa em `img.src = 'data:...'` e `style.backgroundImage`)
- Solução: embutir `src="data:..."` diretamente no atributo HTML, nunca via JavaScript
- Isso afetou tanto o fundo da Tela 3 quanto a tentativa de wallpaper dinâmico

### Decisões tomadas
- Abandonada ideia de wallpaper com transparência atrás do jogo (complexidade e protocolo file://)
- Tela 3 como tela de transição (fullscreen, clique para jogar) em vez de background contínuo
- Servidor Python `http.server 7823` para testes autônomos via Claude Preview MCP

### Pendências
- Telas de vitória ("Wins") com NoBG ainda não embutidas no jogo — próxima sessão

---

## Sessão 4 — Vitória, áudio e polimento (junho 2026)

### O que foi feito
- **Bug do sprite corrigido**: `chooseChar('Griffin')` usava maiúscula mas as classes CSS são minúsculas (`.griffin`). Todos os 4 personagens estavam virando griffin. Corrigido para minúsculas.
- **Tela 3** ganhou zoom-out/scroll para mostrar a imagem inteira do wallpaper.
- **Crédito "by Hukai"** no canto inferior esquerdo de todas as telas (sem cobrir nada).
- **Alinhamento**: personagem mítico e Ugabuga reposicionados (estavam altos demais).
- **Overlay de vitória**: além dos confetes, a imagem "Wins" do animal dá _pop_ na tela com 1–3 estrelas (por eficiência algébrica) e some no clique. Espera 1,5s antes de aparecer (`setTimeout(launchWinPop, 1500)` dentro de `launchConfetti()`).
- **Sistema de áudio completo** (tudo embutido em base64, sem download):
  - Intro (Old School) ao abrir, troca para APT (fade) ao entrar na balança
  - SFX sintetizados via Web Audio API (toc, fiu, ding, fanfarra, buzz, swoosh, arpejo)
  - Gritos dos animais embutidos (Griffin.mp3, Dragon.wav, Phoenix.m4a, Centaur.mp3)
  - Botão 🔊/🔇; ducking da música quando toca grito; pausa em `visibilitychange`
- **Pivô anima**: `festeja` (vitória) / `nega` (erro).
- **Transições de fade** entre telas.

### Problemas encontrados e resolvidos
- **Script quebrava no load** (zonas não clicáveis, intro muda): o módulo `sfx` foi inserido no meio do script, e `newGame()` inicial chamava `render()→sfx.ding()` antes de `sfx` existir. **Fix: módulo sfx PRECISA ficar no TOPO do `<script>`.**
- **Estrelas sempre davam 2, nunca 3**: `solveLog[0]` é o estado inicial (não um passo do aluno). Fix: `passos = Math.max(0, solveLog.length - 1)`.
- **Intro não tocava**: `startIntroMusic()` rodava antes da tag `<audio>` existir no DOM. Fix: `DOMContentLoaded`.
- **Ding em equação nova** (já equilibrada): `render()` dispara no load. Fix: flag `window._equilibrado = true` em `newGame()`.
- **Placar contava duas vezes**: clicar "Verificar" repetido incrementava. Fix: boolean `currentSolved`, resetado em `newGame()`.

---

## Sessão 5 — Versão Aluno × Professor, persistência (junho 2026)

### O que foi feito
- **Dois arquivos**: `equacaoprof.html` (tudo liberado, sem trava) e `index.html` (versão ALUNO).
- **Níveis travados (aluno)**: `const ACERTOS_PARA_PASSAR = 6` — precisa de 6 acertos para liberar o próximo nível. `levelProgress = {1..5}`. Toast de desbloqueio + arpejo.
- **Filtro de exercícios**: `isAlreadyIsolated()` evita gerar equações que "já estão prontas" (x já isolado).
- **Persistência por animal** (só aluno):
  - `chaveSave()` retorna `'balanca_save_' + animal` → cada animal salva separado
  - `salvarProgresso()` grava `{lp: levelProgress, sc: score}`; `temProgresso()` checa se há w>0 ou l>0
  - Na tela 3, se há progresso: caixa "Voltar de onde parou?" (Sim restaura / Recomeçar zera com `removeItem`)
- **Dobro de exercícios** em todos os níveis.
- **Nomes dos alunos** (lista de 35) no lugar de nomes genéricos nas equações-problema.
- **Botão ⏭ TESTE** (temporário) para pular nível durante testes — `pularNivelTeste()`.

### Decisões
- Progresso por animal: cada aluno "evolui" cada criatura separadamente.
- `salvarProgresso()` deve ser chamado DEPOIS de `levelProgress[level]++` (havia bug salvando nível errado).

---

## Sessão 6 — Sistema de auras evolutivas (junho 2026)

### O que foi feito
- **Auras por animal** aplicadas ao pivô e ao sprite do placar via `aplicarAura()`, conforme níveis completados.
- Cores temáticas: griffin=azul/elétrico, dragon=fogo, phoenix=dourado, centaur=verde/natureza.
- Níveis 1–3: glow crescente (drop-shadow). Nível especial: animação única por animal (raios, chama, estrelas, raios dourados).
- **Sprites finais + imagens de aura** (pasta `Imagens/`): `*_final.webp` (sprite evoluído) e `*-aura.webp` (halo).

### Reestruturação das auras (versão atual)
Sistema de 6 estágios baseado em quantos níveis foram completados (`aplicarAura()`):
| Completos | Classe | Visual |
|---|---|---|
| 1 | aura-1 | glow fraco |
| 2 | aura-2 | glow médio |
| 3 | aura-4 | animação especial (raios/chama/estrelas) |
| 4 | aura-5 | sprite trocado pela imagem `_final` |
| 5 | aura-6 | imagem `_final` + aura `*-aura.webp` piscando atrás |

### Problemas encontrados e resolvidos
- **Camadas erradas** (aura cobria o sprite): o `background-image` do próprio div é pintado ATRÁS do `::before`. Fix: div com `background-image:none`, **aura no `::before` (atrás), sprite no `::after` (frente)** — o `::after` sempre pinta na frente do `::before`.
- **`:root` corrompido / sprites sumiram**: o regex de injeção das CSS vars parou no `;` dentro de `data:image/webp;base64`, cortando `--win-centaur` no meio. **Fix: nunca usar regex que pare em `;` ao injetar data URIs — usar `IndexOf`/`Insert` com âncoras de var.**
- **Centaur sem som** (`PIPELINE_ERROR_DECODE`): `Centaur.m4a` tinha codec AAC que o Chrome não decodifica. Fix: re-exportado como **Centaur.mp3** (MIME `audio/mpeg`).

---

## Sessão 7 — Auras dedicadas do Dragão (junho 2026)

### O que foi feito
Sistema de aura **específico do dragão** (pasta `Imagens/Dragon/`), substituindo o genérico só para ele. Aura sempre ATRÁS (`::before`), sprite sempre À FRENTE (`::after`):
| Estado | Sprite (::after) | Aura (::before) |
|---|---|---|
| Nível 4 unlock | dragão normal (spritesheet `--ha` 100% 0%) | `Dragon-aura 2` estática |
| Nível 5 unlock | `Dragon 2` (var `--d-sprite2`) | `Dragon-aura 2` estática |
| Livre unlock | `Dragon 3` (var `--d-sprite3`) | alterna `Aura 3a`↔`Aura 3b` (var `--d-aura3a/3b`), fogo mexendo |

- **Animação do fogo**: `@keyframes dragonFogo` troca `background-image` entre 3a e 3b a cada 0,45s (`steps(1)`).
- **Destaque do dragão** (a paleta verde/fogo se confundia): `::after` ganhou `drop-shadow` preto duplo/triplo + `contrast`/`saturate`/`brightness`. Nível 5 e Livre também ficaram maiores (`inset:-14%` e `-12%`).
- **Aura do Livre subiu**: `top:-52%; bottom:4%` — base do fogo alinhada com a base do dragão, fogo sobe.
- **Clique no dragão liga/desliga a aura**: `toggleRaios()` agora vale em aura-4/5/6; classe `.sem-raios` esconde só o `::before` (sprite `::after` continua via `display:block !important`).

### Vars CSS novas (`:root`)
`--d-aura2, --d-sprite2, --d-sprite3, --d-aura3a, --d-aura3b` (imagens da pasta `Imagens/Dragon/`).

### Pendências
- **Replicar o sistema dedicado para griffin, phoenix e centaur** (Alexis está preparando os sprites).
- **Remover o botão ⏭ TESTE** e confirmar `ACERTOS_PARA_PASSAR = 6` antes do deploy final (Netlify).

### Regra importante (registrada na CLAUDE.md raiz)
- **NUNCA ligar o Preview/Prévia sem perguntar ao Alexis** — fica rodando por trás (música em loop sem parar).

---

## Sessão 8 — Auras dedicadas: phoenix, griffin, centaur + limpeza (junho 2026)

### O que foi feito
- Replicado o **sistema dedicado de auras** (criado para o dragão na S7) para os outros 3 animais. Padrão idêntico para todos: aura no `::before` (atrás), sprite no `::after` (frente), div com `background-image:none`.
- Cada animal tem pasta própria em `Imagens/` (`Phoenix/`, `Griffin/`, `Centaur/`) com: `<animal> aura 2` (estática, níveis 4-5), `<animal> 2` (sprite nível 5), `<animal> 3` (sprite Livre), `<animal> aura 3a/3b` (animadas, alternam no Livre).
- Mapa de cada animal (igual ao dragão):
  - aura-4 (3 níveis): sprite normal (spritesheet) + aura 2 estática
  - aura-5 (4 níveis): sprite "2" + aura 2 estática
  - aura-6 (Livre): sprite "3" + aura 3a↔3b alternando (`@keyframes <animal>Fogo/Raio/Aura`, `steps(1)` 0.45s)
- Vars CSS por animal: `--p-*` (phoenix), `--g-*` (griffin), `--c-*` (centaur), `--d-*` (dragão).
- **Phoenix nível 5** reusa `--fin-phoenix` (já existente). Griffin/centaur usam sprites "2" próprios.
- **Destaque do sprite** (paleta se confundia com a aura de fogo): `::after` ganhou contorno preto em 4 camadas (`drop-shadow` 1+2+4px + halo) + `contrast`/`saturate`/`brightness`. Sprite maior via `inset` negativo.
- **Limpeza de peso**: removidas 7 CSS vars mortas (`--fin-griffin/dragon/centaur`, `--aura-griffin/dragon/phoenix/centaur`) → −2,83 MB. Mantida `--fin-phoenix` (em uso). Backup em `index.bak.html`.

### Cuidados/decisões
- `@keyframes auraPisca` ficou órfã (sem referência) — inofensiva, mantida.
- Typo no arquivo do griffin: `Grififn aura 3a.webp` (mantido como está no disco).
- Sempre verificar vars antes de remover (def vs uso) — confirmamos 0 usos.

---

## Sessão 9 — Reorganização do layout (caber em 1 tela) (junho 2026)

### O que foi feito
- **Divisória vertical** dourada entre os painéis "Lado Esquerdo" e "Lado Direito" (`.painel-divisoria`).
- **Barra de níveis** movida para o canto superior esquerdo, em coluna vertical fixa (`#level-bar` fixed, flex-column, top-left) — aproveita o espaço ocioso e libera vertical.
- **Caixa da missão** ("Encontre o valor de x") movida para o topo-esquerda, ao lado da coluna de níveis (`#mission` fixed). `box-sizing:border-box` + `max-width:215px` garante que termina ANTES da zona central (380px do `padding-left`), sem tapar o título — importante no nível 4/5 onde o texto é mais longo.
- **Painéis de botões** (`#controls`) descidos para `top:17rem` (abaixo da coluna de níveis), posição fixa em rem (estável). Antes era `top:50%` centralizado e a coluna de níveis cobria os botões.
- **Compactação vertical**: body padding 20→14px; h1 1.7→1.5rem; margens de header/level-bar/mission/status reduzidas; `#scale-wrap` margens 110/150 → 84/112px (maior ganho).
- **Miolo deslocado p/ direita**: `padding-left` 340→380px (sem mexer no direito 240px → região dos passos intacta).

### Nota importante
- O screenshot do usuário está em pixels de dispositivo com escala Windows ~150% → 1rem(16css px) ≈ 24 device px. Considerar isso ao interpretar posições do screenshot vs valores CSS.

---

## Sessão 10 — Sistema de músicas desbloqueáveis + mute separado (junho 2026)

### O que foi feito
- **Playlist por progressão** (`MUSICAS[]` com campo `destrava`): APT (base), Actually Romantic (nível 3), Soda Pop (nível 5), Sigma Boy (Livre). Ao desbloquear, a música nova toca automaticamente em loop com fade (`tocarMusicaJogo`).
- **Gancho** em `showUnlockToast(lv)` → `desbloquearMusica(lv)`. Também adicionado em `pularNivelTeste` (o botão TESTE não chamava showUnlockToast, por isso a música não tocava ao testar).
- **Botão 🎶⏭ trocar música** (`#btn-music-next`, `proximaMusica()`) aparece quando há 2+ músicas desbloqueadas. `atualizarBotaoTrocar()` chamado em `refreshLevelButtons()` (robusto, reflete o estado sempre).
- **Mute separado em 2 botões**: `#btn-mute` (🎵/🔇) só música (`musicaMutada`); `#btn-mute-sfx` (🔔/🔕) só efeitos/animações (`efeitosMutados`). Antes um único botão tirava tudo. Os gates dos efeitos passaram de `musicaMutada` para `efeitosMutados` (sfxTone, sfxNoise, sfx.animal).
- `visibilitychange` generalizado para pausar/retomar todas as músicas.
- **Músicas grandes referenciadas (não embutidas)**: as 3 novas (~12 MB) viram `src="Audio/...mp3"` com `preload="none"` (carregam sob demanda). HTML caiu de 33 MB → 18 MB. APT/intro/sprites/efeitos seguem embutidos. ⚠️ Os 3 arquivos NÃO podem ser renomeados/movidos (quebra o link). O Alexis sobe a pasta inteira no Netlify a cada versão.

### Ajustes finos de sprite (após teste visual)
- **Dragão Livre**: sprite maior (`inset:-20%`) e clareado (brightness 1.42, contrast baixo, halo escuro reduzido) — estava escondido/escuro demais na aura de fogo.
- **Centauro Livre**: `transform:scaleX(1.25)` (corrige achatamento) e aura descida (`top:-38% bottom:-10%`).

### Pendências para o deploy final
- Remover botão ⏭ TESTE.
- Confirmar `ACERTOS_PARA_PASSAR = 6`.
- Subir a pasta `EquacaoBalanca` inteira no Netlify (HTML + Audio + Imagens).

---

## Sessão 11 — Versões de teste, tema estético e responsividade (junho 2026)

### Arquivos da pasta agora
- `index.html` — PRODUÇÃO (6 acertos, sem botão TESTE, **responsivo**)
- `teste.html` — cópia do index + camada de tema estético "v2/v3" (fontes Fredoka/Inter, glassmorphism, favicon, etc.). **Alexis NÃO gostou** — não usar como base.
- `versaoteste.html` — cópia do **index (produção)** com `ACERTOS_PARA_PASSAR = 1` + botão ⏭ TESTE, para testar progressão rápido.

### O que foi feito
- **Responsividade (só index, sem mexer no app)**: camada de `@media query` no fim do CSS. Acima de 1100px o desktop fica idêntico; abaixo, os painéis fixos viram `position:static` e empilham em fluxo na ordem do DOM (cabeçalho→níveis→missão→balança→botões→verificar→passos). Balança encolhe com `zoom` (0.8 tablet / 0.56 phone). `overflow-x:hidden`.
- Ajuste mobile: balança desce (`margin-top` maior) p/ não tampar o status; telas de imagem com `object-fit:contain` (mostram inteiras).
- Tema estético (teste.html): tokens de cor, fundo com gradiente/vinheta, glass nos painéis, botões padronizados, status como chip, overlay de vitória polido, scrollbar, favicon ⚖️, fontes Google. (Reprovado pelo Alexis.)

### Bug encontrado e resolvido
- **Tela 1 não clicava** (não avançava): a regra mobile `#char-cards{ pointer-events:auto }` reativava cliques no `#char-cards` (cobre a tela toda, sobre a tela1) mesmo com a tela2 `pointer-events:none` — interceptava o clique. ⚠️ Lição: `pointer-events:auto` num filho de elemento com `pointer-events:none` RE-ATIVA o clique do filho. Removida.

### Pendências
- Confirmar com Alexis QUAL arquivo é o oficial p/ editar (estava editando index, ele testou cópia em outra pasta).

### Ajustes finais desta sessão (desktop + mobile)
- **Margem dinâmica da balança** (`ajustarMargemBalanca`, chamada no render + setTimeout 1450 pós-inclinação): mede o topo REAL dos pratos (`getBoundingClientRect`, já inclui itens empilhados + inclinação) e empurra a balança p/ baixo só o necessário p/ não tapar o status/título. Substituiu a heurística antiga por contagem de itens (que ignorava a inclinação). Transição `margin-top 0.35s`.
  - Funciona no **mobile também**: corrige o `zoom` dividindo o delta medido por `z` (`getComputedStyle.zoom`); inclui o `#pivot-animal` na medição no mobile (era ele que tapava o status nos prints). Bases: 84 desktop / 120 tablet / 110 phone. ⚠️ Tirado o `!important` do margin-top mobile (bloqueava o ajuste); margin-bottom mobile segue `!important`.
- **Tela 3 (wallpaper "clique para jogar")**: as `<img id="t3-*">` tinham `object-fit:cover` INLINE (cortava a imagem). Forçado `object-fit:contain !important` global → wallpaper inteira. Mobile já estava contain (media query), desktop estava cortando.
- **Trava de scroll nas telas de intro**: `<html class="telas-lock">` + `html.telas-lock{overflow:hidden}`; `startGame()` remove a classe (jogo volta a rolar). Evita scrollbar/imagem cortada nas telas 1/2/3.

---

## Como usar este arquivo
A cada sessão com mudanças relevantes, registrar:
- O que foi feito
- Decisões tomadas (e por quê)
- Problemas encontrados e como foram resolvidos
- Pendências para a próxima sessão
