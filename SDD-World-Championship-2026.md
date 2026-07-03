# Software Design Document (SDD)
## World Championship 2026 — Shoot Em In Reloaded

**Versão:** 1.0  
**Data:** 2026-07-03  
**Autor:** Engenharia de Software  
**Status:** Aprovado para Prototipagem

---

## 1. Introdução

### 1.1 Propósito
Este documento descreve o design arquitetural de software do jogo *World Championship 2026 — Shoot Em In Reloaded*, detalhando a estrutura de módulos, padrões de projeto, fluxos de dados, interfaces entre componentes e decisões de design que fundamentam a implementação.

### 1.2 Escopo
O SDD abrange:
- Arquitetura de software e padrões de projeto
- Diagramas de classes e sequência
- Fluxos de dados e estados
- Interfaces entre módulos
- Estratégias de persistência
- Decisões de design e trade-offs

### 1.3 Referências
- TDD-World-Championship-2026.md (Technical Design Document)
- Three.js Documentation (r128)
- GSAP Documentation (v3.12)
- Web Audio API Specification
- IndexedDB Specification (W3C)

---

## 2. Visão Arquitetural

### 2.1 Estilo Arquitetural
**Arquitetura Modular com State Machine (FSM) e Observer Pattern**

O jogo adota uma arquitetura modular monolítica (single-file) com separação de responsabilidades por domínio funcional. A navegação entre telas é gerenciada por uma Finite State Machine (FSM) centralizada, garantindo previsibilidade de estados e transições.

**Justificativa:**
- Single-file elimina complexidade de bundling e módulos ES6 para protótipo
- FSM previne estados inválidos (ex: chutar durante replay)
- Observer implícito via callbacks de eventos DOM e GSAP

### 2.2 Diagrama de Arquitetura de Alto Nível

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           Aplicação (Browser)                            │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        Presentation Layer                          │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐ │   │
│  │  │   HTML/CSS   │  │   HUD Layer  │  │   Screen Manager       │ │   │
│  │  │   (DOM)      │  │   (Overlay)  │  │   (FSM)                │ │   │
│  │  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────────┘ │   │
│  │         │                 │                     │                  │   │
│  │         └─────────────────┴─────────────────────┘                  │   │
│  │                           │                                       │   │
│  │  ┌────────────────────────┴────────────────────────┐              │   │
│  │  │              Application Layer                   │              │   │
│  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────┐ │              │   │
│  │  │  │ Game State   │  │ Input        │  │ Audio│ │              │   │
│  │  │  │ Manager      │  │ Controller   │  │ Engine│              │   │
│  │  │  │ (FSM + Data) │  │ (Mouse/Key/  │  │ (TTS +│              │   │
│  │  │  │              │  │  Touch)      │  │  Web  │              │   │
│  │  │  └──────┬───────┘  └──────┬───────┘  │ Audio)│              │   │
│  │  │         │                 │          └───────┘              │   │
│  │  │         └─────────────────┴─────────────────┘               │   │
│  │  │                           │                                  │   │
│  │  │  ┌────────────────────────┴────────────────────────┐        │   │
│  │  │  │              Domain Layer                        │        │   │
│  │  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────┐ │        │   │
│  │  │  │  │ Tournament   │  │ Match        │  │ Team │ │        │   │
│  │  │  │  │ Engine       │  │ Engine       │  │ Data │ │        │   │
│  │  │  │  │ (Groups +    │  │ (Physics +   │  │ (48  │ │        │   │
│  │  │  │  │  Knockout)   │  │  AI + Rules) │  │ teams)│ │        │   │
│  │  │  │  └──────┬───────┘  └──────┬───────┘  └──────┘ │        │   │
│  │  │  │         │                 │                    │        │   │
│  │  │  │         └─────────────────┴────────────────────┘        │   │
│  │  │  │                           │                            │   │
│  │  │  │  ┌────────────────────────┴────────────────────────┐   │   │
│  │  │  │  │              Infrastructure Layer              │   │   │
│  │  │  │  │  ┌──────────────┐  ┌──────────────┐         │   │   │
│  │  │  │  │  │ 3D Renderer  │  │ Persistence  │         │   │   │
│  │  │  │  │  │ (Three.js)   │  │ (IndexedDB)  │         │   │   │
│  │  │  │  │  └──────────────┘  └──────────────┘         │   │   │
│  │  │  └─────────────────────────────────────────────────┘   │   │
│  │  └─────────────────────────────────────────────────────────┘   │
│  └─────────────────────────────────────────────────────────────────┘
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                        External Dependencies                       │   │
│  │  Three.js (CDN) │ GSAP (CDN) │ Google Fonts (CDN) │ Browser APIs │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Padrões de Projeto

