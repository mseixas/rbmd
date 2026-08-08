# Pista XR — Tiro Defensivo LNTD no Meta Quest 3

Simulador de **treino seco** de Tiro Defensivo da Liga Nacional de Tiro Defensivo
(LNTD) para **Meta Quest 3 / 3S**, em realidade mista (passthrough).

Carrega a Ficha de Pista do mês, monta o estágio em 3D **em tamanho real ou
escalado por similaridade**, cobra a **ordem de engajamento** correta pelo
Regulamento e **apura em segundos** — zonas, tarja, miss, Alvo Amigo, Erros de
Procedimento e TPD.

> **Arquivo único.** `index.html` é autocontido: HTML + CSS + JS + three.js r170
> embutido. Sem CDN, sem build, sem servidor, sem internet depois de carregado.

---

## Enquadramento

Regulamento dos Campeonatos Brasileiros On-line 2026, **art. 4.3**: é proibido
ensaiar a Pista *executando disparos*; o **ensaio em seco com as mãos vazias é
expressamente autorizado**; e a Liga declara que não proíbe o Atleta de replicar
a Pista e treinar — a vedação recai sobre **Árbitros** promoverem treino da Pista
do Online.

Este software é ferramenta de **treino seco do Atleta**. O app exibe esse
enquadramento na primeira execução.

**Passthrough não resolve miras.** A precisão do tracking do controle é da ordem
de milímetros, mas a geometria da sua arma real não está aqui. O que este app
mede bem é **tempo, ordem, transição e disciplina de cano** — não pontaria fina.

**Nenhum dado sai do aparelho.** Diário em IndexedDB local, export manual.

---

## O modelo de pontuação — leia antes de usar

**Não existe "pontos" no Tiro Defensivo. Tudo é segundos.**

```
tempoTotal = tempoDePassagem            (do Bip ao ÚLTIMO estampido, art. 3.2)
           + segundosDeZona             (Z0 = 0 · Z2 = +2 · Z5 = +5, por impacto)
           + segundosDeMiss             (+10 por impacto faltante no Alvo)
           + segundosDeAlvoAmigo        (+10 por impacto, teto 20 s / 30 s)
           + segundosDePenalidade       (EP = +4 cada · Conduta Antidesportiva = +25)
           + segundosDeTPD              (ver abaixo)
```

Vence o **menor** tempo total.

### TPD — a regra mais contraintuitiva do Regulamento

TPD é o **maior tempo aceitável para o primeiro tiro** (art. 3.8.2). Nunca
inferior a 3 s.

| Primeiro disparo | Acréscimo |
|---|---|
| ≤ TPD | **zero** — o tempo nem é anotado |
| > TPD | **o tempo inteiro do primeiro disparo**, não a diferença |

Na pista de exemplo, TPD = 4 s para pistola. Primeiro tiro em **3,99 s custa 0**.
Em **4,01 s custa 4,01 s**. É um penhasco, não uma rampa — por isso a barra de
TPD é o elemento mais destacado do painel antes do primeiro tiro.

### A regra da linha

O impacto tem que estar **completamente dentro** de uma Zona para valer o valor
dela. Se o furo rompe a linha tracejada e qualquer parte cai na Zona exterior,
**vale a exterior** (art. 3.4.1.b). Por isso o teste é
**círculo-dentro-de-polígono** com o diâmetro nominal do calibre da sua Divisão —
não ponto-em-polígono. O calibre entra na conta.

### Cobertura mole e dura

- **Refém** (Amigo sobreposto ao Inimigo no mesmo suporte): cobertura **dura**.
  O tiro pontua +10 s no refém e **não** pontua no Inimigo atrás (art. 3.4.5.h).
- **Amigo solto**: cobertura **mole**. O projétil que o atravessa e atinge outro
  Alvo é computado em **todos** os Alvos envolvidos (art. 3.4.5.i).

Por isso o raycast **não para no primeiro alvo**.

---

## Zonas do Alvo — o que está confirmado e o que não está

Do **Anexo I** do Regulamento LNTD 2025, confirmados:

