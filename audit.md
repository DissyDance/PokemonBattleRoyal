# PokéTFT — Audit d'état des lieux
> Date : 2026-06-14 · Branche cible : `feature/dissydance` · Repo : https://github.com/Haxxxxxx/PokeTFT

---

## 1. Vue d'ensemble

| Dimension | État |
|---|---|
| Stack | Next.js 16 App Router · React 19 · TypeScript · Tailwind v4 · Zustand 5 · @dnd-kit · PixiJS 8 |
| Avancement | Phase 3 ✅ (solo vs 7 IA) · Phase 3b–5 ❌ |
| Multiplayer | Inexistant — single player (joueur vs 7 IA simulés) |
| Lobby pré-combat | Inexistant |
| Items | Hooks présents (`UnitInstance.items: string[]`) mais 0 effet implémenté |
| Générations | Gen 1 Kanto uniquement (~30 familles, ~35 UnitDef) |
| Combat | Moteur déterministe complet (hex pathing, moves, type chart) ✅ |

---

## 1b. Branches remote

| Branche | Commits | Contenu |
|---|---|---|
| `main` | Phase 3 complète | Tout le code de jeu |
| `feature/dissydance` | 1 derrière main | Scaffold Next.js vide (0 code de jeu) |

---

## 2. Architecture actuelle (branche `main`)

```
src/
  game/
    config.ts          ← constantes TFT (éco, shop odds, XP, board geometry, damage par stage)
    types.ts           ← UnitDef, UnitInstance, StatBlock, Move, TraitDef…
    ui.ts              ← couleurs cost/type
    data/
      mons.ts          ← ~35 pokémon Gen 1 (coût 1→5, familles d'évolution = étoiles)
      traits.ts        ← synergies types (Fire/Water/Electric/Grass/Psychic…) + rôles
      typeChart.ts     ← matrice 18×18 d'efficacité de types
    engine/            ← TS pur, sans React, unit-testable
      rng.ts           ← PRNG déterministe seedable
      shop.ts          ← pool partagé + roll shop par level (odds dans config.ts)
      economy.ts       ← intérêts, streak, valeur de vente
      combine.ts       ← fusion 3→⭐⭐ (9→⭐⭐⭐) + makeInstance
      synergies.ts     ← calcul traits actifs
      combat.ts        ← moteur combat déterministe (hex pathing, moves, type chart)
      hex.ts           ← géométrie hexagonale
      enemy.ts         ← génération boards IA (scale level + boardCount)
    store/
      gameStore.ts     ← Zustand : gold/xp/level/health/streak/stage/round/units/shop
      lobbyStore.ts    ← 7 rivaux IA (Brock…Blaine), AI-vs-AI, HP ladder, spectate
      combatStore.ts   ← état combat en cours (result, opponentName, opponentId)
      flow.ts          ← startCombatFlow / resolveCombatFlow orchestrent les 3 stores
      uiStore.ts       ← viewPlayerId (spectate)
  components/game/
    GameClient.tsx     ← root DnD context, timer planification 30s, layout principal
    Board.tsx          ← grille hexagonale 7×4 (interactive ou spectate)
    Bench.tsx          ← banc 9 slots (droppable)
    ShopBar.tsx        ← 5 slots shop + reroll + freeze + buy XP
    TopBar.tsx         ← or, santé, XP/level, stage/round
    TraitPanel.tsx     ← synergies actives (breakpoints visuels)
    UnitDetail.tsx     ← panneau détail unité sélectionnée (stats, move, items)
    UnitChip.tsx       ← composant unité (draggable, étoiles, icône move)
    Scoreboard.tsx     ← classement 8 joueurs + HP + bouton spectate
    CombatStage.tsx    ← visualisation combat animée (PixiJS)
    icons.tsx          ← icônes SVG inline
  app/
    page.tsx           ← shell HTML + header → <GameClient />
    globals.css        ← Tailwind base + reset
    layout.tsx         ← root layout Next.js
```

### 2b. Features implémentées (main)

| Feature | Statut |
|---|---|
| Shop 5 slots, odds par level, pool partagé | ✅ |
| Économie (intérêts, streak, revente) | ✅ |
| XP / montée niveau 1→10 | ✅ |
| Roster Gen 1 (~30 familles) | ✅ |
| Auto-évolution 3→⭐⭐, 9→⭐⭐⭐ | ✅ |
| Synergies de types actives | ✅ |
| Plateau hex 7×4 + drag-and-drop | ✅ |
| Moteur combat déterministe + replay | ✅ |
| 7 rivaux IA nommés (gym leaders Kanto) | ✅ |
| Rounds AI-vs-AI off-screen | ✅ |
| HP ladder + élimination | ✅ |
| Spectate board IA | ✅ |
| Timer planification 30s | ✅ |
| Game over screen + replay | ✅ |
| Sprites PokéAPI par id dex national | ✅ |
| Items / objets (effets) | ❌ hooks seulement |
| Lobby multi-joueurs | ❌ |
| Sélection de génération | ❌ |
| Auth / sessions | ❌ |

