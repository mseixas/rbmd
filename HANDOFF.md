# Documento de Transição — Portfólio de Web Apps para Meta Ray-Ban Display (MRBD)

> **Propósito deste documento.** Ele existe para permitir que **outro assistente (Claude Code)** continue este projeto tendo *apenas* este arquivo como contexto. Está deliberadamente detalhado. Leia-o inteiro antes de escrever qualquer código.
>
> **Como usar no Claude Code.** Coloque este arquivo na raiz do repositório (sugestão: renomeie para `CLAUDE.md`, que o Claude Code carrega automaticamente como contexto do projeto). Mantenha junto o arquivo de referência da API `meta-rbd-webapps-referencia.md` (ver seção 11).
>
> **Idioma do projeto.** A interface dos apps é majoritariamente em **Português**; um app (Shadowbox) suporta PT/EN/ES. O código, comentários e este documento estão em português.

---

## ⚠️ AVISO CRÍTICO DE DEPLOY — LEIA PRIMEIRO

O briefing de transição mencionou *"deploy no Render"*. **Isto está incorreto para o estado atual do projeto.** Todo o portfólio foi construído e é publicado via **GitHub Pages**, não Render.

- **Não** existe backend, servidor Node, Dockerfile, `render.yaml`, build step, nem serviço web. São arquivos **HTML estáticos autocontidos**.
- Render é uma plataforma de *aplicações/serviços*; usá-la aqui seria overkill e mudaria o modelo mental do projeto sem necessidade.
- O **único** requisito de hospedagem da plataforma MRBD é **HTTPS com TLS válido** — e GitHub Pages já satisfaz isso gratuitamente.

**Se o dono do projeto realmente quiser migrar para o Render no futuro**, isso é possível (Render serve estáticos também, via "Static Site"), mas seria uma *decisão nova* — não assuma que já está assim. Enquanto não houver instrução explícita, **trate GitHub Pages como a plataforma de deploy oficial** (detalhes na seção 7).

---

## 1. Objetivo do sistema e contexto de negócio

### 1.1 O que é
Um **portfólio de pequenos web apps** para os óculos **Meta Ray-Ban Display (MRBD)**, usando a **Meta Wearables Web App API**. Cada app é uma ferramenta focada, single-purpose, pensada para uso *hands-free* e *glanceable* (informação que se olha de relance) no display embutido dos óculos.

### 1.2 Para quem / por quê
O dono do projeto ("o usuário") pratica **artes marciais** (boxe/muay thai) e **musculação**, e é **caçador licenciado**. Essas atividades motivam diretamente os apps: as ferramentas nascem de necessidades reais de treino e de campo. A tese central de produto é: *os óculos brilham quando entregam informação no campo de visão sem ocupar as mãos* — timer de round que você vê no canto do olho durante o soco, bússola de retorno ao ponto de partida, combos de shadowbox chamados na tela, etc.

### 1.3 Natureza do "negócio"
É um **projeto pessoal / portfólio de exploração de plataforma**, não um produto comercial com usuários pagantes. As metas são: (a) explorar o que a API dos óculos permite hoje; (b) construir ferramentas genuinamente úteis para o próprio uso; (c) validar na prática quais sensores/recursos funcionam bem no hardware real. Não há monetização, backend, contas de usuário, nem analytics.

### 1.4 Princípio de honestidade técnica
Uma diretriz recorrente e importante do projeto: **não prometer o que o hardware não entrega.** Vários recursos desejáveis (contar socos, contar repetições de musculação, áudio garantido) esbarram em limitações reais dos sensores. O projeto documenta essas limitações abertamente em vez de fingir que funcionam. **Mantenha essa postura** — se o usuário pedir algo que os sensores não suportam bem, diga isso claramente e ofereça a alternativa viável mais próxima.

---

## 2. Arquitetura e stack

### 2.1 Stack
- **HTML + CSS + JavaScript puro (vanilla).** Sem frameworks, sem build tools, sem dependências externas, sem npm.
- Cada app é **um único arquivo `index.html` autocontido** (HTML + `<style>` + `<script>` inline no mesmo arquivo).
- **Web APIs padrão do W3C** apenas. Não há SDK proprietário da Meta em runtime — os óculos rodam web comum.
- **Hospedagem:** GitHub Pages (estático, HTTPS).