### 3.1 Padrões Utilizados

| Padrão | Onde Aplicado | Justificativa |
|--------|--------------|---------------|
| **State Machine (FSM)** | GameState.phase | Transições controladas entre telas; evita estados inconsistentes |
| **Observer** | Eventos DOM + GSAP callbacks | Desacoplamento entre input e lógica de jogo |
| **Factory** | createBarrierPlayer, createCrowd | Criação parametrizada de objetos 3D sem duplicação |
| **Strategy** | Input handlers (Mouse/Key/Touch) | Algoritmos de input intercambiáveis sem alterar core |
| **Singleton** | Scene, Renderer, Camera | Apenas uma instância de contexto WebGL por aplicação |
| **Command** | executeShot | Encapsulação da ação de chute para undo/replay (futuro) |
| **Template Method** | endMatch → continueTournament | Fluxo comum de finalização com variações por fase |

### 3.2 Anti-Padrões Evitados

| Anti-Padrão | Mitigação |
|-------------|-----------|
| **God Object** | GameState centralizado mas delega para módulos especializados (Tournament, Match, Physics) |
| **Spaghetti Code** | Funções com responsabilidade única; separação clara DOM/3D/Logic |
| **Callback Hell** | Uso de GSAP timelines para sequências; async/await para IndexedDB |
| **Magic Numbers** | Constantes centralizadas (goalWidth, gravity, maxShots) |

---

## 4. Diagrama de Classes

### 4.1 Diagrama UML (Simplificado)

```
┌─────────────────────┐
│    GameState        │
├─────────────────────┤
│ - phase: string     │
│ - playerTeam: Team    │
│ - tournament: Tourn │
│ - currentMatch: Match│
│ - matchData: MData  │
├─────────────────────┤
│ + transition(to)    │
│ + save()            │
│ + load()            │
└──────────┬──────────┘
           │
     ┌─────┴─────┐
     │           │
┌────▼────┐ ┌────▼────┐
│  Team   │ │Tournament│
├─────────┤ ├─────────┤
│ id      │ │ phase    │
│ name    │ │ groups[] │
│ colors[]│ │ knockout[]│
│ players[]││ round    │
│ attr    │ │ playerGroup│
├─────────┤ ├─────────┤
│ +getName()│ +advance()│
│ +getAttr()│ +getNext()│
└─────────┘ └─────────┘

┌─────────────────────┐
│   MatchEngine       │
├─────────────────────┤
│ - ball: Mesh        │
│ - goalkeeper: Group │
│ - kicker: Group     │
│ - barrier: Group[]  │
│ - velocity: Vector3 │
│ - spin: Vector3     │
├─────────────────────┤
│ +setupShot()        │
│ +executeShot()      │
│ +updatePhysics(dt)  │
│ +checkCollisions()  │
│ +endShot(result)    │
│ +resetPositions()   │
└──────────┬──────────┘
           │
     ┌─────┴─────┐
     │           │
┌────▼────┐ ┌────▼────┐
│ Physics │ │   AI    │
├─────────┤ ├─────────┤
│ gravity │ │ predict()│
│ drag    │ │ react()  │
│ magnus  │ │ jump()   │
│ wind    │ │ celebrate()│
├─────────┤ ├─────────┤
│ +apply()│ │ +moveGK()│
│ +collide()│ +moveBarrier()│
└─────────┘ └─────────┘

┌─────────────────────┐
│   InputController   │
├─────────────────────┤
│ - isAiming: bool    │
│ - mouseStart: Point │
│ - mouseCurrent: Point│
│ - keys: Map         │
│ - shootPower: float │
│ - shootDir: Point   │
├─────────────────────┤
│ +onMouseDown()      │
│ +onMouseMove()      │
│ +onMouseUp()        │
│ +onTouchStart()     │
│ +onTouchMove()      │
│ +onKeyDown()        │
│ +onKeyUp()          │
│ +getAimVector()     │
└─────────────────────┘

┌─────────────────────┐
│   UIManager         │
├─────────────────────┤
│ - screens: Map      │
│ - hudElements: Map  │
│ - narrationQueue[]  │
├─────────────────────┤
│ +showScreen(id)     │
│ +hideScreen(id)     │
│ +updateScore()      │
│ +updateStamina()    │
│ +showNarration()    │
│ +updateWind()       │
│ +renderTournament() │
│ +renderTeamSelect() │
└─────────────────────┘

┌─────────────────────┐
│   AudioEngine       │
├─────────────────────┤
│ - ctx: AudioContext │
│ - tts: SpeechSynth  │
├─────────────────────┤
│ +playKick()         │
│ +playGoal()         │
│ +playSave()         │
│ +playCrowd()        │
│ +speak(text, lang)  │
│ +setVolume(v)       │
└─────────────────────┘

┌─────────────────────┐
│   Persistence       │
├─────────────────────┤
│ - db: IDBDatabase   │
│ - storeName: string │
├─────────────────────┤
│ +init()             │
│ +save(state)        │
│ +load(): State      │
│ +clear()            │
└─────────────────────┘
```

