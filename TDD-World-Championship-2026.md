# Technical Design Document (TDD)
## World Championship 2026 — Shoot Em In Reloaded

**Versão:** 1.0  
**Data:** 2026-07-03  
**Autor:** Engenharia de Software  
**Status:** Aprovado para Prototipagem

---

## 1. Visão Geral

### 1.1 Propósito
Este documento define a arquitetura técnica, stack de tecnologia, decisões de design e especificações de implementação para o jogo *World Championship 2026 — Shoot Em In Reloaded*, uma releitura em Three.js do clássico *Shoot Em In* (Flash), readequada para a temática da FIFA World Cup 2026, com mecânicas de penalty e free kick, torneio completo de 48 seleções, física 3D, estilo toon-shader com HUD grafite e deploy como PWA.

### 1.2 Escopo
- Single-player vs IA (goleiro preditivo, barreira dinâmica)
- Torneio completo: Fase de Grupos (12 grupos de 4) → Oitavas → Quartas → Semi → Final
- 48 seleções com jogadores fictícios (caricaturas estilizadas)
- Mecânicas: vento, altitude, umidade, cansaço, orbe de energia
- Controles: Mouse (drag), Teclado (setas + espaço), Touch
- PWA responsivo para navegador (GitHub Pages)
- Persistência local via IndexedDB

### 1.3 Fora de Escopo (v1.0)
- Multiplayer online / PvP
- Backend servidor
- Assets de áudio customizados (são gerados via Web Audio API / TTS)
- Publicação nas lojas App Store / Play Store (PWA apenas)
- Modelos 3D high-poly (low-poly toon-shader)

---

## 2. Stack Tecnológica

| Camada | Tecnologia | Versão | Justificativa |
|--------|-----------|--------|---------------|
| **Engine 3D** | Three.js | r128 | Leve, ampla adoção, compatível com PWA, não requer build complexo |
| **Build Tool** | Vite (recomendado) | 5.x | HMR rápido, gera PWA com Vite PWA Plugin, otimiza bundle |
| **Física** | Custom (Euler integration) | — | Para colisões bola-barreira-gol, Cannon.js/Ammo.js seria overkill para escopo de penalty; física manual permite controle total sobre Magnus, arrasto, altitude |
| **Animação** | GSAP | 3.12.x | Animações de UI, câmera, personagens, tweens suaves, performático |
| **Áudio** | Web Audio API + SpeechSynthesis | Nativo | Narração TTS multilíngue (PT/EN), efeitos sonoros proceduralmente gerados (sem assets externos) |
| **Persistência** | IndexedDB (via API nativa) | — | Limite teórico ~50MB+, suficiente para estado de torneio completo; LocalStorage insuficiente |
| **PWA** | Service Worker (inline) + Web App Manifest | — | Instalação no mobile via Chrome/Safari, offline básico |
| **Linguagem** | Vanilla JavaScript (ES2020+) | — | Sem framework UI para minimizar bundle e complexidade; React Three Fiber descartado por overhead desnecessário para protótipo |
| **CSS** | CSS3 puro | — | Variáveis CSS, media queries, animações keyframes; sem pré-processador |
| **Fontes** | Google Fonts (Permanent Marker, Black Ops One, Roboto Condensed) | — | Estilo grafite + broadcast esportivo |

---

## 3. Arquitetura de Sistema

### 3.1 Diagrama de Componentes (C4 — Nível 1: Contexto)