### 2.2 Por que single-file vanilla
Decisão deliberada (ver seção 3). Resumo: portabilidade máxima, zero fricção de deploy (basta subir o arquivo), nada para "buildar" ou quebrar, e o runtime dos óculos é um navegador simples — não vale a pena introduzir complexidade de bundler para apps desse tamanho.

### 2.3 Estrutura do repositório
Repositório **guarda-chuva único** chamado `mrbd-apps`, com cada app em sua própria subpasta, cada uma contendo um `index.html`:

```
mrbd-apps/
├── index.html                 # Launcher: lista todos os apps com links (tela 600×600, D-pad)
├── nivel-bolha/index.html     # Nível de bolha / inclinômetro (sensor de orientação)
├── bussola/index.html         # "Volta pra Casa": bússola + waypoint (GPS + heading + storage)
├── contador/index.html        # Contador de movimento por picos de aceleração (acelerômetro + storage)
├── round-timer/index.html     # Timer de rounds para artes marciais (timing + beep + flash)
├── shadowbox/index.html       # Chamador de combos de shadowbox — APP MAIS COMPLEXO (ver 5.3)
├── shadowbox-mobile/index.html# Versão mobile responsiva (touch) do shadowbox
└── treino-musculacao/index.html # Timer de descanso entre séries + registro (storage)
```

> **Nota sobre nomes:** em iterações anteriores existiram pastas `combo-caller/` e `treino/` que eram versões antigas renomeadas de `shadowbox/` e `treino-musculacao/`. Elas **foram removidas**; se aparecerem de novo, são lixo e devem sair.

### 2.4 URLs de produção (após deploy)
Com GitHub Pages ligado no repo `mrbd-apps` do usuário **`mseixas`**, as URLs são:
- Launcher: `https://mseixas.github.io/mrbd-apps/`
- `https://mseixas.github.io/mrbd-apps/nivel-bolha/`
- `https://mseixas.github.io/mrbd-apps/bussola/`
- `https://mseixas.github.io/mrbd-apps/contador/`
- `https://mseixas.github.io/mrbd-apps/round-timer/`
- `https://mseixas.github.io/mrbd-apps/shadowbox/`
- `https://mseixas.github.io/mrbd-apps/shadowbox-mobile/`
- `https://mseixas.github.io/mrbd-apps/treino-musculacao/`

Cada app dos óculos é adicionado **individualmente pela sua URL** no app Meta AI (ver seção 7.3). A versão mobile é aberta no navegador do celular/tablet, não nos óculos.

---

## 3. Decisões técnicas já tomadas e o porquê