### 4.2 Relações de Dependência

```
GameState
  ├── depends on: Tournament, MatchData, Team
  ├── uses: Persistence (save/load)
  └── notifies: UIManager (screen transitions)

MatchEngine
  ├── depends on: Physics, AI
  ├── uses: InputController (aim vector)
  ├── notifies: UIManager (score updates)
  └── uses: AudioEngine (SFX)

Tournament
  ├── depends on: Team (48 instances)
  └── uses: MatchData (for simulation)

InputController
  ├── listens to: DOM Events
  └── provides: AimVector to MatchEngine

UIManager
  ├── listens to: GameState changes
  ├── listens to: MatchEngine events
  └── uses: DOM API, CSS animations

AudioEngine
  ├── uses: Web Audio API
  ├── uses: SpeechSynthesis API
  └── listens to: MatchEngine events

Persistence
  ├── uses: IndexedDB API
  └── stores: GameState (serialized JSON)
```

---

## 5. Fluxos de Dados

### 5.1 Fluxo: Início do Jogo → Seleção de Time

```
Usuário
  │
  ▼
[Clique "NOVO TORNEIO"]
  │
  ▼
UIManager.showScreen('teams')
  │
  ▼
UIManager.renderTeamSelect()
  │  ├── Carrega TEAMS[] (48 seleções)
  │  ├── Gera cards DOM
  │  └── Aplica estilos grafite
  │
  ▼
Usuário seleciona time
  │
  ▼
GameState.playerTeam = selectedTeam
  │
  ▼
[Clique "CONFIRMAR"]
  │
  ▼
Tournament.generateTournament()
  │  ├── Embaralha 48 times
  │  ├── Cria 12 grupos de 4
  │  └── Identifica grupo do jogador
  │
  ▼
GameState.tournament = tournament
  │
  ▼
Persistence.save(gameState)
  │
  ▼
UIManager.showScreen('tournament')
  │
  ▼
UIManager.renderTournament()
```

### 5.2 Fluxo: Partida Completa (1 Cobrança)

```
Usuário
  │
  ▼
[Clique "PRÓXIMA PARTIDA"]
  │
  ▼
MatchEngine.setupMatch()
  │  ├── Inicializa MatchData
  │  ├── Define vento, altitude, umidade, dia/noite
  │  ├── Posiciona bola, kicker, goleiro
  │  ├── Cria barreira (se falta)
  │  └── Gera orbe de energia (30% chance)
  │
  ▼
UIManager.updateHUD()
  │
  ▼
Usuário mira (Mouse drag / Teclas / Touch)
  │
  ▼
InputController.updateAim()
  │  ├── Calcula shootPower (distância do drag)
  │  └── Calcula shootDirection (ângulo do drag)
  │
  ▼
UIManager.updatePowerMeter()
  │
  ▼
Usuário solta / pressiona ESPAÇO
  │
  ▼
MatchEngine.executeShot()
  │  ├── Aplica fator de cansaço
  │  ├── Aplica fator de altitude
  │  ├── Aplica fator de umidade
  │  ├── Calcula velocidade inicial
  │  ├── Define spin (Magnus)
  │  ├── Anima kicker (GSAP)
  │  ├── Goleiro.react() (predição + movimento)
  │  └── Barreira.jump() (aleatório)
  │
  ▼
[Game Loop: 60fps]
  │
  ▼
MatchEngine.updatePhysics(dt)
  │  ├── Aplica gravidade
  │  ├── Aplica arrasto (umidade)
  │  ├── Aplica vento
  │  ├── Aplica Magnus
  │  ├── Decai spin (altitude)
  │  └── Atualiza posição da bola
  │
  ▼
MatchEngine.checkCollisions()
  │  ├── Chão? → Reflete Vy
  │  ├── Goleiro? → DEFESA
  │  ├── Barreira? → Reflete + Vy positivo
  │  ├── Poste/Travessão? → NA TRAVE
  │  ├── Gol? → GOL
  │  ├── Orbe? → Recupera stamina
  │  └── Fora? → ERROU
  │
  ▼
[Resultado detectado]
  │
  ▼
MatchEngine.endShot(result)
  │  ├── Atualiza placar
  │  ├── AudioEngine.play(result)
  │  ├── UIManager.showNarration(result)
  │  ├── Ativa replay (câmera lenta)
  │  └── Aguarda 2.5s
  │
  ▼
MatchEngine.resetPositions()
  │
  ▼
MatchEngine.nextShot()
  │  ├── Incrementa shot counter
  │  ├── Verifica fim de partida
  │  └── Se não: setupShot()
  │
  ▼
[Se 5 chutes completos]
  │
  ▼
MatchEngine.endMatch()
  │  ├── Calcula resultado final
  │  ├── Atualiza Tournament (pontos/classificação)
  │  ├── Persistence.save()
  │  └── UIManager.showScreen('result')
```