```
┌─────────────────────────────────────────────────────────────┐
│                    Jogador (Browser)                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   Desktop   │  │   Mobile    │  │   Tablet            │ │
│  │  (Mouse+KB) │  │  (Touch)    │  │  (Touch/Mouse)      │ │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘ │
│         └─────────────────┴────────────────────┘            │
│                           │                                 │
│              ┌────────────┴────────────┐                   │
│              │    PWA (GitHub Pages)    │                   │
│              │  ┌─────────────────────┐  │                   │
│              │  │  World Championship │  │                   │
│              │  │     2026 — Game     │  │                   │
│              │  └─────────────────────┘  │                   │
│              └───────────────────────────┘                   │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Diagrama de Componentes (C4 — Nível 2: Containers)

```
┌────────────────────────────────────────────────────────────────────┐
│                        Single HTML File                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐ │
│  │   index.html  │  │  inline CSS  │  │   inline JS (modules)   │ │
│  │  (entrypoint) │  │  (UI/HUD)    │  │   (game logic + 3D)     │ │
│  └──────┬───────┘  └──────────────┘  └────────────┬─────────────┘ │
│         │                                          │              │
│         │         ┌─────────────────────────────────┘              │
│         │         │                                                │
│         ▼         ▼                                                │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                    CDN Dependencies                          │  │
│  │  Three.js (r128)  │  GSAP (3.12)  │  Google Fonts         │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                    Browser APIs                              │  │
│  │  WebGL  │  Web Audio  │  SpeechSynthesis  │  IndexedDB    │  │
│  │  Service Worker  │  Touch Events  │  Gamepad (future)     │  │
│  └─────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────┘
```

### 3.3 Diagrama de Componentes (C4 — Nível 3: Componentes Internos)

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Game Engine (JavaScript)                      │
│                                                                      │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────────────┐ │
│  │   Game State    │  │   3D Renderer   │  │   Input Manager    │ │
│  │   (FSM)         │  │   (Three.js)    │  │   (Mouse/Key/Touch)│ │
│  │                 │  │                 │  │                    │ │
│  │ • title         │  │ • Scene         │  │ • Mouse drag       │ │
│  │ • select        │  │ • Camera        │  │ • Keyboard arrows  │ │
│  │ • tournament    │  │ • Lights        │  │ • Touch swipe      │ │
│  │ • match         │  │ • Meshes        │  │ • Power mapping    │ │
│  │ • result        │  │ • Animations    │  │                    │ │
│  │ • champion      │  │ • Particles     │  │                    │ │
│  └────────┬────────┘  └────────┬────────┘  └────────┬───────────┘ │
│           │                    │                   │                │
│           └────────────────────┴───────────────────┘                │
│                              │                                      │
│           ┌──────────────────┴──────────────────┐                   │
│           │         Physics Engine              │                   │
│           │  • Euler integration                │                   │
│           │  • Gravity (-9.8 m/s²)             │                   │
│           │  • Drag (humidity-dependent)       │                   │
│           │  • Magnus effect (curve)            │                   │
│           │  • Wind vector                      │                   │
│           │  • Altitude factor (air density)   │                   │
│           │  • Collision detection (AABB/Sphere)│                   │
│           └─────────────────────────────────────┘                   │
│                                                                     │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────────────┐  │
│  │  Tournament     │  │   AI Systems    │  │   Persistence      │  │
│  │  Generator      │  │                 │  │   (IndexedDB)     │  │
│  │                 │  │ • Goalkeeper    │  │                    │  │
│  │ • 48 teams      │  │   (predictive)  │  │ • Save state       │  │
│  │ • Group stage   │  │ • Barrier jump  │  │ • Load state       │  │
│  │ • Knockout      │  │   (randomized)  │  │ • Auto-save        │  │
│  │ • Match sim     │  │ • Difficulty    │  │ • Clear save       │  │
│  └─────────────────┘  │   scaling       │  └────────────────────┘  │
│                       └─────────────────┘                         │
│                                                                     │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────────────┐  │
│  │   Audio Engine  │  │   UI/HUD Layer  │  │   Localization     │  │
│  │                 │  │                 │  │                    │  │
│  │ • TTS Narration │  │ • Graffiti HUD  │  │ • PT / EN          │  │
│  │ • Procedural    │  │ • Scoreboard    │  │ • Dynamic text     │  │
│  │   SFX (kick,    │  │ • Power meter   │  │   replacement      │  │
│  │   goal, crowd)  │  │ • Stamina/Humid │  │ • TTS lang switch  │  │
│  │ • Crowd ambience│  │ • Wind indicator│  │                    │  │
│  └─────────────────┘  └─────────────────┘  └────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Especificação de Requisitos Técnicos

### 4.1 Requisitos Funcionais (RF)

| ID | Requisito | Prioridade | Status |
|----|-----------|------------|--------|
| RF-01 | Seleção de 1 entre 48 seleções no início do torneio | Alta | Implementado |
| RF-02 | Torneio completo: Grupos → Oitavas → Quartas → Semi → Final | Alta | Implementado |
| RF-03 | 5 cobranças por partida (pênalti ou falta, aleatório) | Alta | Implementado |
| RF-04 | Cada erro = 1 gol para o adversário | Alta | Implementado |
| RF-05 | Empate → prorrogação com Gol de Ouro (6ª cobrança, dificuldade máxima) | Alta | Implementado |
| RF-06 | Física da bola: gravidade, arrasto, Magnus, vento, altitude | Alta | Implementado |
| RF-07 | Goleiro preditivo com dificuldade escalonada por fase | Alta | Implementado |
| RF-08 | Barreira de 2-4 jogadores com pulo aleatório e colisão | Alta | Implementado |
| RF-09 | Cansaço reduz força a cada chute; stamina < 40% alerta HUD | Alta | Implementado |
| RF-10 | Orbe de energia em posição paralela desafiadora; recupera 25% stamina | Alta | Implementado |
| RF-11 | Umidade afeta arrasto; barra pisca quando > 70% | Média | Implementado |
| RF-12 | Altitude afeta velocidade da bola e curva; placa visual no estádio | Média | Implementado |
| RF-13 | Dia/noite dinâmico aleatório por partida | Média | Implementado |
| RF-14 | Replay em câmera lenta com câmera única | Média | Implementado |
| RF-15 | Comemoração do jogador (braços abertos, pulo) | Média | Implementado |
| RF-16 | Torcida 2D em sprites que pulam no gol, muda cor por seleção | Média | Implementado |
| RF-17 | Narração TTS multilíngue (PT/EN): palavras-chave soltas | Média | Implementado |
| RF-18 | HUD estilo grafite (Permanent Marker, cores neon, painéis tortos) | Alta | Implementado |
| RF-19 | Placar estilo transmissão de TV | Média | Implementado |
| RF-20 | Salvamento automático via IndexedDB | Alta | Implementado |
| RF-21 | PWA com Service Worker e Manifest | Média | Implementado |
| RF-22 | Responsivo: desktop, tablet, mobile | Alta | Implementado |
| RF-23 | Controles: mouse drag, teclado (setas + espaço), touch | Alta | Implementado |
| RF-24 | Toon shader em todos os personagens e campo | Alta | Implementado |
| RF-25 | Jogadores fictícios com atributos únicos (força, precisão, curva) | Alta | Implementado |

### 4.2 Requisitos Não-Funcionais (RNF)

| ID | Requisito | Métrica | Estratégia |
|----|-----------|---------|------------|
| RNF-01 | Performance | 60 FPS em dispositivos médios (Snapdragon 7xx, iPhone 11+) | Three.js r128, geometria low-poly, shadow map 2048px, frustum culling implícito |
| RNF-02 | Bundle size | < 200KB HTML + CDN | Single file, sem assets externos, texturas geradas proceduralmente (Canvas API) |
| RNF-03 | Tempo de carregamento | < 3s em 4G | CDN para Three.js/GSAP, lazy load de fontes, inline de todo código |
| RNF-04 | Offline | Funcionalidade básica offline (Service Worker) | Cache de assets estáticos, IndexedDB para estado |
| RNF-05 | Acessibilidade | Suporte a daltonismo (cores alternativas) | TODO: filtros de cor no HUD, contraste WCAG AA |
| RNF-06 | Compatibilidade | Chrome 90+, Firefox 90+, Safari 14+, Edge 90+ | WebGL 2.0, ES2020, SpeechSynthesis API |
| RNF-07 | Segurança | Sem XSS, sem execução de código arbitrário | Sanitização de innerHTML, sem eval(), CSP recomendado |
| RNF-08 | Privacidade | Nenhum dado enviado para servidor | Tudo local, IndexedDB isolado por origin |

---

## 5. Modelagem de Dados

### 5.1 Estrutura de Estado do Jogo (Game State)

```typescript
interface GameState {
  phase: 'title' | 'select' | 'tournament' | 'match' | 'result' | 'champion';
  playerTeam: Team | null;
  tournament: Tournament | null;
  currentMatch: MatchInfo | null;
  matchData: MatchData;
}