| # | Decisão | Motivo |
|---|---------|--------|
| D1 | **Single-file HTML/CSS/JS vanilla por app** | Zero build/deps; o runtime dos óculos é um browser simples; apps pequenos não justificam bundler; subir = copiar um arquivo. |
| D2 | **Repositório guarda-chuva único (`mrbd-apps`) com subpastas** | Um só GitHub Pages, uma só configuração; cada app tem URL limpa por subpasta; fácil de versionar junto. |
| D3 | **GitHub Pages como deploy** | Único requisito da plataforma é HTTPS; Pages entrega isso de graça; sem servidor para manter. (Ver aviso crítico no topo.) |
| D4 | **Viewport fixo 600×600, fundo preto, alto contraste** | Imposição do hardware: display é 600×600; é *aditivo* (preto puro = transparente, não emite luz); cores vivas são as mais legíveis sobre o mundo real. |
| D5 | **Navegação só por D-pad (setas + Enter) com classe `.focusable`** | A Neural Band e o touch da haste traduzem gestos em teclas de seta + Enter. Não há mouse/toque/cursor nos óculos. Cada interativo é `.focusable` e a gestão de foco é manual em JS. |
| D6 | **Permissões de sensor/GPS disparadas por gesto do usuário** | `DeviceOrientationEvent`/`DeviceMotionEvent`/geolocation exigem gesto (ex.: Enter num botão) para pedir permissão; nunca no load automático. |
| D7 | **Áudio (beep) via Web Audio API SEMPRE pareado com flash visual de tela** | A doc da Meta **não confirma** saída de áudio em Web Apps. Então todo sinal sonoro tem um flash de tela cheia como fallback garantido. Nunca dependa só do áudio. |
| D8 | **Ícones em PNG, nunca SVG (para ícone de app)** | Bug conhecido documentado pela Meta: SVG não funciona como ícone de app. (SVG *inline no corpo da página* funciona e é usado — ex.: a luva do shadowbox.) |
| D9 | **Tradução do Shadowbox por _tokens de golpe_, não por strings de combo inteiras** | Permite trocar idioma (PT/EN/ES) sem alterar a numeração dos combos nem os encadeamentos 2.0/3.0. Cada golpe é um token com 3 traduções; combos são montados a partir de tokens. (Ver 5.3.) |
| D10 | **Musculação foca em descanso + registro, NÃO em contagem de reps** | Os óculos ficam na cabeça; o acelerômetro não "vê" o movimento da barra/haltere. Contar reps de agachamento/supino não é viável. Contagem por movimento só serve para coisas que mexem a cabeça/corpo (polichinelo, corrida, pular corda). |
| D11 | **Detecção de execução de golpe = INVIÁVEL sem a Neural Band, e limitada mesmo com ela** | Os sensores só leem o movimento da **cabeça**, não das mãos/pernas. Dá para detectar esquivas/ducks (a cabeça se move), mas não socos/chutes. Prometer contagem de golpes seria enganar o usuário. |
| D12 | **Idioma padrão Português; Shadowbox com seletor PT/EN/ES na 1ª tela** | Usuário é brasileiro; Shadowbox tem uso potencial multilíngue. |

---

## 4. Convenções de código e padrões a seguir

Estas convenções são **consistentes em todos os apps**. Siga-as ao criar apps novos ou editar existentes — a consistência é o que mantém o portfólio coeso e previsível.

### 4.1 Metadata obrigatória no `<head>` (apps dos óculos)
```html
<meta name="description" content="...">
<meta name="mrbd-web-app-capable" content="yes">
<meta name="viewport" content="width=600, height=600, initial-scale=1.0, user-scalable=no">
```
E no CSS: `html, body { width:600px; height:600px; overflow:hidden; background:#000; color:#fff; }`.

### 4.2 Design
- **Fundo preto `#000`** (vira transparente no display). Texto claro de alto contraste.
- Cor de destaque padrão do projeto: **ciano `#00d4ff` / `#00e0ff`**. Verde `#2ecc71` = ativo/OK; âmbar `#ffb020` = descanso/atenção; vermelho `#e11d2a` = luva/acento.
- **Fontes grandes:** mínimo 16px; conteúdo principal 20–24px+; números de destaque bem grandes (até ~150px).
- Fonte: `-apple-system, system-ui, sans-serif`.

### 4.3 Navegação D-pad (padrão obrigatório)
- Todo elemento interativo tem a classe **`.focusable`** e **`min-height: 88px`** (alvo mínimo de toque dos óculos).
- Estado de foco visível: borda ciano + `box-shadow` glow.
- Um listener global de `keydown` trata `ArrowUp/Down/Left/Right` e `Enter`:
  - Up/Down (e geralmente Left/Right) movem o foco entre os `.focusable` visíveis da tela ativa, com wrap-around.
  - Enter dispara `.click()` no elemento focado.
  - Em telas de configuração, Left/Right ajustam valores (−/+) quando o foco está numa "linha" de config (atributo `data-key`).
- Padrão de seleção de foco:
  ```js
  var f = Array.from(document.querySelectorAll('.screen.active .focusable'))
    .filter(function(el){ return el.offsetParent !== null; });
  ```
  (só focáveis visíveis da tela ativa).

### 4.4 Multi-tela dentro de um app
- Telas são `<div class="screen">`, com `.active` marcando a visível (`display:flex`). Uma função `show(screen)` remove `.active` de todas e adiciona na alvo, depois foca o primeiro `.focusable`.