### 5.3 Fluxo: Persistência (Save/Load)

```
[Evento: endMatch, saveAndQuit, ou auto-save]
  │
  ▼
Persistence.save(gameState)
  │  ├── JSON.stringify(gameState)
  │  ├── Comprime (opcional: LZ-string)
  │  ├── Abre transação IndexedDB (readwrite)
  │  ├── objectStore.put({ id: 'main', state, date })
  │  └── Commit
  │
  ▼
[Evento: loadGame]
  │
  ▼
Persistence.load()
  │  ├── Abre transação IndexedDB (readonly)
  │  ├── objectStore.get('main')
  │  ├── JSON.parse(result.state)
  │  ├── Restaura GameState
  │  └── UIManager.renderTournament()
```

---

## 6. Especificação de Interfaces

### 6.1 Interface: GameState → UIManager

```typescript
interface GameStateEvents {
  onPhaseChange: (from: Phase, to: Phase) => void;
  onScoreUpdate: (home: number, away: number) => void;
  onStaminaChange: (value: number, max: number) => void;
  onMatchEnd: (result: MatchResult) => void;
  onTournamentAdvance: (phase: TournamentPhase) => void;
}
```

### 6.2 Interface: InputController → MatchEngine

```typescript
interface AimVector {
  power: number;      // 0.0 - 1.0
  directionX: number; // -1.0 - 1.0 (lateral)
  directionY: number; // -0.5 - 1.0 (vertical/altura)
}

interface InputController {
  getAimVector(): AimVector | null;
  isAiming(): boolean;
  reset(): void;
}
```

### 6.3 Interface: MatchEngine → AudioEngine

```typescript
interface AudioEvent {
  type: 'kick' | 'goal' | 'save' | 'post' | 'miss' | 'barrier' | 'orb' | 'crowd' | 'narration';
  payload?: {
    text?: string;      // For narration
    lang?: 'pt-BR' | 'en-US';
    intensity?: number;  // For crowd volume
  };
}
```

### 6.4 Interface: Persistence → GameState

```typescript
interface SaveData {
  id: 'main';
  state: string;      // JSON.stringify(GameState)
  version: number;    // Schema version for migration
  date: string;       // ISO 8601
}

interface PersistenceAPI {
  init(): Promise<void>;
  save(data: SaveData): Promise<void>;
  load(): Promise<SaveData | null>;
  clear(): Promise<void>;
}
```

---

## 7. Diagrama de Estados (FSM)

### 7.1 Máquina de Estados do Jogo

```
                    ┌─────────────┐
                    │    TITLE    │
                    │  (tela ini- │
                    │   cial)     │
                    └──────┬──────┘
                           │ [Novo Torneio]
                           ▼
                    ┌─────────────┐
                    │   SELECT    │
                    │ (escolha de │
                    │   time)     │
                    └──────┬──────┘
                           │ [Confirmar]
                           ▼
                    ┌─────────────┐
                    │ TOURNAMENT  │
                    │ (tabela de  │
                    │  jogos)     │
                    └──────┬──────┘
                           │ [Próxima Partida]
                           ▼
                    ┌─────────────┐
                    │   MATCH     │
                    │ (5 chutes + │
                    │  possível   │
                    │  prorroga-  │
                    │   ção)      │
                    └──────┬──────┘
                           │ [Fim da Partida]
                           ▼
                    ┌─────────────┐
                    │   RESULT    │
                    │ (placar e  │
                    │  detalhes)  │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │ [Continuar]│            │ [Fim do Torneio]
              ▼            │            ▼
       ┌─────────────┐     │     ┌─────────────┐
       │ TOURNAMENT  │◄────┘     │  CHAMPION   │
       │  (próxima   │           │  (tela de   │
       │   rodada)   │           │  vitória)   │
       └─────────────┘           └─────────────┘
              │                         │
              │ [Não classificou]       │ [Novo Torneio]
              ▼                         ▼
       ┌─────────────┐           ┌─────────────┐
       │   TITLE     │◄──────────┘   TITLE     │
       │  (game over)│               (reinicia)│
       └─────────────┘               └─────────────┘
```