| | Padrão | Reduzido (2/3) |
|---|---|---|
| Largura do corpo | 450,00 mm | 300,00 mm |
| Altura total | 758,00 mm | 505,33 mm |
| Altura do corpo | 608,00 mm | 405,33 mm |
| Cabeça | 150 × 150 mm | 100 mm de largura |
| Faixa Z0 (do topo) | 480,00 mm | 320,00 mm |
| Faixa Z2 | 126,00 mm | 84,00 mm |
| Faixa Z5 | 152,00 mm | 101,33 mm |

`480 + 126 + 152 = 758` exatamente, e todo o reduzido é 2/3 do padrão — as duas
identidades são testes de sanidade da suíte.

**O que NÃO está confirmado:** a **largura lateral da Zona Zero** não é legível
na cota do Anexo I renderizado. Enquanto você não confirmar essa medida em
*Ajuste do alvo*, o app **mostra a zona de cada impacto mas NÃO soma os segundos
de zona**, e exibe banner amarelo. O valor hipotético aparece em cinza ao lado,
para você ver o que mudaria.

Meça a coluna central no alvo oficial, digite em *Ajuste do alvo* e marque
"confirmo" — aí a apuração fica completa.

---

## Ordem de engajamento

É a **distância entre os Alvos** que decide a doutrina, fora de Cobertura:

- **< 1 m entre si** → mesmo grau de ameaça → **Sequência Defensiva**
- **> 1 m entre si** → o mais próximo tem prioridade → **Prioridade Defensiva**

O app **infere** a doutrina da geometria e **compara com o que a Ficha declara**.
Divergência é sinal de erro de importação, e ele avisa.

**Prioridade Defensiva** (art. 3.8.4.a): do mais próximo ao mais afastado,
completando os impactos de um Alvo antes de passar ao próximo. Alvo com Fuzil ou
Tomada de Refém é **Prioritário** e engaja antes, independente da distância — mas
só **fora** de Cobertura.

**Sequência Defensiva** (art. 3.8.4.b): cada Alvo uma vez indo de uma extremidade
à outra, e cada um mais uma vez voltando. 2 Alvos → **1‑2‑1**; 3 Alvos →
**1‑1‑2‑1‑1**. É a única maneira correta. Pode iniciar por qualquer extremidade,
exceto se numa delas houver Fuzil ou Refém.

---

## Escala: os três modos

O app mede a área livre pela malha do cômodo (quando o Quest expõe
`plane-detection`/`mesh-detection`) ou usa o que você digitar.

| Modo | O que faz | Deslocamento entra no total? |
|---|---|---|
| **`real` (k = 1)** | Pista em tamanho real. Padrão quando couber. | **Sim** |
| **`angular` (k < 1)** | Similaridade uniforme: escala Postos, Alvos e o *tamanho* dos Alvos pelo mesmo fator. Preserva **todos os ângulos a partir de todos os pontos**, exatamente — mesmas transições em graus, mesmo overswing, mesmo tamanho angular. | **Não** — sai hachurado |
| **`estatico`** | Não anda; o app reposiciona a origem entre os Postos ao comando. | Não |

`k = min(profundidadeLivre / 8,00 ; larguraLivre / 2,79 ; 1)` para a pista de
exemplo. Com 8 × 3 m livres ela **cabe em tamanho real**. Com 3 × 4 m, k = 0,375.

O `k` fica sempre visível. O app nunca esconde que houve escala.

---

## Como usar

### 1. Hospedar

WebXR exige **HTTPS**. Três caminhos:

```bash
# a) GitHub Pages (é onde este repositório já publica)
#    https://mseixas.github.io/rbmd/pista-xr/

# b) Netlify Drop — arraste a pasta em https://app.netlify.com/drop

# c) Local, pelo cabo USB, com o Quest em modo desenvolvedor:
python -m http.server 8080
```

```bash
adb reverse tcp:8080 tcp:8080
```

Com o `adb reverse` ativo, abra `http://localhost:8080/pista-xr/` no navegador
do Quest — `localhost` é considerado origem segura.

### 2. Carregar a Ficha

**O caminho normal: o PNG que a Liga publica todo mês.**

Botão **Carregar croqui (PNG da Liga)** — solte o arquivo, arraste para a área,
ou cole com `Ctrl+V`. O app então:

1. **Acha os elementos** por cor e forma: os **suportes azuis** (a cor mais única
   da página, e por isso o marcador mais confiável de onde está cada Alvo), as
   **silhuetas de papelão**, as **barras pretas dos Postos**, a **tarja** e o
   **fuzil**. O que separa a barra de um Posto de uma linha de cota é o
   *preenchimento*: a barra é cheia, a cota é traço com serifas.