### 4.5 Sensores
- **Sempre** cheque disponibilidade e peça permissão via gesto:
  ```js
  if (typeof DeviceOrientationEvent !== 'undefined' &&
      typeof DeviceOrientationEvent.requestPermission === 'function') {
    DeviceOrientationEvent.requestPermission().then(function(s){ if (s==='granted') start(); });
  } else { start(); } // runtime dos óculos e Android geralmente concedem direto
  ```
- Faça throttle/`requestAnimationFrame` em updates de alta frequência; remova listeners quando sair da tela (bateria).
- `DeviceOrientationEvent`: `alpha` = heading/bússola (0–360), `beta` = tilt frente/trás, `gamma` = tilt esq/dir.
- `DeviceMotionEvent`: `accelerationIncludingGravity` (x,y,z em m/s²); magnitude em G = `sqrt(x²+y²+z²)/9.81`.
- `navigator.geolocation`: `getCurrentPosition` + `watchPosition`; use `timeout: 15000`; trate os 3 códigos de erro (1=permissão, 2=indisponível, 3=timeout). A localização vem do **celular pareado** (precisão 5–50 m).

### 4.6 Beep + flash (padrão de sinalização)
- Beep via `AudioContext` + oscilador (tipo `square`), criado/resumido **dentro de um gesto** (`ensureAudio()` chamado no primeiro `keydown`/click).
- **Todo** beep é acompanhado de um flash: um `<div id="flash">` posicionado sobre a tela, que ganha `.show` (opacidade) por ~300 ms. É o fallback garantido caso o áudio não saia nos óculos.

### 4.7 Storage
- `localStorage` para persistência (preferências, histórico, waypoint), com try/catch e `JSON.parse/stringify`. Chaves usadas hoje: `mrbd_waypoint`, `mrbd_repcount`, `mrbd_musc_hist`. Mantenha o prefixo `mrbd_`.

### 4.8 Estilo de JS
- Vanilla, ES5-ish defensivo (`var`, `function`), sem transpilação. Evite recursos que exijam build. Funciona direto no browser.
- **Validação obrigatória antes de entregar** (ver seção 7.5): balanceamento de `{} () []`, contagem `<script>`/`</script>`, e `node --check` na porção JS extraída.

### 4.9 Entrega de arquivos
- Cada app é entregue como arquivo pronto ou dentro de um **zip do repo** (`mrbd-apps.zip`) para upload manual. Sempre informe o **caminho sugerido no repo** junto com cada entregável (ex.: "vai em `mrbd-apps/round-timer/index.html`").
- O usuário costuma **pré-visualizar** no desktop cópias renomeadas (ex.: `A-round-timer.html`) antes de subir, porque vários apps se chamam `index.html`.

### 4.10 Restrições da plataforma que NÃO se deve violar
- **Sem entrada de texto** (não há teclado na API) — toda entrada é por seleção via D-pad/presets.
- Sem câmera, sem microfone (ainda), sem offline, sem notificações via Web Apps API.
- SVG **não** serve como ícone de app (use PNG ≥52×52). SVG inline no corpo, ok.

---

## 5. O que já foi feito (estado atual dos apps)

Todos os apps abaixo estão **construídos, validados (JS aprovado no `node --check`) e prontos**. O que falta é o **deploy** (subir ao GitHub + ligar Pages) — ver seções 6 e 7.

### 5.1 Utilitários de sensor
- **`nivel-bolha/`** — Nível de bolha/inclinômetro. Bolha se move numa mira conforme `beta`/`gamma`; fica verde e mostra "NIVELADO" quando reto (±1°). Botão de calibrar (zera o offset) e resetar. **É o app-chave de teste**: isola o sensor de orientação. O resultado dele (quão responsivo/preciso é o giroscópio na prática) decide se vale construir apps mais ousados que dependem de orientação em tempo real (ex.: "modo esquiva", jogo de labirinto).
- **`bussola/` ("Volta pra Casa")** — Marca a posição atual com um preset (Carro/Casa/Ponto), depois mostra uma seta grande apontando de volta ao ponto + distância em tempo real. Usa `getCurrentPosition` + `watchPosition`, heading da bússola (`alpha`), haversine para distância, bearing para direção, e `localStorage` (`mrbd_waypoint`) para lembrar o último ponto.
- **`contador/`** — Conta "repetições" por picos de aceleração da cabeça/corpo (`DeviceMotionEvent`). Troca de exercício (limiares diferentes por exercício), salva histórico por dia (`mrbd_repcount`). **Limitação documentada:** só funciona para movimentos que mexem a cabeça/corpo (polichinelo, corrida, pular corda), **não** para musculação com peso.