### 7.2 Máquina de Estados da Partida (Sub-FSM)

```
                    ┌─────────────┐
                    │   SETUP     │
                    │ (posiciona  │
                    │  elementos) │
                    └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │   AIMING    │
                    │ (aguardando │
                    │  input do   │
                    │   jogador)  │
                    └──────┬──────┘
                           │ [Chute executado]
                           ▼
                    ┌─────────────┐
                    │  SHOOTING   │
                    │ (física ati-│
                    │   va, bola  │
                    │   em mov.)  │
                    └──────┬──────┘
                           │ [Colisão detectada]
                           ▼
                    ┌─────────────┐
                    │   REPLAY    │
                    │ (câmera lenta│
                    │  mostrando  │
                    │  resultado) │
                    └──────┬──────┘
                           │ [2.5s decorridos]
                           ▼
                    ┌─────────────┐
                    │  DECISION   │
                    │ (verifica se│
                    │  fim da     │
                    │  partida)   │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │ [Mais chutes]│           │ [5 chutes + não empatou]
              ▼            │            ▼
       ┌─────────────┐    │     ┌─────────────┐
       │   SETUP     │◄───┘     │  END_MATCH  │
       │  (próximo   │          │  (calcula   │
       │   chute)    │          │  resultado) │
       └─────────────┘          └─────────────┘
              │                         │
              │ [Empate após 5]       │
              ▼                         ▼
       ┌─────────────┐           ┌─────────────┐
       │ EXTRA_TIME  │           │   RESULT    │
       │ (6º chute,  │           │  (tela de   │
       │  dific. max)│           │  resultado) │
       └─────────────┘           └─────────────┘
```

---

## 8. Estrutura de Diretórios (Recomendada para Vite)

```
world-championship-2026/
├── public/
│   ├── icon-192.png
│   ├── icon-512.png
│   └── manifest.json
├── src/
│   ├── main.js                    # Entry point, inicialização
│   ├── index.html                 # Template HTML
│   ├── styles/
│   │   └── main.css               # Estilos globais + HUD grafite
│   ├── engine/
│   │   ├── GameState.js           # FSM central + estado global
│   │   ├── Tournament.js          # Geração e gestão do torneio
│   │   ├── MatchEngine.js         # Física, colisões, regras da partida
│   │   ├── Physics.js             # Integração Euler, gravidade, Magnus
│   │   └── AI.js                  # Goleiro preditivo, barreira
│   ├── rendering/
│   │   ├── SceneManager.js        # Three.js scene, camera, renderer
│   │   ├── Field.js               # Campo, grama, linhas, estádio
│   │   ├── Characters.js          # Jogador, goleiro, barreira (meshes)
│   │   ├── Ball.js                # Bola, trail, partículas
│   │   ├── Crowd.js               # Sprites de torcedores
│   │   └── Effects.js             # Partículas de vento, luzes, orbe
│   ├── input/
│   │   └── InputController.js     # Mouse, keyboard, touch handlers
│   ├── ui/
│   │   ├── UIManager.js           # Gestão de telas e HUD
│   │   ├── Screens.js             # Renderização de cada tela
│   │   └── Localization.js        # PT/EN textos e TTS
│   ├── audio/
│   │   └── AudioEngine.js         # Web Audio API + SpeechSynthesis
│   ├── data/
│   │   └── Teams.js               # 48 seleções, jogadores, atributos
│   └── persistence/
│       └── IndexedDBStore.js      # Save/load/clear
├── vite.config.js                 # Config Vite + PWA plugin
├── package.json
└── README.md
```

**Nota:** A versão atual (protótipo) é single-file para simplicidade. A estrutura acima representa a arquitetura alvo para produção com Vite.

---

## 9. Decisões de Design e Trade-offs

### 9.1 Decisão: Single-File vs. Modular

| Aspecto | Single-File (Atual) | Modular (Vite) |
|---------|---------------------|----------------|
| **Complexidade** | Baixa | Média |
| **Bundle Size** | ~89KB HTML | ~50KB JS + split chunks |
| **Cache/CDN** | Tudo cacheado junto | Chunks separados, cache granular |
| **HMR** | Não disponível | Disponível (Vite) |
| **Manutenção** | Difícil (código misturado) | Fácil (separação clara) |
| **Build** | Nenhum necessário | Vite necessário |
| **Escolha** | ✅ Protótipo | 🎯 Produção |

**Justificativa:** Single-file elimina barreira de entrada para testes rápidos no GitHub Pages sem pipeline de build. Modularização é recomendada para iterações futuras.

### 9.2 Decisão: Física Custom vs. Cannon.js