2. **Calibra a escala de cada eixo separadamente**, pela régua vertical de 2 m e
   pela cota horizontal. **O croqui da Liga não é isotrópico** — na Ficha "Sem o
   carregador!" a horizontal dá ~108 px/m e a vertical ~92 px/m, 15% de
   diferença. Escala única erraria a distância em 15%, e distância errada troca a
   doutrina, porque o limiar de 1 m do art. 3.4.5 é absoluto. Acima de 3% de
   divergência o app avisa.
3. **Mostra o que detectou por cima da própria imagem** — caixa ciano nos Alvos,
   verde nos Postos, âmbar na tarja, magenta nas cotas. Tracejado = palpite
   fraco.
4. **Deixa você corrigir**: arrastar, acrescentar, remover, e marcar as duas
   cotas na mão quando a leitura automática falhar.

Depois: nomeie os Alvos, preencha o cabeçalho (ou cole o texto da Ficha, se você
tiver o PDF — as regex extraem TIPO, ALVOS, IMPACTOS, TPD, RECARGA, TRAJE e os
engajamentos de PROCEDIMENTOS), e aplique.

Outros caminhos: **pista de exemplo** embutida, **arquivo .json** no schema
`pista/1`, **colar JSON**, e **gerar link** (comprime a pista com `deflate-raw` +
base64url no hash da URL — é o jeito prático de levar do PC para o Quest).

#### O que o importador NÃO faz

- **Não lê texto da imagem.** Não há OCR embutido: tesseract precisaria de
  vários megabytes e de CDN, o que quebraria o arquivo único — e você teria que
  revisar campo a campo do mesmo jeito. São 8 campos, mais rápido no dedo.
- **Não sabe os rótulos impressos.** Ele numera `A1…An` pela **posição**; a Liga
  numera por grupo de Posto (na Ficha de exemplo, A1–A3 são de P1 e A4–A5 de P2).
  Ajuste os nomes olhando o croqui. Se você colar o texto de PROCEDIMENTOS sobre
  uma numeração posicional errada, o app **pega o erro** pela divergência entre a
  doutrina declarada e a que a geometria exige (art. 3.4.5.a/b) e avisa.
- **Não inventa engajamento.** Sem o texto da Ficha ele *propõe* uma atribuição
  Alvo↔Posto pela geometria e marca como `inferido`, em amarelo. Quem manda é o
  que está escrito em PROCEDIMENTOS.
- **Não desenha as Linhas de Partida e de Falta.** Elas não são identificáveis
  por cor nem por forma no croqui; saem como inferência de 30 cm a 15 cm do
  Posto (Online 2026, art. 4.2.1), sinalizada.

#### Âncora do Alvo

O Alvo é desenhado como uma silhueta **sobre um suporte**, e as linhas de cota da
Ficha saem do **pé do suporte** — é ali que o Alvo toca o chão. É essa a âncora
padrão. Se no seu croqui a régua se alinhar com o meio da silhueta, troque em
*Posição do Alvo no croqui*: a diferença é de cerca de uma altura de Alvo
(0,76 m) na profundidade da Pista.

### 3. Conferir a validação cruzada

Antes de entrar, o app checa a Ficha contra o Regulamento:

- a doutrina inferida bate com a declarada? *(art. 3.4.5.a/b)*
- os Alvos do mesmo Posto têm vão ≥ 10 cm? *(Online 2026, art. 4.2.3)*
- Alvo reduzido convive com padrão no mesmo Posto? *(art. 3.4.5.c)*
- há mais de um Prioritário no mesmo Posto? *(art. 3.8.4.a.1)*
- em Sequência, o Prioritário está numa extremidade? *(art. 3.8.4.b.1)*
- há Sequência numa pista de 3 disparos? *(art. 3.8.4.b.4)*
- os Alvos em Sequência têm altura alternada? *(Online 2026, art. 4.2.2)*
- TPD ≥ 3 s? *(art. 3.8.2.a)*
- LP e LF ≤ 30 cm? *(Online 2026, art. 4.2.1)*

### 4. Calibrar o cano (opcional, mas é o que dá precisão)

