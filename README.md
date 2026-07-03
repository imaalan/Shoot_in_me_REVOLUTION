# World Championship 2026 — Shoot Em In Reloaded

> Releitura em Three.js do classico *Shoot Em In*, readequada para a tematica da FIFA World Cup 2026. Penalty e falta, torneio completo com 48 selecoes, fisica 3D com efeito Magnus, estilo toon-shader com HUD grafite, deploy como PWA.

<p align="center">
  <img alt="Three.js" src="https://img.shields.io/badge/Three.js-r128-049EF4?logo=threedotjs&logoColor=white">
  <img alt="GSAP" src="https://img.shields.io/badge/GSAP-3.12-88CE02">
  <img alt="JavaScript" src="https://img.shields.io/badge/JavaScript-ES2020-F7DF1E?logo=javascript&logoColor=black">
  <img alt="PWA" src="https://img.shields.io/badge/PWA-Ready-5A0FC8">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-007D48">
  <img alt="World Cup 2026" src="https://img.shields.io/badge/Copa%202026-FF5000">
</p>

---

## Funcionalidades

### Gameplay
- **48 selecoes** com jogadores ficticios e atributos unicos (forca, precisao, curva)
- **Torneio completo**: Fase de Grupos (12 grupos de 4) → Oitavas → Quartas → Semi → Final
- **5 cobrancas por partida** — penalty ou falta (40% chance), aleatorio
- **Gol de ouro** na prorrogacao (6a cobranca, dificuldade maxima)
- **Cada erro = gol do adversário** — sem zero-a-zero

### Fisica 3D
- Gravidade, arrasto do ar, efeito Magnus (curva)
- **Vento** dinamico com indicador visual e particulas
- **Altitude** afeta velocidade e curva da bola (placa visual no estadio)
- **Umidade** aumenta arrasto (barra pisca acima de 70%)
- Colisao com goleiro, barreira, traves, travessao e chao

### IA
- **Goleiro preditivo** com dificuldade escalonada por fase do torneio (0.6x grupos → 1.5x final)
- **Barreira** de 2-4 jogadores com pulo aleatorio sincronizado
- **Torcida** 2D em sprites que pulam no gol e mudam cor pela selecao

### Controles
- **Mouse**: clique e arraste para mirar e definir forca
- **Teclado**: setas para mirar, espaco para chutar
- **Touch**: toque e arraste (mobile-first)

### Audio
- **TTS narracao** multilingue (PT/EN) via SpeechSynthesis: "GOL!", "DEFENDEU!", "NA TRAVE!"
- **Efeitos sonoros procedurais** via Web Audio API: chute, gol, defesa, trave, barreira, orbe, torcida, apito

### Mecanicas Extras
- **Cansaco** reduz forca a cada chute; stamina abaixo de 40% alerta no HUD
- **Orbe de energia** em posicao desafiadora — recupera 25% stamina
- **Replay** em camera lenta apos cada chute
- **Comemoracao** do jogador (pulo + bracos abertos) e da torcida

### UI/UX
- **HUD estilo grafite**: Permanent Marker, cores neon, painéis tortos (skew)
- **Placar estilo transmissao** de TV com bandeiras
- **Dia/noite dinamico** com floodlights
- **Responsivo**: desktop, tablet, mobile
- **Localization**: PT/EN com seletor de idioma

### Infraestrutura
- **PWA** com Service Worker e Web App Manifest (instalavel no mobile)
- **Persistencia** via IndexedDB (save/load automatico)
- **Acessibilidade**: modos daltonismo (protanopia/deuteranopia/tritanopia), alto contraste, labels ARIA, atalho Alt+A
- **Vite build** com PWA plugin e Workbox caching

---

## Como Jogar

| Acao | Controle |
|------|----------|
| Mirar | **Arraste para tras** com o mouse, ou use **setas** ← → ↑ ↓ |
| Chutar | **Solte** o mouse, ou pressione **ESPACO** |
| Coletar orbe | Acerte o **orbe dourado** flutuante para recuperar energia |

### Mecanicas importantes
- O **vento** desvia a bola — observe a seta no HUD
- **Altitude alta** = bola mais rapida, menos curva
- **Umidade alta** = mais arrasto, bola mais lenta
- O **goleiro** fica mais rapido nas fases finais
- **Cansaco** acumula — gerencie seus chutes