interface Team {
  id: string;           // 3-letter code (e.g., 'bra')
  name: string;         // Display name (PT)
  nameEn: string;       // Display name (EN)
  colors: string[];     // Primary/secondary hex colors
  players: string[];    // Fictional player names (3)
  attr: {
    forca: number;      // 1-10, affects shot power
    precisao: number;   // 1-10, affects accuracy
    curva: number;      // 1-10, affects Magnus effect
  };
}

interface Tournament {
  phase: 'group' | 'round16' | 'quarter' | 'semi' | 'final';
  currentPhase: string;
  groups: Group[];
  groupMatches: Match[];
  knockout: KnockoutMatch[];
  playerGroup: Group | null;
  playerTeamId: string;
  round: number;        // 0-2 for group stage
}

interface Group {
  name: string;         // 'A'-'L'
  teams: GroupTeam[];
}

interface GroupTeam extends Team {
  points: number;
  gf: number;           // Goals for
  ga: number;           // Goals against
  wins: number;
  draws: number;
  losses: number;
}

interface KnockoutMatch {
  home: Team;
  away: Team;
  homeGoals: number;
  awayGoals: number;
  finished: boolean;
  active: boolean;      // Is this the player's match?
}

interface MatchData {
  shots: number;        // Current shot index (0-5, or 6 for golden goal)
  maxShots: number;     // 5 normally, 6 for extra time
  playerGoals: number;
  opponentGoals: number;
  opponentBase: number; // Simulated opponent goals
  stamina: number;        // 0-100
  maxStamina: number;     // 100
  humidity: number;       // 30-80%
  altitude: number;       // 0-3000m
  wind: {
    x: number;          // Lateral wind component
    z: number;          // Longitudinal wind component
    speed: number;      // km/h for display
  };
  isNight: boolean;
  isFreeKick: boolean;
  barrierCount: number; // 0-4
  extraTime: boolean;
  orbCollected: boolean;
}