---

## 3. État de la branche `feature/dissydance`

- SHA : `e5564e5` — **1 commit derrière `main`** — aucun changement, branche presque vide
- Fichiers présents : scaffold Next.js uniquement (layout, globals.css, page.tsx minimal, configs)
- **0 fichier de code de jeu** (pas de `src/game/`, pas de `src/components/game/`)
- Point de départ propre pour le lobby — les fichiers de jeu seront mergés depuis main

---

## 4. Projet local (worktree `claude/focused-ellis-e44116` — PokeBrawl)

Projet **distinct** de PokéTFT dans ce worktree local. À ne pas confondre.

| Caractéristique | Valeur |
|---|---|
| Fichiers | 1 seul `index.html` (4385 lignes) |
| Stack | HTML/CSS/JS vanilla — aucune dépendance |
| Type de jeu | Battle Royale tour-par-tour (2-5 joueurs **locaux**) |
| Génération | Gen 1-7 + Gen 8-9 en arrière-plan (PokeAPI GraphQL) |
| IA | Toggle par joueur, équipe auto-générée |
| Bots diffs | Non (binaire : IA ou humain) |
| Fusion | Oui (slot 7 = réserve fusion) |
| Méga-évolutions | Oui |
| Shinies | Oui (35% chance, +10% stats) |
| Météo | Oui (4 types, rotation) |
| Items | 5 (Restes, Bandeau Choix, Lunettes Choix, Orbe Vie, Ceinture Force) |
| Draft de pouvoirs | Oui (pouvoirs spéciaux, 1 par joueur avant combat) |
| Lobby en ligne | Non — local uniquement |

**Design system PokeBrawl** (CSS variables utilisées dans index.html) :
```css
--bg: #1a1a2e  --bg-2: #16213e  --bg-3: #0f1626
--accent: #e94560  --accent-2: #f5a623
--txt: #f1f1f1  --muted: #9aa3b2
--ok: #2ecc71  --bad: #e74c3c  --border: #2a2f4a
Fonts : Orbitron (titres pixel) + Press Start 2P + system-ui
```

---

## 5. Ce qui manque pour le lobby pré-combat (feature/dissydance)

### 5.1 Gestion des joueurs
- `lobbyStore.ts` (main) gère 7 IA fixes, pas de joueurs humains réels
- Pas de lobby code / partage de partie
- Pas de slots joueurs 1–8 configurables
- Pas de niveaux de difficulté IA (Easy/Medium/Hard)
- Pas de statut de connexion par slot

### 5.2 Règles de partie configurables
- `config.ts` = constantes fixes (HP = 100, pas de paramètre host)
- Aucun sélecteur de génération (Gen 1 hardcodé dans `mons.ts`)
- Items : hooks présents (`UnitInstance.items: string[]`) mais liste vide, 0 effet
- Aucun paramètre modifiable avant la partie

### 5.3 Réseau / multijoueur
- Aucune couche réseau — 100% local
- Phase 5 roadmap = "server-authoritative online" (lointain)
- Pas de WebSocket, pas de Supabase Realtime, pas de PartyKit
- Lobby code = stub à implémenter (génération côté client ok pour Phase 3b)

### 5.4 Données Pokémon multi-gen
- `mons.ts` = ~35 Pokémon Gen 1 uniquement (hardcodé)
- PokeAPI supporte 1025 Pokémon (Gen 1–9) + endpoint `/generation/{id}`
- Aucun système de draft par génération

---

## 6. Direction artistique (à respecter — conventions PokéTFT main)

| Token | Valeur observée dans le code |
|---|---|
| Fond global | `bg-[#0b1020]` (dark navy profond) |
| Texte | `text-slate-100` |
| Accent primaire | `text-rose-500` / `border-rose-*` (rouge Pokéball) |
| Accent secondaire | `text-amber-400` (or) |
| Bordures | `border-slate-800` (header) · `border-slate-700` (composants) |
| Panels | `bg-slate-900/60` + `backdrop-blur` |
| Boutons primaires | `bg-amber-500 hover:bg-amber-400 text-black` |
| Boutons secondaires | `bg-sky-700 hover:bg-sky-600` |
| Danger / rouge | `bg-rose-500/20 border-rose-400 text-rose-200` |
| Fonts | system-ui + Tailwind defaults (pas de police custom) |
| Sprites | `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/{dex}.png` |
| Icônes | SVG inline dans `icons.tsx` |
| Layout | max-w-[1440px] mx-auto p-4, flex column, gap-3 |

---

## 7. Plan d'implémentation — Interface Lobby Pré-Combat (`feature/dissydance`)

### Feature cible : Interface Lobby Pré-Combat

#### 7.1 Nouveaux fichiers à créer