### 5.2 Fitness / artes marciais
- **`round-timer/`** — Timer de rounds estilo boxe/MMA. Configura nº de rounds, tempo de luta, descanso e preparação inicial (◀▶ ajusta, ▲▼ navega). Relógio gigante, fase atual (LUTA/DESCANSO), nº do round. Beep + flash na virada; bipes de aviso nos últimos 3 s.
- **`treino-musculacao/`** — Monta o treino (marca exercícios via D-pad, ajusta séries de cada, define descanso padrão). Na sessão, cada "✓ Série feita" dispara o countdown de descanso com beep no fim; navega entre exercícios; resumo final salvo em `localStorage` (`mrbd_musc_hist`). **Foca em descanso + registro** (ver D10). Listas de exercícios editáveis no array `EXERCISES` no topo do script.

### 5.3 Shadowbox — o app mais complexo (leia com atenção)
`shadowbox/` é o app mais desenvolvido e o que mais recebeu iterações. Funcionalidades:

- **Seleção de idioma (1ª tela): PT / EN / ES.** Traduz **toda a interface E os nomes dos golpes**.
- **Modos:** 🎲 **Random** (sorteia combos, sem repetir o mesmo seguido), 🔢 **Sequência** (combos em ordem), 📚 **Aprender** (cards de estudo, sem tempo/cronômetro).
- **Níveis:**
  - **1.0** — os 18 combos individuais.
  - **2.0** — 9 encadeamentos de 2 combos: `[2,14],[3,13],[4,7],[5,9],[6,10],[7,15],[8,16],[9,12],[10,11]`.
  - **3.0** — 9 encadeamentos de 3 combos: `[2,14,12],[3,13,11],[4,7,8],[5,9,17],[6,10,18],[7,15,17],[8,16,14],[9,12,17],[10,11,18]`.
- **Random/Sequência** entram numa tela de config (rounds, tempo de execução, descanso, intervalo entre combos) e rodam com timer + beep/flash. **Aprender** pula config e vai direto para os cards.
- **Destaque do NÚMERO do combo:** o objetivo pedido pelo usuário é *memorizar os combos pelo número*. Então o número aparece em "chips" grandes e brilhantes (150px para 1 combo, 104px para pares, 78px para trios), e o texto dos golpes vem menor embaixo, com a fonte escalando conforme a quantidade de combos encadeados para caber nos 600px.
- **Cards (modo Aprender):** navega com ◀▶ (que é como o swipe lateral chega nos óculos) e com swipe de dedo/mouse; ▲▼ move o foco entre card e botões; pontinhos indicam a posição (ex.: 4/18). **Não há** texto "arraste" no card (foi removido a pedido).
- **Ícone de luva de boxe vermelha** em SVG inline no cabeçalho.

**Arquitetura de tradução (importante — D9):**
```js
var TOKENS = { 'Jab': {pt:'Jab', en:'Jab', es:'Jab'}, 'Cross': {pt:'Direto', en:'Cross', es:'Directo'}, ... };
var COMBO_TOKENS = [ ['Jab'], ['Jab','Cross'], ... ];   // 18 combos como arrays de tokens
var CHAINS_2 = [...]; var CHAINS_3 = [...];              // encadeamentos por número (1-based)
```
Cada combo é montado juntando os tokens traduzidos. **Trocar/adicionar um golpe = editar o dicionário `TOKENS`;** a numeração e os encadeamentos não mudam. Os arrays ficam no topo do `<script>`, comentados para facilitar edição.