interface MatchInfo {
  opponent: Team;
  phase: string;
  round: number;
  match?: KnockoutMatch;
}
```

### 5.2 Schema IndexedDB

```
Database: WC2026DB
Version: 1
Object Store: saves
  Key: id (string, primary key)
  Fields:
    - id: "main"
    - state: JSON.stringify(GameState)
    - date: ISO 8601 timestamp
```

---

## 6. Especificação de Física

### 6.1 Parâmetros do Mundo

| Parâmetro | Valor | Unidade | Notas |
|-----------|-------|---------|-------|
| Gravidade | -9.8 | m/s² | Aplicada a cada frame |
| Raio da bola | 0.22 | unidades 3D | Escala arbitrária do mundo |
| Massa da bola | 1.0 | kg | Normalizado |
| Largura do gol | 14.0 | unidades | ~7.32m em escala |
| Altura do gol | 5.0 | unidades | ~2.44m em escala |
| Distância pênalti | 11.0 | unidades | Z = -11 |
| Distância gol | 40.0 | unidades | Z = -40 |

### 6.2 Fórmulas de Física

**Potência do chute:**
```
basePower = 15 + (playerAttr.forca × 2)
staminaFactor = stamina / maxStamina
power = basePower × shootPower × staminaFactor
```

**Altitude (densidade do ar):**
```
altitudeFactor = 1 + (altitude / 5000)    // Bola mais rápida
curveFactor = max(0.2, 1 - (altitude / 4000))  // Menos curva
```

**Umidade (arrasto):**
```
humidityDrag = 1 + (humidity / 100)     // Maior arrasto
```

**Velocidade inicial:**
```
vx = direction.x × 10 × (precisao / 10)
vy = max(0.1, direction.y × 8 × (precisao / 10))
vz = -power × altitudeFactor
```

**Efeito Magnus (curva):**
```
curve = (direction.x × 3 × (curva / 10) × curveFactor) / humidityDrag
ballSpin.y = curve
```

**Atualização por frame (dt):**
```
ballVelocity.y += gravity × dt
ballVelocity *= (1 - (0.3 + humidity / 200) × dt)  // Drag
ballVelocity.x += wind.x × dt × 0.5
ballVelocity.z += wind.z × dt × 0.5
ballVelocity.x += ballSpin.y × dt × 2  // Magnus lateral
ballSpin *= (1 - (altitude / 10000) × dt)  // Spin decay at altitude
ball.position += ballVelocity × dt
```

### 6.3 Colisões

| Objeto | Tipo de Colisão | Resposta |
|--------|----------------|----------|
| Chão | Plano Y=0 | Reflete Vy × 0.6, reduz Vx/Vz × 0.8 |
| Poste esquerdo | Cilindro (raio 0.15) | Reflete Vx × -0.5 |
| Poste direito | Cilindro (raio 0.15) | Reflete Vx × -0.5 |
| Travessão | Cilindro (raio 0.15) | Reflete Vy × -0.5 |
| Goleiro | Esfera (raio 0.8) | Defesa: V refletida para trás, Vy positivo |
| Barreira | Esfera (raio 0.6) | Reflete Vx × 5, Vy positivo × 0.5 + 2, Vz × 0.5 |
| Orbe de energia | Esfera (raio 0.8) | Coleta: stamina += 25, remove mesh |
| Gol | Caixa (-7 < x < 7, 0 < y < 5, z < -40) | Gol detectado |

---

## 7. Especificação de IA

### 7.1 Goleiro Preditivo

**Fase de dificuldade (speed multiplier):**
```
phaseSpeed = {
  group: 0.6,
  round16: 0.8,
  quarter: 1.0,
  semi: 1.2,
  final: 1.5
}
```

**Predição de posição da bola no gol:**
```
timeToGoal = |goalZ - ballZ| / |ballVz|
predictX = ballX + ballVx × timeToGoal
predictY = ballY + ballVy × timeToGoal + 0.5 × gravity × timeToGoal²
clampedX = clamp(predictX, -goalWidth/2 + 0.5, goalWidth/2 - 0.5)
clampedY = clamp(predictY, 0.5, goalHeight - 0.3)
```

**Movimento:**
```
dist = |clampedX - gkX|
reachTime = dist / (speed × 8)
// GSAP tween para gkX em reachTime segundos
```

**Reação de braços:**
- Se predictY > 1.0: braços levantados (defesa alta)
- Se predictY ≤ 1.0: braços abaixados (defesa baixa)

**Pulo:**
```
if (predictY > 1.2 && random() < speed × 0.5) {
  gkY += 1.5 (GSAP tween 0.3s)
}
```

### 7.2 Barreira

- Número de jogadores: `random(2, 3, 4)`
- Posição: Z = goalLineZ + 6.5, espaçados 0.6 unidades
- Pulo: ativado 200-600ms após o chute, duração `sin(t × 5) × 1.5` unidades
- Altura máxima do pulo: 1.5 unidades

---

## 8. Especificação de UI/HUD

### 8.1 Layout Responsivo

```
┌─────────────────────────────────────────────────────────────┐
│  [Vento]          [Placar: BRA 2 x 1 ARG]        [Fase]    │
│  ↗ 15 km/h        [Bandeira] 2 x 1 [Bandeira]   Grupo A   │
│                                                              │
│                                                              │
│                     [Replay / 3D Scene]                      │
│                                                              │
│                                                              │
│  [ALTITUDE]                                    [Orbe]       │
│  2,240m                                        (se ativo)   │
│                                                              │
│  [CANSAÇO ████████░░]  [UMIDADE ██████░░░░]                │
│                                                              │
│              [PÊNALTI — Chute 3/5]                          │
│                                                              │
│              [████████████░░░░] Power Meter                │
│                                                              │
│  [Tutorial]                                  [Narração]     │
│  Como Jogar                                   "GOL!"         │
└─────────────────────────────────────────────────────────────┘
```

### 8.2 Paleta de Cores Grafite

| Elemento | Cor | Hex |
|----------|-----|-----|
| Fundo HUD | Preto translúcido | `rgba(20, 20, 35, 0.92)` |
| Borda primária | Rosa choque | `#ff0066` |
| Destaque | Amarelo ouro | `#ffcc00` |
| Info secundária | Ciano | `#00d4ff` |
| Sucesso | Verde neon | `#00ff88` |
| Perigo | Vermelho | `#ff0066` |
| Texto grafite | Branco + sombra | `#fff` + `drop-shadow` |