```
src/
  game/
    store/
      preLobbyStore.ts        ← Zustand : config lobby (slots, règles, code)
    data/
      generations.ts          ← mapping Gen 1-9 → ranges dex nationaux
      itemPool.ts             ← liste d'items disponibles (à activer/désactiver)
  components/lobby/
    LobbyScreen.tsx           ← écran principal lobby (8 slots + panneau règles)
    PlayerSlot.tsx            ← card slot joueur (humain / bot + difficulté)
    LobbyCodeBadge.tsx        ← code de lobby + copie clipboard
    GameRulesPanel.tsx        ← sélection génération + HP + items
    GenerationPicker.tsx      ← picker multi-gen (checkbox par gen)
    ItemPoolEditor.tsx        ← toggle items disponibles
```

#### 7.2 Modèle de données (preLobbyStore)

```typescript
type PlayerSlot = {
  id: string;                            // "slot-1" … "slot-8"
  type: "empty" | "human" | "bot";
  name: string;
  botDifficulty?: "easy" | "medium" | "hard";
  status: "waiting" | "ready" | "connected";
};

type GameRules = {
  generations: number[];                 // [1] | [1,2] | … | [1..9]
  draftPoolSize: number;                 // nb Pokémon tirés (ex: 150 parmi le sous-ensemble)
  startingHp: number;                    // défaut: 100 (ECONOMY.startingHealth)
  itemsEnabled: string[];                // ids items actifs
  maxPlayers: 2 | 3 | 4 | 5 | 6 | 7 | 8;
};

type PreLobbyState = {
  lobbyCode: string;                     // 6 chars alphanum, ex: "K9XP4M"
  isHost: boolean;
  slots: PlayerSlot[];                   // toujours 8 éléments
  rules: GameRules;
  phase: "lobby" | "starting";
  // actions
  generateCode: () => void;
  setSlot: (id: string, update: Partial<PlayerSlot>) => void;
  addBot: (slotId: string, difficulty: "easy"|"medium"|"hard") => void;
  removeSlot: (slotId: string) => void;
  setRules: (update: Partial<GameRules>) => void;
  startGame: () => void;
};
```

#### 7.3 Gestion multi-gen

```typescript
// src/game/data/generations.ts
export const GEN_DEX_RANGES: Record<number, [number, number]> = {
  1: [1, 151],
  2: [152, 251],
  3: [252, 386],
  4: [387, 493],
  5: [494, 649],
  6: [650, 721],
  7: [722, 809],
  8: [810, 905],
  9: [906, 1025],
};

// Draft : tirer N pokémon parmi le sous-ensemble des gens sélectionnées
// → utiliser rng.ts déjà présent pour seed déterministe partagé via code lobby
```

#### 7.4 Items disponibles

```typescript
// src/game/data/itemPool.ts
// Reprend les 5 de PokeBrawl + extension possible Phase 4
export const ITEM_POOL = [
  { id: "leftovers",    name: "Restes",         effect: "Régénère 5% PV/tour" },
  { id: "choice-band",  name: "Bandeau Choix",  effect: "+50% Attaque, 1 seule capacité" },
  { id: "choice-specs", name: "Lunettes Choix", effect: "+50% ATS, 1 seule capacité" },
  { id: "life-orb",     name: "Orbe Vie",       effect: "+30% dégâts, -10% PV/attaque" },
  { id: "focus-sash",   name: "Ceinture Force", effect: "Survit à 1 PV si PV pleins" },
];
```

#### 7.5 Connexion réseau

- **Phase 3b (immédiate)** : lobby local + bots simulés, lobby code = décoration (partage manuel)
- **Phase 5** : Supabase Realtime ou PartyKit pour sync multi-device

---

## 8. Ordre d'implémentation

1. `preLobbyStore.ts` — state complet avant l'UI
2. `generations.ts` + `itemPool.ts` — données
3. `LobbyScreen.tsx` — layout + 8 slots
4. `PlayerSlot.tsx` — card avec toggle humain/bot + sélecteur difficulté
5. `GameRulesPanel.tsx` — génération + HP + items
6. `LobbyCodeBadge.tsx` — code 6 chars + copie clipboard
7. Wiring `page.tsx` — entrée par lobby → jeu

---

## 9. Points de vigilance

| Sujet | Détail |
|---|---|
| Tailwind v4 | CSS-first, pas de `tailwind.config.js` — utiliser classes utilitaires directement |
| Next.js 16 App Router | `"use client"` obligatoire pour les composants avec state/events |
| `ECONOMY.startingHealth` | Valeur fixe 100 dans `config.ts` — exposer via `GameRules.startingHp` |
| `mons.ts` hardcodé Gen 1 | Nécessite un refactor pour multi-gen (draft depuis pool dynamique) |
| PokeAPI rate limit | Pas de clé API, mais cache les fetches génération côté client |
| Seed déterministe | Utiliser `rng.ts` + lobby code pour que tous les joueurs aient le même draft |
| `lobbyStore.ts` (main) | Store existant gère les 7 IA fixes — ne pas confondre avec `preLobbyStore` |