| Aspecto | Custom | Cannon.js |
|---------|--------|-----------|
| **Tamanho** | 0KB extra | ~80KB gzipped |
| **Precisão** | Suficiente para penalty | Alta (corpos rígidos) |
| **Controle** | Total (Magnus, altitude) | Limitado (requer hacks) |
| **Curva de aprendizado** | Baixa | Média |
| **Performance** | Otimizado para caso de uso | Genérico, overhead |
| **Escolha** | ✅ Custom | ❌ Não necessário |

**Justificativa:** Penalty/free kick é um caso de uso físico simples (1 bola, poucos colisores). Física custom permite implementar Magnus, altitude e umidade de forma direta sem abstrações de engine física.

### 9.3 Decisão: Three.js vs. Babylon.js vs. Unity WebGL

| Aspecto | Three.js | Babylon.js | Unity WebGL |
|---------|----------|------------|-------------|
| **Tamanho** | ~150KB | ~200KB | ~5MB+ |
| **PWA** | ✅ Nativo | ✅ Nativo | ❌ Pesado |
| **GitHub Pages** | ✅ Ideal | ✅ Ideal | ❌ Limitado |
| **Toon Shader** | MeshToonMaterial | CelMaterial | Custom Shader |
| **Curva** | Média | Média | Baixa (editor) |
| **Escolha** | ✅ Three.js | — | ❌ Overkill |

**Justificativa:** Three.js é o padrão de facto para WebGL leve, com comunidade vasta e documentação extensa. Unity WebGL é inviável para PWA/GitHub Pages devido ao tamanho do runtime.

### 9.4 Decisão: IndexedDB vs. LocalStorage

| Aspecto | IndexedDB | LocalStorage |
|---------|-----------|--------------|
| **Limite** | ~50MB+ | ~5MB |
| **Dados** | Estruturado, binário | Apenas strings |
| **Async** | ✅ Sim | ❌ Síncrono (bloqueia) |
| **Complexidade** | Média (API verbosa) | Baixa |
| **Escolha** | ✅ IndexedDB | ❌ Insuficiente |

**Justificativa:** Estado do torneio com 48 seleções, estatísticas e múltiplos saves pode exceder 5MB. IndexedDB oferece async e capacidade superior.

### 9.5 Decisão: TTS vs. Assets de Áudio Pré-gravados

| Aspecto | TTS (SpeechSynthesis) | Assets Pré-gravados |
|---------|----------------------|---------------------|
| **Tamanho** | 0KB | ~500KB-2MB |
| **Qualidade** | Variável por navegador | Alta (profissional) |
| **Multilíngue** | ✅ Automático | ❌ Requer gravação |
| **Narração** | "GOL!", "DEFENDEU!" | Mesmo conteúdo |
| **Escolha** | ✅ TTS | 🎯 Futuro (assets) |

**Justificativa:** TTS elimina dependência de assets externos, reduzindo bundle e permitindo narração dinâmica. Qualidade é aceitável para protótipo.

---

## 10. Especificação de Segurança

### 10.1 Threat Model

| Ameaça | Vetor | Mitigação |
|--------|-------|-----------|
| XSS via innerHTML | Injeção de código em nomes de times/jogadores | Sanitização: nomes são hardcoded (TEAMS[]), não input do usuário |
| XSS via save state | Injeção JSON malicioso no IndexedDB | Validação de schema ao carregar; rejeitar campos desconhecidos |
| CSRF | Não aplicável (sem backend) | N/A |
| Data Tampering | Modificação de save no DevTools | Não crítico para jogo single-player; checksum opcional |
| Service Worker Hijacking | SW malicioso | SW inline, não carregado de CDN externo |

### 10.2 Content Security Policy (Recomendado)

```http
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' https://cdnjs.cloudflare.com;
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  font-src 'self' https://fonts.gstatic.com;
  connect-src 'self';
  img-src 'self' blob: data:;
  media-src 'self';
  worker-src 'self';
```

---

## 11. Especificação de Performance

### 11.1 Budget de Performance

| Métrica | Target | Máximo Aceitável | Ferramenta de Medição |
|---------|--------|------------------|----------------------|
| First Contentful Paint (FCP) | < 1.5s | < 3s | Lighthouse |
| Time to Interactive (TTI) | < 3s | < 5s | Lighthouse |
| FPS (Gameplay) | 60fps | 30fps | Chrome DevTools |
| Memory Heap | < 80MB | < 150MB | Chrome DevTools |
| Bundle Size | < 200KB | < 500KB | Webpack Bundle Analyzer |
| Total Blocking Time | < 200ms | < 500ms | Lighthouse |

### 11.2 Estratégias de Otimização