### 8.3 Fontes

| Uso | Fonte | Fallback |
|-----|-------|----------|
| Títulos/HUD | Permanent Marker | cursive |
| Placar/Números | Black Ops One | monospace |
| Corpo/UI | Roboto Condensed | sans-serif |

---

## 9. Especificação de Áudio

### 9.1 Narração TTS

```
API: SpeechSynthesis (Web Speech API)
Config PT-BR: { lang: 'pt-BR', rate: 1.2, pitch: 1.1 }
Config EN:    { lang: 'en-US', rate: 1.2, pitch: 1.1 }

Palavras-chave:
  PT: "GOL!", "DEFENDEU!", "NA TRAVE!", "ERROU!", "ENERGIA!", "NA BARRERA!", "PRORROGAÇÃO!", "GOL DE OURO!"
  EN: "GOAL!", "SAVED!", "HIT THE POST!", "MISS!", "ENERGY!", "HIT BARRIER!", "EXTRA TIME!", "GOLDEN GOAL!"
```

### 9.2 Efeitos Sonoros Procedurais (Web Audio API)

| Evento | Tipo | Parâmetros |
|--------|------|------------|
| Chute | Oscilador + Noise | Sawtooth 200Hz, decay 0.3s, noise burst |
| Gol | Oscilador + Reverb | Sine sweep 400→800Hz, 1.5s, reverb 2s |
| Defesa | Noise + Filter | White noise, bandpass 2kHz, 0.5s |
| Trave | Oscilador | Square 800Hz, 0.2s, decay rápido |
| Torcida | Oscilador bank | Múltiplos sines 200-600Hz, tremolo, 3s loop |