O perfil sintético ("controle na mão") assume a boca 10 cm à frente do grip,
apontando para −Z. Serve para treinar ordem e tempo.

Para medir o **seu** cano, use o **pivô duplo**:

1. **Ponto A (boca do cano):** apoie a boca num ponto físico fixo e gire a arma
   em ≥ 6 orientações bem diferentes.
2. **Ponto B (referência traseira):** repita apoiando um segundo ponto no mesmo
   lugar.
3. O app resolve `R·p + t = c` por mínimos quadrados e monta o eixo:
   `origem = p_A`, `direção = normalize(p_A − p_B)`.

Resíduo acima de 5 mm → refaça. O perfil fica salvo por arma.

### 5. Correr

| Controle | Ação |
|---|---|
| **Gatilho** | disparo (`buttons[0].value > 0,6`, rearme abaixo de 0,2) |
| **A / X** | armar a corrida (delay aleatório + Bip) · em modo `estatico`, avançar de Posto |
| **B / Y** | encerrar a passagem |
| **Grip** | liga/desliga o traçado do cano |

Se ligar o **microfone**, o app detecta o estampido do seco por onset acústico
(passa-alta em 1,8 kHz, limiar `mediana + 8·MAD`, refratário de 120 ms) e
**compensa a latência** interpolando a pose no instante real do onset, via
`AudioContext.getOutputTimestamp()`. Sem essa compensação o "para onde o cano
apontava" erra 30–80 ms — meio metro no alvo, numa transição.

---

## O que o app mede além do cronômetro

| Métrica | Definição |
|---|---|
| `saque` | `t(disparo 1) − t0` — **é o que o TPD julga** |
| `split` | `t(i) − t(i−1)` no mesmo Alvo |
| `transição` | `t(primeiro em B) − t(último em A)` |
| `picoAngular` | `max \|ω\|` na transição, em °/s |
| `overswing` | maior ângulo além do centro de B, entre o pico de ω e o disparo |
| `assentamento` | tempo entre `\|ω\|` cair abaixo de 30 °/s e o disparo |
| `antecipação` | média do pitch 80 ms depois − 80 ms antes; negativo = mergulho |
| `deslocamento` | entre sair do raio de 0,4 m de um Posto e entrar no do próximo |

Mais: **replay** com rastro do cano colorido por velocidade angular e duas
corridas sobrepostas; **mapa de tempo** com a segmentação (saque → engajamento
P1 → deslocamento → engajamento P2 → final); **auditoria de segurança** (arco de
180°, cano muito alto, varredura do próprio corpo, dedo no gatilho fora da
janela, pé além da LF), com varredura de corpo e passo com cano alto marcados
como **DQ**, citando o artigo.

---

## Verificação

O app traz a suíte de aceite embutida. Na tela inicial, **RODAR TESTES**, ou no
console:

```js
PISTAXR.testes.rodar()
```

Cobre os critérios M1 a M8 do plano: identidades do Anexo I, similaridade com
erro < 1e‑9, o caso trabalhado da pista anexa (`A1,A1,A3,A3,A2,A2` em P1 e
`1‑2‑1` em P2), a regra da linha com furo de 9 mm a 3 mm da divisória, tarja,
refém, amigo solto, TPD em 3,99 s × 4,01 s, escala 8×3 × 3×4, pivô duplo com
σ = 2 mm, transição de 30°/250 ms com 4° de overswing, antecipação de −1,5°,
round-trip do replay e os eventos de segurança.

O harness completo também está exposto, para montar suas próprias trilhas:

```js
PISTAXR.sim.iniciarCorrida(pistaJson, cfg)
PISTAXR.sim.pose(tMs, {cabeca, cano, gatilho})
PISTAXR.sim.disparo(tMs)
PISTAXR.sim.resultado()
```

---

## Schema `pista/1`