**Os 18 combos (referência canônica, em inglês/base):**
```
1  Jab
2  Jab + Cross
3  Cross + Hook + Cross
4  Jab + Cross + Hook + Cross
5  Upper + Hook + Cross + Upper
6  Upper + Cross + Hook + Upper
7  Duck L + Hook + Cross + Hook
8  Duck R + Cross + Hook + Cross
9  Esq L + Jab + Cross            (Esq = Slip)
10 Esq R + Cross + Jab
11 Block L + Duck + Hook + Hook + Cross + Hook
12 Block R + Duck + Hook + Upper + Hook + Cross
13 Block Kick L + Hook + Cross + Kick R
14 Block Kick R + Cross + Hook + Switch + Kick L
15 5 Elbows + 3 Knees
16 5 Elbows + 3 Knees
17 Front 1 + Round 2 + Round Side Full
18 Front 2 + Switch + Round 2 + Round Side
```

- **`shadowbox-mobile/`** — Versão **responsiva touch** do mesmo app, arquivo separado, para celular e tablet. Diferenças: `viewport width=device-width`, layout fluido com `clamp()`/`vw`, `100dvh`, respeito a `safe-area-inset` (notch), botões grandes tocáveis, **steppers ◀▶ tocáveis** na config (no lugar do D-pad), swipe nativo nos cards. Mesma lógica, mesmos `TOKENS`/combos, mesmas traduções.

### 5.4 Launcher (`index.html` da raiz)
Página inicial 600×600 (dos óculos) que lista os apps com links e navegação D-pad. Útil para o usuário pegar as URLs. Não é um "app" em si; é o índice. Não inclui o `shadowbox-mobile` (que é para celular, não óculos).

### 5.5 Documentação da API
Existe um arquivo de referência consolidado de toda a documentação oficial da Web App API da Meta (ver seção 11) — deve acompanhar o projeto.

---

## 6. O que está em andamento e próximos passos

### 6.1 Bloqueio imediato: DEPLOY
Os 7 apps + launcher estão prontos, mas **ainda não confirmadamente publicados**. O próximo passo concreto é: subir a estrutura ao repo `mrbd-apps` (público) e **ligar o GitHub Pages**. Ver seção 7 para o passo a passo. Este é o item nº 1.

### 6.2 Validação no hardware real (crítico, ainda pendente)
Vários pontos só se resolvem testando nos óculos de verdade. Prioridade:
1. **Áudio nos óculos** — o beep (Web Audio) funciona no desktop, mas a doc da Meta não confirma áudio nos óculos. **Precisa testar.** O resultado influencia todos os timers. (O flash visual já cobre o pior caso.)
2. **Nível de bolha** — mede na prática a **responsividade/precisão do sensor de orientação**. É o gate que decide se vale investir em apps ousados baseados em orientação em tempo real (modo esquiva, jogo de labirinto de inclinação).
3. **Swipe → seta** no Shadowbox mobile/óculos — confirmar que o swipe lateral realmente chega como `ArrowLeft/Right` no runtime dos óculos (a doc indica que sim, mas confirmar). Se vier como outro evento, ajustar o mapeamento.

### 6.3 Ideias já propostas e priorizadas (backlog)
Do mais seguro/útil ao mais ousado:
- **Modo Esquiva** (para o Shadowbox ou app separado): usa orientação (tilt/roll da cabeça) para chamar e *validar* slips/ducks — o único uso de sensor viável para luta sem a Neural Band. **Depende** do resultado do teste do nível de bolha.
- **EMOM/Tabata timer** (complementa o Round Timer com outros formatos de intervalo). Seguro.
- **Respira** (guia de respiração/box breathing, animação + streak em storage). Seguro, bonito no display aditivo.
- **Timer de cozinha/Pomodoro multi-timer.** Seguro.
- **Bússola+altímetro**, **inclinômetro de trilha/offroad** (variações "instrumento" dos apps de sensor). Seguros.
- **Labirinto de inclinação** (jogo, orientação em game loop) e **Reflexo/Reação** (tempo de reação) — bons para demonstração; o labirinto depende do teste de orientação.

### 6.4 Recursos bloqueados por hardware/plataforma (não prometer)
- **Cronômetro por voz** — foi um teste anterior que funcionou como cronômetro, mas depende de **microfone**, ainda não liberado na Web Apps API. Em espera até a Meta liberar.
- **Detecção de socos/chutes** — inviável (sensores só leem a cabeça). Ver D11.
- **Contagem de reps de musculação** — inviável pelo mesmo motivo. Ver D10.

