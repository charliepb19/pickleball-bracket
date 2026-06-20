# 🏓 Pickleball Tournament

A single-file browser app that simulates a pickleball tournament for a group of friends. Pick your player, get paired up, and run through a full bracket — or go 1v1 in World Cup mode with group stages and a knockout round.

No frameworks, no build tools, no dependencies. Just one `index.html` file you open in a browser.

---

## Modes

### 2v2 Tournament
- 32 players are randomly paired into 16 teams
- You pick your player and get assigned a random teammate
- Teams are seeded 1–16 by combined average rating
- Single-elimination bracket: Round of 16 → Quarterfinals → Semifinals → Final
- Tournament MVP and Finals MVP are awarded based on simulated per-player stats

### World Cup Mode (1v1)
- All 32 players are randomly split into 8 groups of 4
- Round-robin group stage: every player faces the other 3 in their group (6 matches per group)
- Points are earned per **set won** — a 2-0 win earns 2 points, a 2-1 win still earns 2 points, a 2-1 loss earns 1 point
- Group standings tiebreakers (in order): sets won → points scored → fewest points conceded
- Top 2 from each group advance — 16 qualifiers total
- Qualifiers are seeded 1–16 using the same tiebreaker criteria and placed into a knockout bracket
- Knockout runs: Round of 16 → Quarterfinals → Semifinals → Final

---

## How the simulation works

### Win probability
Each match uses an Elo-style formula to calculate the probability that a given player/team wins any individual rally:

```
P(A wins rally) = 1 / (1 + 10^((ratingB - ratingA) / 30))
```

The divisor of 30 controls how much the rating gap matters. A 30-point difference gives roughly a 73% win probability — close enough that upsets happen regularly.

### Match format
Each match is best-of-2 sets, with a tiebreak third set played if the first two are split 1-1. Sets are played to 11, win by 2 (no cap — sets can go 15-13, etc.).

Each rally inside a set is resolved by drawing a random number against the win probability. This means the simulation plays out naturally rally by rally rather than jumping straight to a result.

### 2v2 point attribution
In 2v2, individual player stats (points scored, aces) are tracked per rally. When a team wins a rally, the point is credited to one of the two players using a weighted split:

```
ratio = clamp(p1.rating / (p1.rating + p2.rating) + noise, 0.2, 0.8)
scorer = random() < ratio ? p1 : p2
```

A noise term of ±0.125 is added each rally so lower-rated teammates can still have standout performances. Aces are applied with a ~9% probability on any point scored.

### Seeding
The bracket uses a recursive seeding function that produces the standard tournament layout (1v16, 2v15, 8v9, etc.):

```javascript
function tournamentSeedOrder(n) {
  if (n === 2) return [0, 1];
  const top = tournamentSeedOrder(n / 2);
  const result = [];
  for (const s of top) result.push(s, n - 1 - s);
  return result;
}
```

### Upset detection
After each match, the result is flagged as an upset if the winner had a higher seed number (i.e. was the lower-ranked entrant). These get an **UPSET** badge on the bracket and in the match recap toast.

---

## Features

- **Win probabilities** shown on every unplayed match in the bracket
- **Match recap toast** after each result with a generated one-liner describing the match
- **Upset indicators** on the score line when a lower seed wins
- **Pause / resume** on auto-sim
- **Adjustable sim speed** via a slider
- **MVP + Finals MVP** awards with a full stat leaderboard at the end

---

## Running it

Just open `index.html` in any modern browser. No server needed.

```bash
# or from the terminal
start index.html       # Windows
open index.html        # Mac
```

---

## Customizing players

The player roster lives in the `PLAYERS` array near the top of the `<script>` block in `index.html`. Each entry is just a name and a rating (1–99):

```javascript
const PLAYERS = [
  { name: "Vernon",       rating: 99 },
  { name: "Robert Ward",  rating: 97 },
  // ...
];
```

Ratings are relative — what matters is the gap between players, not the absolute value.

---

## Tech

- Vanilla HTML/CSS/JS — no frameworks, no build step
- All state lives in module-level JS variables
- CSS custom properties for theming (`--accent`, `--surface`, etc.)
- `localStorage` not currently used — each page load starts fresh