```jsonc
{
  "schema": "pista/1",
  "meta": { "liga", "evento", "nome", "tipo", "autores": [], "obsCroqui" },
  "parametros": {
    "contagem": "ilimitada",
    "impactosPorAlvo": 2,
    "disparosNecessarios": 10,
    "tpd": { "pistola": 4, "outras": 5 },
    "recargaObrigatoria": false,
    "trajeOcultacao": "armas curtas"
  },
  "geometria": { "unidade": "m", "downrange": [0, 1], "origem": "P1" },
  "postos": [ { "id": "P1", "x": 0, "y": 0 } ],
  "linhas": [ { "id": "LP", "tipo": "partida", "centro": [0, -0.15], "largura": 0.30 } ],
  "alvos": [ {
    "id": "A1", "x": -0.56, "y": 8.0,
    "classe": "inimigo|amigo|refem|metal|prato",
    "tipo": "padrao|reduzido",
    "impactos": 2,
    "altura": "normal|baixo",          // "baixo" = 30 cm mais baixo (art. 4.2.2)
    "adornos": ["fuzil", "vestido", "cortado"],
    "ladoCabeca": "esquerda|direita",  // só para refém
    "tarja": {
      "poligono": [[u, v]],            // UV em mm, origem no canto inferior esquerdo
      "cobre": ["Z2", "Z5"],           // alternativa independente do modelo
      "retanguloEstimado": { "orientacao": "vertical", "lado": "direita", "largura": 0.14 }
    }
  } ],
  "engajamentos": [
    { "posto": "P1", "alvos": ["A1", "A2", "A3"], "doutrina": "prioridade", "cobertura": false }
  ],
  "procedimentos": "texto da Ficha"
}
```

Coordenadas em metros. `x` é lateral, `y` é o downrange (`downrange: [0,1]`
significa que o +y aponta para os Alvos). Origem em P1.

---

## Limitações — o que este app NÃO faz

Honestidade técnica é regra do projeto:

- **Não substitui o tiro real.** Mede tempo, ordem e disciplina de cano. Recuo,
  gatilho, empunhadura e pontaria fina ficam de fora.
- **Não conhece a geometria da sua arma** até você calibrar o pivô duplo. Sem
  calibrar, o eixo é uma aproximação da mão.
- **Os polígonos laterais das zonas não estão confirmados** no Anexo I
  renderizado. Enquanto isso, zona sai como diagnóstico, não como segundos.
- **A largura da tarja de A4** vem da obs do croqui ("cobrir todas as Zonas 2 e
  5"). Confira na Ficha oficial; o app deixa alternar para o retângulo estimado.
- **Pé além da Linha de Falta** é aproximado pela projeção da cabeça — confiança
  média, marcado como tal.
- **Cobertura** só é avaliada quando a Ficha traz o bloco `coberturas`; sem ele
  o app assume campo aberto.
- **Não sabe se você sanou uma pane ou remuniciou** — por isso cano alto sem
  deslocamento sai como aviso, não como DQ.
- **A detecção do croqui é heurística**, calibrada no gabarito da Ficha "Sem o
  carregador!". Se a Liga mudar as cores ou o traçado, os limiares em
  `Croqui.PADROES` precisam de ajuste — e a sobreposição sobre a imagem existe
  justamente para você ver isso na hora, em vez de descobrir dentro dos óculos.

---

## Correção na Ficha de exemplo

A Ficha de exemplo embutida traz **A3 em `x = +0,46 m`**, não em `−0,03` como
saiu da primeira derivação do croqui. Com `−0,03`, A3 fica quase colinear com P1
e A2 e **mascara A2 por inteiro**: engajar A2 de P1 seria geometricamente
impossível, e a Pista não teria como ser corrida. Relendo o croqui a 107,5 px/m
(a cota horizontal entre P1 e P2), A3 está à **direita** de P1:
`(465 − 415) / 107,5 = +0,465`. Todos os demais X batem com a derivação original.

O app hoje **recusa** esse tipo de Ficha: um novo verificador acusa quando um
Alvo fica inteiramente escondido atrás de outro do mesmo Posto, e avisa acima de
60% de mascaramento parcial. Não é regra do Regulamento — é sanidade geométrica,
e é o sinal mais forte de que as coordenadas derivadas do croqui saíram erradas.

A fonte definitiva continua sendo o PNG da Liga. Use **Carregar croqui**.

---

## Fontes

- Regulamento da Liga Nacional de Tiro Defensivo **2025** (Revisão final) —
  arts. 3.2, 3.3, 3.4, 3.8; Anexos I e II.
- Regulamento dos Campeonatos Brasileiros On-line de Tiro Defensivo **2026**
  (atualizado em 31 de julho) — arts. 4.1 a 4.3, 5.1.

Cada regra implementada carrega o artigo de origem em comentário no código.