---

## 10. Especificação de Deploy

### 10.1 GitHub Pages

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run build
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

### 10.2 Vite Config (vite.config.js)

```javascript
import { defineConfig } from 'vite';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  base: '/world-championship-2026/',
  build: {
    outDir: 'dist',
    assetsDir: 'assets',
    rollupOptions: {
      output: {
        manualChunks: {
          three: ['three'],
          gsap: ['gsap'],
        }
      }
    }
  },
  plugins: [
    VitePWA({
      registerType: 'autoUpdate',
      manifest: {
        name: 'World Championship 2026',
        short_name: 'WC2026',
        theme_color: '#1a1a2e',
        background_color: '#0f0f1a',
        display: 'standalone',
        start_url: '/',
        icons: [
          { src: '/icon-192.png', sizes: '192x192', type: 'image/png' },
          { src: '/icon-512.png', sizes: '512x512', type: 'image/png' }
        ]
      }
    })
  ]
});
```

---

## 11. Plano de Testes Técnicos

### 11.1 Testes Unitários (Jest/Vitest)

| Módulo | Caso de Teste | Esperado |
|--------|--------------|----------|
| Physics | Bola caindo com gravidade | Y decresce a 9.8m/s² |
| Physics | Colisão com chão | Vy inverte × 0.6, Y = 0.22 |
| Physics | Magnus com curva máxima | Trajetória parabólica lateral |
| Physics | Altitude 3000m | vz 60% maior, curva 25% menor |
| Tournament | Sorteio de 48 times | 12 grupos de 4, sem repetição |
| Tournament | Classificação | 2 primeiros + 8 melhores 3º |
| AI | Goleiro fase final | speed = 1.5, antecipa 90% dos chutes |
| State | Save/Load IndexedDB | Estado idêntico após serialização |

### 11.2 Testes de Integração

| Fluxo | Passos | Critério de Sucesso |
|-------|--------|---------------------|
| Partida completa | Selecionar time → iniciar partida → 5 chutes → resultado | Placar correto, stamina decresce, vento aplicado |
| Prorrogação | Empatar 5 chutes → 6º chute → gol | Vitória por golden goal |
| Orbe de energia | Chutar em direção ao orbe → coletar → stamina +25 | Stamina aumenta, orbe desaparece |
| PWA Install | Abrir no Chrome → "Adicionar à tela inicial" → abrir offline | Jogo carrega, IndexedDB funciona |

