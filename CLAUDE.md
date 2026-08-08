# CLAUDE.md — Portfólio de Web Apps para Meta Ray-Ban Display

**Leia primeiro o [HANDOFF.md](HANDOFF.md)** — é o documento de transição completo do projeto (objetivo, decisões técnicas, convenções de código, armadilhas). Este arquivo só registra o que **mudou** em relação ao que o HANDOFF descreve.

## Correções ao HANDOFF (estado real em 2026-07-05)

1. **O repositório é `mseixas/rbmd`, não `mrbd-apps`.** O deploy já foi feito: GitHub Pages está **ligado e servindo** em `https://mseixas.github.io/rbmd/<pasta>/`. O "bloqueio imediato de deploy" (HANDOFF §6.1) está resolvido.

2. **Nomes de pastas diferem do HANDOFF:**
   | HANDOFF | Real |
   |---|---|
   | `nivel-bolha/` | `nivel/` |
   | `bussola/` | `goback/` |
   | `contador/` | `counter/` |
   | demais | iguais |

3. **Dois apps novos que o HANDOFF não conhece** (contexto: usuário é caçador licenciado / tiro esportivo):
   - `chrono/` — **Shot Timer IPSC**: countdown com delay aleatório, beep de largada, TPD, splits, histórico (`localStorage` chave `ipsc_sessions`). Tema laranja `#ff3c00` em vez do ciano padrão.
   - `mira/` — **Red Dot**: retículo em canvas (8 tipos, vermelho/verde), reposicionável por D-pad sobre fundo preto (transparente no display).

4. **O launcher da raiz (`index.html`) não existe** — por isso `https://mseixas.github.io/rbmd/` retorna 404. É o próximo entregável óbvio (HANDOFF §5.4 descreve como deve ser; incluir também `chrono` e `mira`; não incluir `shadowbox-mobile`, que é para celular).

5. **`meta-rbd-webapps-referencia.md` não está no repo** (HANDOFF §11). Se precisar da referência da API, pedir ao dono ou reconstruir de `https://wearables.developer.meta.com/docs/develop/webapps/`.

## Apps de WebXR — NÃO seguem as convenções dos óculos

Duas pastas são **Meta Quest 3**, não Ray-Ban Display. Elas quebram de propósito
quase todas as regras da seção seguinte (viewport, D-pad, 600×600, `var`):

- **`shadowbox-xr/`** — Shadowbox em realidade mista, three.js r170 embutido.
- **`pista-xr/`** — **Simulador de treino seco de Tiro Defensivo da LNTD**.
  Leia [pista-xr/README.md](pista-xr/README.md). É o app mais complexo do repo.

### O que vale em `pista-xr/`

- **Arquivo único** `index.html` (~1,3 MB), com three.js r170 embutido como
  `<script id="three-src" type="text/plain">` + `import()` de Blob. Sem CDN, sem
  build no cliente, sem rede depois de carregado. `viewport width=device-width`,
  ES moderno (`const`/`let`, `async`), não ES5.
- **Fonte da verdade das regras:** Regulamento da LNTD 2025 (Revisão final) e
  Regulamento dos Campeonatos Brasileiros On-line 2026. **Toda regra
  implementada carrega o artigo de origem em comentário.**
- **NÃO EXISTE "pontos" no Tiro Defensivo — tudo é SEGUNDOS** (art. 3.2/3.3). O
  `build.js` tem uma checagem que falha se aparecer um campo
  `pontos`/`pontuacao`/`score`.
- **A Ficha de Pista chega como PNG todo mês.** O importador (`Croqui`) detecta
  os elementos por cor/forma e calibra **por eixo separado** — o croqui da Liga
  é anisotrópico (~108 px/m horizontal contra ~92 vertical). Sem OCR embutido.
- **Nada é inferido em silêncio:** todo campo derivado carrega
  `estado: 'lido' | 'inferido' | 'manual'`, e a atribuição Alvo↔Posto nunca é
  inventada — quem manda é o texto de PROCEDIMENTOS (art. 3.8.4).
- **Os polígonos laterais das Zonas do Alvo não estão confirmados** no Anexo I
  renderizado. Enquanto o usuário não medir e confirmar, o app mostra a zona de
  cada impacto mas **não soma os segundos de zona**, com banner amarelo.

### Como editar

O arquivo é montado a partir de 17 partes por um `build.js`. As partes e o script
**não estão no repositório** (ficam no scratchpad da sessão que o construiu). Para
mudanças pequenas, edite `pista-xr/index.html` direto e valide com
`node --check` no módulo extraído. Para mudanças grandes, reconstrua as partes a
partir do próprio arquivo (os cabeçalhos `/* ===== NN-nome.js ===== */` marcam os
cortes) — e mantenha a suíte embutida verde: `PISTAXR.testes.rodar()` no console,
ou o botão **RODAR TESTES** na tela inicial.

## Regras que continuam valendo para os apps dos ÓCULOS (resumo — detalhes no HANDOFF §4 e §7)

- Apps dos óculos: single-file `index.html`, vanilla JS (ES5-ish, `var`), 600×600, fundo `#000`, meta `mrbd-web-app-capable`, navegação D-pad via `.focusable` (min-height 88px), beep **sempre** pareado com flash visual, storage com prefixo `mrbd_`, permissões de sensor só via gesto.
- Deploy: GitHub Pages (o usuário sobe via upload manual no GitHub; **não** usar automação de navegador para deploy).
- Validar antes de entregar: balanceamento de chaves, `node --check` no JS extraído.
- Honestidade técnica (HANDOFF §1.4, D10, D11): não prometer detecção de socos, contagem de reps de musculação, nem áudio garantido nos óculos.