### 6.5 Caminho alternativo de plataforma (explorado, em espera)
Foi investigado o **Device Access Toolkit (DAT)** da Meta, que habilitaria câmera, microfone, LED, Bluetooth e detecção de objetos por IA (o interesse concreto era **detecção de javali/caça** por câmera). **Requer aprovação manual da Meta** em `developers.meta.com/wearables`. Está **em espera**, pendente de decisão de aplicar. Isto é um SDK/fluxo diferente do Web Apps atual — não confundir com o portfólio estático.

---

## 7. Deploy, armadilhas conhecidas e restrições

### 7.1 Plataforma de deploy: GitHub Pages (reafirmando o aviso do topo)
- Repo **público** `mrbd-apps` do usuário `mseixas`. Conta gratuita do GitHub Pages exige repo **público** para publicar (privado só com Pages pago).
- Não há build. Pages serve os arquivos como estão.

### 7.2 Ligar o GitHub Pages (o usuário faz isso manualmente)
No repo: **Settings → Pages → Source: "Deploy from a branch" → branch `main`, pasta `/ (root)` → Save.** Em ~1 minuto as URLs (seção 2.4) ficam no ar. **Mexer nessa configuração fica com o dono do projeto** (é config de conta); o assistente guia, não executa.

### 7.3 Adicionar um app nos óculos
App Meta AI (com **Developer Mode** ligado: Settings → App Info → tocar 5× na versão) → **App Connections → Web Apps → Add a Web App** → colar a URL HTTPS. Cada app entra por sua própria URL.

### 7.4 ARMADILHA: automação de navegador (Claude in Chrome) é instável aqui
Houve tentativas de usar a extensão **Claude in Chrome** para criar o repo e subir arquivos automaticamente. **Foi consistentemente problemático:** timeouts longos (4 min), quedas de conexão no meio de ações, e diálogos de permissão negados. **Recomendação firme:** para poucos arquivos, o **upload manual é mais rápido e confiável** — gerar `mrbd-apps.zip`, o usuário descompacta e faz **Add file → Upload files** (drag-and-drop preserva a estrutura de pastas) no GitHub, ou `git push`. Não insista na automação de navegador para deploy pontual.

### 7.5 ARMADILHA: ambiente de execução reseta entre tarefas
O sandbox de código **reinicia o filesystem entre tarefas** — arquivos em `/home/claude` não persistem de uma sessão para outra. Consequências práticas:
- Não confie em arquivos gerados numa sessão anterior ainda estarem no disco; **regenere a partir do conteúdo conhecido** quando necessário.
- Sempre valide os arquivos na sessão atual antes de entregar.

### 7.6 ARMADILHA: `str_replace` e editor do GitHub com auto-fechamento
- Ao editar arquivos, lembre que uma `str_replace` bem-sucedida invalida views anteriores do mesmo arquivo (re-veja antes de nova edição).
- Se algum dia tentar colar código no **editor web do GitHub via digitação simulada**, cuidado: editores tipo CodeMirror fazem **auto-fechamento de tags/brackets**, o que corrompe código colado tecla a tecla. Por isso o método preferido de deploy é **upload de arquivo** (não digitar no editor).

### 7.7 Rotina de validação obrigatória antes de entregar qualquer app
Para cada `index.html`, rode as checagens:
- Balanceamento de `{ } ( ) [ ]`.
- `<script>` count == `</script>` count.
- Presença de `mrbd-web-app-capable` e `width=600` (nos apps dos óculos).
- Extraia a porção JS e rode **`node --check`** (pega erros de sintaxe reais). Padrão usado:
  ```python
  import re; s=open(path,encoding='utf-8').read()
  open('/tmp/x.js','w').write(re.search(r'<script>(.*)</script>', s, re.S).group(1))
  ```
  ```bash
  node --check /tmp/x.js
  ```
  > Obs.: o shell padrão do sandbox é `dash` (`/bin/sh`), que **não** suporta process substitution `<(...)`. Use arquivo temporário como acima.