| Estratégia | Implementação | Impacto |
|-------------|--------------|---------|
| **Lazy Loading** | Fontes Google com `display=swap` | Reduz FCP |
| **Code Splitting** | Three.js e GSAP em chunks separados (Vite) | Cache eficiente |
| **Geometry Instancing** | Crowd sprites (não meshes individuais) | -50% draw calls |
| **Texture Procedural** | Canvas API para bandeiras, altitude sign | Zero downloads |
| **Shadow Optimization** | Shadow map 2048px, apenas objetos dinâmicos | -30% GPU |
| **Object Pooling** | Reutilizar meshes de barreira entre chutes | -20% GC pressure |
| **Debounce/Throttle** | Input events a 60fps | -10% CPU |

---

## 12. Especificação de Testes de Software

### 12.1 Pirâmide de Testes

```
         ┌─────────┐
         │   E2E   │  (5%)  - Cypress/Playwright: fluxo completo do torneio
         │         │
        ┌┴─────────┴┐
        │ Integration│  (15%) - Jest: Tournament + MatchEngine + Persistence
        │            │
       ┌┴────────────┴┐
       │    Unit       │  (80%) - Jest/Vitest: Physics, AI, Input, Localization
       │               │
       └───────────────┘
```

### 12.2 Casos de Teste Unitário

```javascript
// Physics.test.js
describe('Physics Engine', () => {
  test('gravity reduces ball Y velocity', () => {
    const v = new THREE.Vector3(0, 5, -10);
    applyGravity(v, 0.016); // 1 frame @ 60fps
    expect(v.y).toBeLessThan(5);
  });

  test('altitude increases ball speed', () => {
    const v1 = calculateInitialVelocity({ power: 1, altitude: 0 });
    const v2 = calculateInitialVelocity({ power: 1, altitude: 3000 });
    expect(v2.z).toBeLessThan(v1.z); // More negative = faster
  });

  test('humidity increases drag', () => {
    const v = new THREE.Vector3(10, 0, -20);
    applyDrag(v, 80, 0.016); // 80% humidity
    const v2 = v.clone();
    applyDrag(v2, 20, 0.016); // 20% humidity
    expect(v.length()).toBeLessThan(v2.length());
  });

  test('Magnus curves ball trajectory', () => {
    const v = new THREE.Vector3(0, 5, -10);
    const spin = new THREE.Vector3(0, 5, 0);
    applyMagnus(v, spin, 0.016);
    expect(v.x).not.toBe(0);
  });
});

// Tournament.test.js
describe('Tournament Generator', () => {
  test('generates 12 groups of 4 teams', () => {
    const t = generateTournament();
    expect(t.groups).toHaveLength(12);
    t.groups.forEach(g => expect(g.teams).toHaveLength(4));
  });

  test('no duplicate teams across groups', () => {
    const t = generateTournament();
    const allIds = t.groups.flatMap(g => g.teams.map(t => t.id));
    expect(new Set(allIds).size).toBe(48);
  });

  test('player team is in one group', () => {
    const t = generateTournament();
    const playerGroup = findPlayerGroup(t, 'bra');
    expect(playerGroup).toBeDefined();
    expect(playerGroup.teams.some(t => t.id === 'bra')).toBe(true);
  });
});

// AI.test.js
describe('Goalkeeper AI', () => {
  test('predicts ball position at goal line', () => {
    const ball = { pos: new THREE.Vector3(2, 1, -20), vel: new THREE.Vector3(1, 2, -10) };
    const predict = predictGoalPosition(ball);
    expect(predict.x).toBeGreaterThan(ball.pos.x);
  });

  test('difficulty scales with phase', () => {
    expect(getPhaseSpeed('group')).toBe(0.6);
    expect(getPhaseSpeed('final')).toBe(1.5);
  });
});
```

### 12.3 Casos de Teste de Integração

```javascript
// MatchFlow.test.js
describe('Full Match Flow', () => {
  test('completes 5-shot match with correct scoring', async () => {
    const match = createMatch({ playerTeam: TEAMS[0], opponent: TEAMS[1] });

    for (let i = 0; i < 5; i++) {
      match.aim({ power: 0.8, directionX: 0.2, directionY: 0.5 });
      match.shoot();
      await waitForShotEnd();
    }

    expect(match.shots).toBe(5);
    expect(match.isFinished()).toBe(true);
    expect(match.playerGoals + match.opponentGoals).toBeGreaterThan(0);
  });

  test('triggers extra time on draw', async () => {
    const match = createMatch({ playerTeam: TEAMS[0], opponent: TEAMS[1] });
    // Simulate 5 shots all saved (0-0)
    // ... setup mocks ...
    expect(match.extraTime).toBe(true);
    expect(match.maxShots).toBe(6);
  });

  test('golden goal ends match immediately', async () => {
    const match = createMatch({ playerTeam: TEAMS[0], opponent: TEAMS[1], extraTime: true });
    match.aim({ power: 1, directionX: 0, directionY: 0.3 });
    match.shoot();
    await waitForShotEnd();
    expect(match.isFinished()).toBe(true);
    expect(match.playerGoals).toBe(1);
  });
});
```

