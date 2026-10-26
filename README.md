# Footy Skills

A set of Claude Code / OpenCode skills for scraping a football match report and turning it into an
independent, market-free prediction: scrape → specialist notes → a council of advocates → a decider →
an auditable final pick. No bookmaker price ever enters the analysis; the market is only shown
afterwards, as a reference-only scoreboard.

Everything here is Python standard library only except `football-scrape`, which needs `httpx` and
`playwright` (see its own `requirements.txt`).

## Install

Each folder below is a self-contained skill. To install one (or all of them):

```bash
git clone https://github.com/zetamins/Footy-Skills.git
cd Footy-Skills

# Claude Code
cp -r football-match-lite football-match-collection football-scrape football-*-advocate \
      football-team-news football-tactical-matchup football-underlying-stats \
      football-context-motivation football-media-scout \
      /path/to/your/project/.claude/skills/

# OpenCode (identical skills, same folders)
cp -r football-match-lite football-match-collection football-scrape football-*-advocate \
      football-team-news football-tactical-matchup football-underlying-stats \
      football-context-motivation football-media-scout \
      /path/to/your/project/.opencode/skills/
```

Copy only the skills you want — each one works independently. `football-match-lite` and
`football-match-collection` are the two entry points; everything else is a specialist or advocate
they call.

For `football-scrape`, also install its Python dependencies once:

```bash
pip install -r football-scrape/requirements.txt
playwright install chromium
```

## What's in here

### Entry points (start here)

| Skill | What it does |
|---|---|
| **football-scrape** | Scrapes ONE team's match report (fixtures, lineups, injuries, referee, odds, standings, form, xG, H2H) from up to 13 public sources. Run this first if you don't already have a report JSON. |
| **football-match-lite** | The lighter, single-skill analyst: reads one report JSON, sets its own expected goals, and recommends one bet with its reasoning. No trained model, no bookmaker price used. |
| **football-match-collection** | The full pipeline: runs the specialists and the council of advocates below, then a decider, with every step saved to disk so a crash or timeout loses nothing and the run resumes. |
| **football-match-analyst** | An alternative, independent-engine analyst (a trained national-team goals model, exact score distribution, draw-risk lens) for a quantitative second opinion. |

### Specialists (each reads one slice of the report)

| Skill | Covers |
|---|---|
| **football-team-news** | Injuries, suspensions, doubts, replacements, rotation, fitness, substitution habits. |
| **football-tactical-matchup** | Formations, pressing, build-up, defensive line, transitions, flanks, set pieces, weak areas. |
| **football-underlying-stats** | xG, shots, shot quality, chances conceded, luck vs. performance, the goalkeeper, ratings by line. |
| **football-context-motivation** | Motivation and stakes, home/away split, form and momentum, psychology, fatigue, schedule, head-to-head relevance. |
| **football-media-scout** | Pre-kickoff public headlines (expected XI, injuries, quotes, tactics, weather) with any price or tip stripped out. |

### The council of advocates (argue one case each, blind to each other and to the market)

| Skill | Argues for |
|---|---|
| **football-home-advocate** | The strongest honest case for the home side. |
| **football-away-advocate** | The strongest honest case for the away side. |
| **football-draw-advocate** | The strongest honest case that the match ends level. |
| **football-goals-advocate** | One direction in the goals market (over/under 2.5, both teams to score). |
| **football-underdog-advocate** | Whether the underdog scores, and if so, whether the favourite still wins, draws, or loses. |
| **football-gamestate-advocate** | How the match swings: first goal decides, a comeback, a swing, or a stalemate. |
| **football-chaos-advocate** | What the forecast cannot see: errors, a keeper's day, a set piece or penalty, cards — and whether it widens the draw or the underdog. Never moves the expected-goals number. |

A decider weighs every case and writes one final, checked answer.

## Design notes

- **Market-independent.** No step in the pipeline ever reads a bookmaker price. Reports are stripped of
  odds before any specialist or advocate sees them; the only place a price is read is a reference-only
  scoreboard printed after the final pick is already made.
- **Nothing is invented.** Every number an advocate or specialist cites must trace back to the report, a
  teammate's note, or its own evidence slice — a mechanical checker enforces this.
- **Confidence is evidence, not a price.** The expected-goals anchor moves only for confirmed team news
  (absences, lineups), never for form, luck, or a judgement call — tested against dozens of real
  matches before being locked in.