---

## Como Rodar

### Opcao 1: Abrir direto (zero setup)

```
world-championship-2026.html
```

Double-click para abrir no navegador. Funciona 100% offline.

### Opcao 2: Vite dev server (recomendado para dev)

```bash
git clone https://github.com/imaalan/Shoot_in_me_REVOLUTION.git
cd Shoot_in_me_REVOLUTION
npm install
npm run dev
```

Acesse `http://localhost:5173`. HMR ativo, PWA funciona via HTTP.

### Build de Producao

```bash
npm run build    # gera dist/ com PWA completa
npm run preview  # preview do build
```

### Deploy no GitHub Pages

Ativar em: `Settings → Pages → Source: Deploy from a branch → main / (root)`

Ou via GitHub Actions (ja configurado no `vite.config.js`):

```yaml
# Push na main → build automatico → deploy
```

---

## Stack Tecnica

| Camada | Tecnologia | Justificativa |
|--------|-----------|---------------|
| Engine 3D | Three.js r128 | Padrão WebGL, leve, comunidade vasta |
| Animacao | GSAP 3.12 | Tweens suaves, performático |
| Física | Custom (Euler) | Controle total sobre Magnus, altitude, umidade |
| Áudio | Web Audio API + TTS | Procedural, sem assets externos |
| Persistencia | IndexedDB | Async, ~50MB+ (LocalStorage insuficiente) |
| Build | Vite 5.x | HMR rapido, PWA plugin |
| Linguagem | Vanilla JS (ES2020+) | Sem framework UI, bundle minimo |
| CSS | CSS3 puro | Variáveis, media queries, keyframes |
| Fontes | Permanent Marker, Black Ops One, Roboto Condensed | Estilo grafite + broadcast |

---

## Estrutura do Projeto

```
Shoot_in_me_REVOLUTION/
├── world-championship-2026.html   # Jogo completo (single-file ~3000 linhas)
├── SDD-World-Championship-2026.md # Software Design Document
├── TDD-World-Championship-2026.md # Technical Design Document
├── package.json                   # Dependencias (Vite, Vitest)
├── vite.config.js                 # Config Vite + PWA plugin
├── public/
│   ├── icon-192.svg              # Icone PWA 192x192
│   └── icon-512.svg              # Icone PWA 512x512
├── LICENSE                        # MIT
├── README.md
└── .gitignore
```

---

## Arquitetura

```
┌─────────────────────────────────────────────────┐
│              Aplicacao (Browser)                  │
│                                                   │
│  Presentation ──► Application ──► Domain          │
│  (DOM + HUD)     (GameState FSM)  (Tournament)   │
│  (Screen Mgr)    (Input Ctrl)     (Match Engine)  │
│                  (Audio Engine)   (Physics + AI)   │
│                                                   │
│  Infrastructure: Three.js Renderer + IndexedDB    │
└─────────────────────────────────────────────────┘
```

**Padroes de projeto**: State Machine (FSM), Observer, Factory, Strategy, Singleton, Command, Template Method.

**Decisões de design**: Single-file para prototipo (zero build), fisica custom (penalty e caso simples), Three.js (leve para PWA), IndexedDB (capacidade superior a LocalStorage), TTS (sem assets de audio externos).

---

## Roadmap

- [x] Torneio completo (48 selecoes, grupos + mata-mata)
- [x] Fisica 3D (Magnus, vento, altitude, umidade)
- [x] IA preditiva (goleiro + barreira)
- [x] Procedural SFX (Web Audio API)
- [x] Acessibilidade (daltonismo, ARIA)
- [x] PWA (Service Worker, IndexedDB)
- [x] Vite build setup
- [ ] Unit tests (Vitest)
- [ ] Analytics / Error tracking
- [ ] Modo multiplayer online (futuro)
- [ ] Assets de audio customizados (futuro)

---

## Licenca

Distribuido sob a licenca **MIT**. Veja [`LICENSE`](LICENSE) para detalhes.

## Autor

**Alan de Jesus Lima** — desenvolvimento, design e arquitetura.

> Projeto de portfólio. Contribuicoes, issues e pull requests sao bem-vindos.