---

## 13. Especificação de Deploy

### 13.1 Pipeline CI/CD (GitHub Actions)

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run test:unit
      - run: npm run test:integration
      - run: npm run lint
      - run: npm run build

  lighthouse:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
      - uses: treosh/lighthouse-ci-action@v10
        with:
          configPath: './lighthouserc.json'
          uploadArtifacts: true

# .github/workflows/deploy.yml
name: Deploy
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

### 13.2 Configuração Lighthouse CI

```json
// lighthouserc.json
{
  "ci": {
    "collect": {
      "url": ["http://localhost:4173/"],
      "numberOfRuns": 3
    },
    "assert": {
      "preset": "lighthouse:recommended",
      "assertions": {
        "categories:performance": ["warn", { "minScore": 0.8 }],
        "categories:accessibility": ["error", { "minScore": 0.9 }],
        "categories:best-practices": ["warn", { "minScore": 0.9 }],
        "categories:seo": ["warn", { "minScore": 0.8 }],
        "categories:pwa": ["error", { "minScore": 0.9 }]
      }
    }
  }
}
```

---

## 14. Glossário de Software

| Termo | Definição |
|-------|-----------|
| **FSM** | Finite State Machine — máquina de estados finitos para controle de fluxo de jogo |
| **Observer Pattern** | Padrão onde objetos (subscribers) reagem a eventos de outros (publishers) |
| **Command Pattern** | Encapsula requisição como objeto, permitindo parametrização e undo |
| **Mesh** | Objeto 3D composto por geometria (vértices) e material (aparência) |
| **GSAP** | GreenSock Animation Platform — biblioteca de animação JavaScript |
| **HMR** | Hot Module Replacement — substituição de módulos em runtime sem reload |
| **TTI** | Time to Interactive — tempo até a página responder a interações |
| **FCP** | First Contentful Paint — tempo até primeiro conteúdo visível |
| **CSP** | Content Security Policy — política de segurança contra XSS |
| **PWA** | Progressive Web App — aplicação web com funcionalidades nativas |
| **LOD** | Level of Detail — técnica de redução de complexidade 3D por distância |
| **GC** | Garbage Collection — coleta de lixo da memória pelo JavaScript engine |

---

## 15. Apêndice: Diagrama de Sequência — Cobrança de Pênalti

```
Usuário    InputCtrl    MatchEng    Physics    AI(Goleiro)    UI    Audio    Scene
  │           │           │           │           │            │      │        │
  │──drag───►│           │           │           │            │      │        │
  │          │──aim────►│           │           │            │      │        │
  │          │          │           │           │            │      │        │
  │──release►│           │           │           │            │      │        │
  │          │──shoot──►│           │           │            │      │        │
  │          │          │           │           │            │      │        │
  │          │          │──calc────►│           │            │      │        │
  │          │          │◄──vel─────│           │            │      │        │
  │          │          │           │           │            │      │        │
  │          │          │──predict─►│           │            │      │        │
  │          │          │◄──pos─────│           │            │      │        │
  │          │          │           │           │            │      │        │
  │          │          │           │           │──moveGK──►│      │        │
  │          │          │           │           │            │      │        │
  │          │          │──animate─►│           │            │      │        │
  │          │          │           │           │            │      │        │
  │          │          │◄──frame───│           │            │      │        │
  │          │          │           │           │            │      │        │
  │          │          │──update──►│           │            │      │        │
  │          │          │           │           │            │      │        │
  │          │          │◄──pos─────│           │            │      │        │
  │          │          │           │           │            │      │        │
  │          │          │──check───►│           │            │      │        │
  │          │          │           │           │            │      │        │
  │          │          │◄──GOL!────│           │            │      │        │
  │          │          │           │           │            │      │        │
  │          │          │           │           │            │──►   │        │
  │          │          │           │           │            │      │──►     │
  │          │          │           │           │            │      │      │──►
  │          │          │           │           │            │      │      │
  │          │          │──replay──►│           │            │      │        │
  │          │          │           │           │            │      │        │
  │          │          │◄──2.5s────│           │            │      │        │
  │          │          │           │           │            │      │        │
  │          │          │──next────►│           │            │      │        │
```

---

*Documento aprovado. Revisões devem incrementar versão e registrar no changelog.*