### 7.8 Restrições de plataforma (resumo consolidado)
- Viewport **600×600 fixo**, sem scroll. Fundo preto (aditivo). Cores vivas/alto contraste. Fonte ≥16px.
- **Só D-pad** (setas + Enter); sem mouse/toque/cursor/teclado nos óculos. `.focusable` + `min-height:88px`.
- **HTTPS obrigatório** (GitHub Pages atende).
- Permissões de sensor/GPS **só via gesto**. GPS vem do celular pareado (5–50 m).
- **Sem** entrada de texto, câmera, microfone, offline, notificações (na Web Apps API atual).
- Ícone de app **PNG** (≥52×52), **não SVG**.
- Áudio **não confirmado** nos óculos → sempre pareie beep com flash visual.

---

## 8. Glossário rápido
- **MRBD** — Meta Ray-Ban Display (os óculos com display).
- **Neural Band** — pulseira de entrada por gestos (opcional); traduz gestos em teclas. O usuário pode treinar **sem** ela (para não danificá-la), então apps não devem *depender* dela.
- **D-pad** — as setas + Enter que os gestos/touch produzem.
- **Display aditivo** — o preto puro não emite luz, logo fica transparente; por isso fundo preto + cores vivas.
- **Combo / Encadeamento** — no Shadowbox, "combo" é uma sequência de golpes numerada (1–18); "encadeamento" (2.0/3.0) junta 2 ou 3 combos.
- **DAT** — Device Access Toolkit (SDK avançado da Meta, requer aprovação; em espera).

---

## 9. Como continuar a partir daqui (checklist para o próximo assistente)
1. **Confirme a plataforma de deploy** com o dono: GitHub Pages (padrão) — só mude para Render se ele pedir explicitamente.
2. **Priorize o deploy** dos 7 apps + launcher (upload manual do zip → ligar Pages). Não use automação de navegador para isso.
3. Depois do deploy, **oriente o teste no hardware**, na ordem: nível de bolha (sensor), beep nos óculos, swipe→seta no shadowbox.
4. Ao criar app novo, **siga a seção 4 inteira** (metadata, 600×600, `.focusable`, D-pad, beep+flash, storage `mrbd_`).
5. **Valide sempre** (seção 7.7) antes de entregar; regenere arquivos do zero se o filesystem tiver resetado (seção 7.5).
6. **Mantenha a honestidade técnica** (seção 1.4 / D10 / D11): não prometa detecção de socos, contagem de reps de peso, nem áudio garantido.
7. Entregue como **zip do repo** com caminhos sugeridos, e ofereça previews renomeados para o desktop.

---

## 10. Contatos / identificadores do projeto
- **GitHub:** usuário `mseixas`, repositório `mrbd-apps` (público).
- **Plataforma:** Meta Ray-Ban Display, via Meta Wearables Web App API.
- **App companheiro:** Meta AI app (Developer Mode) para adicionar/testar Web Apps.
- **Docs oficiais:** `https://wearables.developer.meta.com/docs/develop/webapps/`
- **Repo de exemplo da Meta:** `github.com/facebookincubator/meta-wearables-webapp` (BSD; contém o exemplo Snake e um plugin de skills para assistentes de código).
- **MCP de docs da Meta (opcional no ambiente de dev):** `https://mcp.developer.meta.com/wearables` (ferramenta `search_webapps_docs`) — busca semântica na doc viva; plugável no Claude Code.

---

## 11. Anexo que deve acompanhar este documento
- **`meta-rbd-webapps-referencia.md`** — referência consolidada de toda a API (input/D-pad, sensores, geolocalização, storage, ícones, setup, teste, hospedagem, checklist), com exemplos de código. Mantenha na raiz do repo junto deste HANDOFF/CLAUDE.md. Se ele não estiver presente, peça ao dono do projeto ou reconstrua a partir de `https://wearables.developer.meta.com/docs/develop/webapps/`.

---
*Fim do documento de transição. Se algo aqui conflitar com instruções diretas do dono do projeto, as instruções dele prevalecem — mas confirme especialmente qualquer mudança de plataforma de deploy (GitHub Pages ↔ Render) antes de agir.*