### 11.3 Testes de Performance

| Métrica | Ferramenta | Target |
|---------|-----------|--------|
| FPS | Chrome DevTools | ≥ 60fps em Moto G Power |
| Bundle size | Webpack Bundle Analyzer | < 200KB (excluindo CDN) |
| TTI | Lighthouse | < 3s em 4G |
| Memory | Chrome DevTools | < 100MB heap |

---

## 12. Riscos Técnicos e Mitigações

| Risco | Probabilidade | Impacto | Mitigação |
|-------|--------------|---------|-----------|
| WebGL não suportado em dispositivo antigo | Média | Alto | Fallback: mensagem "Seu navegador não suporta WebGL" |
| IndexedDB quota excedida | Baixa | Médio | Compressão de estado (JSON minify), limite de saves |
| TTS não disponível no navegador | Média | Baixo | Fallback: texto apenas no HUD |
| Performance baixa em mobile | Média | Alto | Reduzir shadow map, LOD, desativar partículas |
| Touch events conflitando com scroll | Média | Médio | `touch-action: none` no canvas, `preventDefault` |
| Física manual com bugs de colisão | Baixa | Alto | Testes extensivos, bounding boxes visuais para debug |
| Service Worker cache stale | Baixa | Médio | Versionamento de cache, `skipWaiting` |

---

## 13. Checklist de Implementação

- [x] Three.js scene, camera, renderer
- [x] Toon shader materials (MeshToonMaterial)
- [x] Field, goal, ball, goalkeeper, kicker meshes
- [x] Custom physics (gravity, drag, Magnus, wind)
- [x] Collision detection (ball-goalkeeper, ball-barrier, ball-posts)
- [x] Input system (mouse drag, keyboard, touch)
- [x] Power meter UI
- [x] Wind indicator with particles
- [x] Stamina and humidity bars with alerts
- [x] Altitude sign (3D + HUD)
- [x] Day/night cycle with floodlights
- [x] Tournament generator (48 teams, 12 groups)
- [x] Group stage logic (3 rounds, points table)
- [x] Knockout stage logic (Round of 16 → Final)
- [x] Golden goal extra time
- [x] Goalkeeper AI with phase scaling
- [x] Barrier with random jump
- [x] Energy orb with collection
- [x] Replay system (slow motion, camera follow)
- [x] Celebration animations (player + crowd)
- [x] Crowd sprites with color change
- [x] TTS narration (PT/EN)
- [x] Graffiti HUD styling
- [x] Responsive layout
- [x] IndexedDB save/load
- [x] PWA manifest and service worker
- [x] Localization (PT/EN)
- [x] Procedural SFX (Web Audio API) — **CONCLUIDO: kick, goal, save, post, miss, barrier, orb, crowd, whistle**
- [x] Accessibility (colorblind modes, ARIA labels, high contrast, keyboard shortcuts) — **CONCLUIDO**
- [x] Vite build setup (package.json, vite.config.js, PWA plugin) — **CONCLUIDO**
- [x] Icon assets (SVG 192/512) — **CONCLUIDO**
- [ ] Unit tests — **PENDENTE**
- [ ] Analytics / Error tracking (Sentry) — **PENDENTE**

---

## 14. Glossário

| Termo | Definição |
|-------|-----------|
| **PWA** | Progressive Web App — aplicação web instalável no mobile |
| **Toon Shader** | Cel shading — técnica de renderização que simula desenho animado |
| **Magnus Effect** | Efeito aerodinâmico que curva a trajetória de uma bola em rotação |
| **Golden Goal** | Regra de desempate onde o primeiro gol na prorrogação define o vencedor |
| **IndexedDB** | Banco de dados NoSQL do navegador, persistente entre sessões |
| **TTS** | Text-to-Speech — síntese de voz para narração |
| **FSM** | Finite State Machine — máquina de estados finitos para gerenciamento de fases do jogo |
| **HUD** | Heads-Up Display — interface de informações sobreposta à tela |

---

*Documento aprovado para desenvolvimento. Revisões devem ser versionadas e registradas na seção de changelog (não incluída nesta versão inicial).*
