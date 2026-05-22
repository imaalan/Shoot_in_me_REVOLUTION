# ⚽ Shoot'em In: Apex Striker

> Releitura moderna do clássico jogo de navegador de cobranças de falta e pênalti — **Edição Copa do Mundo 2026**.

<p align="center">
  <img alt="HTML5" src="https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white">
  <img alt="JavaScript" src="https://img.shields.io/badge/JavaScript-F7DF1E?logo=javascript&logoColor=black">
  <img alt="Canvas" src="https://img.shields.io/badge/Canvas%202D-111111">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-007D48">
  <img alt="World Cup 2026" src="https://img.shields.io/badge/Copa%202026-FF5000">
</p>

Um arcade de precisão: você é o batedor, posicionado na falta/pênalti, e precisa dominar a física do chute para superar a barreira, enganar o goleiro e acertar as gavetas mais difíceis do gol. Sem zero‑a‑zero: **errar custa uma vida**, acertar o ângulo vale ouro.

---

## ✨ Funcionalidades

- **Modo Arcade Infinito** — 3 vidas, dificuldade que escala a cada 3 gols: surge a barreira → a barreira passa a pular → entra o vento dinâmico → o goleiro fica mais rápido, com mais alcance e antecipação.
- **Controle de estilingue** (arrastar e soltar) com **preview da trajetória** em tempo real e **efeito/curva** (Magnus) aplicado pelo *flick* lateral no momento de soltar.
- **Visão de trás do batedor** com **projeção em perspectiva real** (motor pensado em 3 eixos).
- **Edição Copa 2026** — escolha entre 12 seleções (Brasil em destaque + EUA/México/Canadá como sedes). A seleção pinta a bola, o rastro e o uniforme.
- **Sistema de pontuação por zonas**: meio `100` · gaveta `500` · ângulo `1000`, com **alvos bônus (x2)** e **multiplicador de combo**.
- **Feedback de estádio**: refletores, gramado com listras em perspectiva, rede projetada, *screen shake* e áudio sintetizado (WebAudio) com botão de mudo.
- **Touch‑first**: funciona com mouse e toque — pronto para futuro empacotamento mobile.

---

## 🎮 Como jogar

| Ação | Controle |
|------|----------|
| Mirar e dar força | **Arraste para trás** (estilingue). Quanto mais longo o arrasto, mais potência. |
| Conferir a mira | Siga a **linha pontilhada** (preview da trajetória) e o marcador no gol. |
| Curvar a bola | Dê um **flick lateral** ao soltar — essencial para contornar a barreira. |

**Pontuação**

- Meio do gol → **100**
- Gaveta (canto baixo) → **500**
- Ângulo (canto alto) → **1000**
- Acertar o alvo vermelho → **x2**
- Gols consecutivos aumentam o **combo** (multiplicador crescente).

Você tem **3 vidas**. Cada chute que não vira gol (para fora, na trave, na barreira ou defendido) custa uma vida.

---

## 🚀 Como rodar

O jogo é um **único arquivo HTML autossuficiente** — sem build, sem dependências.

```bash
# 1. Clone o repositório
git clone https://github.com/<seu-usuario>/shootem-in-apex-striker.git
cd shootem-in-apex-striker

# 2. Abra o index.html no navegador (duplo clique já funciona)
#    ou suba um servidor local:
python3 -m http.server 8000
# acesse http://localhost:8000
```

### Publicar online (GitHub Pages)

Como o `index.html` está na raiz, é só ativar o Pages:
`Settings → Pages → Source: Deploy from a branch → main / root`. O jogo fica disponível em `https://<seu-usuario>.github.io/shootem-in-apex-striker/`.

---

## 🧱 Arquitetura técnica

Apesar de ser renderizado em **Canvas 2D**, o jogo simula um espaço de **3 eixos** — projetado para um futuro port a 3D (Three.js) trocar apenas o renderizador, não a lógica.

- **Coordenadas de mundo** `x` (lateral) · `y` (altura) · `z` (profundidade rumo ao gol).
- **Projeção em perspectiva** tipo *pinhole*: pontos mais distantes (maior `z`) ficam menores e mais altos na tela, dando a clássica visão de trás do batedor.
- **Física em passo fixo** (`1/120s`) com gravidade, arrasto do ar, vento lateral e **efeito Magnus** (spin) — estável independente do FPS.
- **IA reativa do goleiro**: prevê o ponto de cruzamento da bola na linha (com erro que diminui conforme o nível), reage após um tempo de reação e tem alcance/antecipação escaláveis.
- **Colisões por travessia de plano**: barreira, linha do gol e zonas de pontuação são resolvidas no instante em que a bola cruza cada profundidade `z`.

```
Entrada (drag) ──► vetor de lançamento (vx, vy, vz, spin)
                     │
                     ▼
         Integrador de física (passo fixo)
                     │
        ┌────────────┼─────────────┐
        ▼            ▼             ▼
   Barreira      Goleiro       Linha do gol
                                   │
                                   ▼
                      Zonas + combo + alvo bônus ──► pontuação
```

---

## 🗺️ Roadmap

- [ ] **Modo Torneio mata‑mata** (Copa 2026) — caminho até a final.
- [ ] **Port para 3D** com Three.js, reaproveitando o motor x/y/z.
- [ ] **Leaderboard / recorde persistente** (back‑end ou armazenamento local fora do sandbox).
- [ ] **Empacotamento mobile** (Capacitor/Cordova) para Android e iOS.
- [ ] Loja de cosméticos (skins de bola, rastros, luvas).

---

## 🎨 Design system

A identidade visual das telas e do HUD aplica o sistema de design **inspirado na Nike**: UI monocromática (`#111111` / branco / cinzas), tipografia display condensada em caixa alta (Oswald como substituto livre da Nike Futura ND, *line-height* 0.90), corpo Helvetica peso 500, botões *pill* (30px) e elevação plana — deixando a cor vir do "produto" (gramado, bola e cores da seleção), com acento expressivo único **Orange Flash `#FF5000`**.

Tokens extraídos do catálogo [**awesome-design-md**](https://github.com/VoltAgent/awesome-design-md) (MIT).

---

## 📁 Estrutura do projeto

```
shootem-in-apex-striker/
├── index.html      # Jogo completo (HTML + CSS + JS em um arquivo)
├── README.md
├── LICENSE
└── .gitignore
```

---

## 📄 Licença

Distribuído sob a licença **MIT**. Veja [`LICENSE`](LICENSE) para detalhes.

## 👤 Autor

**Alan de Jesus Lima** — desenvolvimento e design.

> Projeto de portfólio. Contribuições, *issues* e *pull requests* são bem‑vindos.
