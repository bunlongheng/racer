<div align="center">
  <img src="docs/icon.png" alt="Racer" width="96" height="96" />
  <h1>Racer</h1>
  <p><em>A playful 3D marble race where 10 flag countries dodge Mario-Kart hazards around</em></p>
  <p><a href="https://racer-bheng.vercel.app">Live</a> &middot; <a href="https://github.com/bunlongheng/racer">Repo</a> &middot; <a href="https://bunlongheng.com/projects?name=racer">Portfolio</a></p>
  <img src="docs/social-preview.png" alt="Racer - preview" width="820" />
</div>

---

# Racer

> Pick a themed stage, set the laps, and watch 10 random countries race as glossy 3D flag marbles around an oval track - dodging hazards, jockeying the whole way - until a gold, silver, and bronze podium is crowned. A tiny, playful game for kids.

<p>
  <img alt="CI" src="https://github.com/bunlongheng/racer/actions/workflows/ci.yml/badge.svg" />
  <img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-brightgreen.svg" />
  <img alt="Next.js" src="https://img.shields.io/badge/Next.js-16-black?logo=nextdotjs" />
  <img alt="TypeScript" src="https://img.shields.io/badge/TypeScript-strict-3178c6?logo=typescript&logoColor=white" />
  <img alt="PWA" src="https://img.shields.io/badge/PWA-offline-5a0fc8" />
</p>

**Live:** https://racer-bheng.vercel.app

![A race in progress](docs/screenshots/race.png)

---

## What it does

- On the setup screen you pick a **stage**, the number of **laps** (rounds), and whether to show the **live results** board or let it **auto-play** hands-free.
- **10 random countries** line up as **true 3D glossy flag marbles** (WebGL) and race around a themed oval track.
- **8 themed stages** paint the whole map as their scene - Horse Race, Soccer Field, Football Field, Stadium, Airport, River Side, Beach Vibe, Snow Park - and the marble runs on the terrain.
- **8 mystery `?` boxes** sit fixed at the 3 and 9 o'clock sides; open one and it rolls a random effect - boost, mud, tar, banana, shrink, grow, or a big **fire** speed boost - then refills.
- Rare **bomb mines** drop onto the track - roll over one and you are knocked out (unless too few racers remain to spare one).
- A **live top-5 board** (top-left) and **1/2/3 medal badges** over the leaders track the standings in real time.
- After the set laps, the first 3 across the line take the **podium** - gold, silver, bronze - with confetti and a celebration fanfare.
- Each stage has its own **synth music** (starts on GO) and all sound effects are synthesised - **zero audio files**.

![The Winners podium](docs/screenshots/podium.png)

---

## How the race works

The race is a small, **pure, unit-tested core** driven by a single `requestAnimationFrame` loop. The 2D canvas draws the themed track + HUD; a transparent React-Three-Fiber layer renders the glossy 3D marbles over it.

```mermaid
flowchart LR
  A[10 random racers<br/>random form] --> B[step by real dt<br/>speed wobbles]
  B --> C{rolled over a<br/>? box or mine?}
  C -- yes --> D[box: random effect<br/>mine: knocked out]
  C -- no --> E[full speed]
  D --> F[separate<br/>no overlap]
  E --> F
  F --> G{crossed all laps?}
  G -- no --> B
  G -- yes --> H[assign place<br/>furthest first]
  H --> I{3 finished?}
  I -- no --> B
  I -- yes --> J[3s hold -> Winners]
```

Three ideas keep it fun **and** correct:

- **Deterministic overtaking** comes from a two-sine `wobble` per racer, so the pack shuffles the whole way with no real randomness in the physics (`lib/race.ts`).
- **No visible overlap** even on the tight bends: marbles are pushed apart in **real screen space**, then the push is split back into along-track and lateral motion via the local track frame (`lib/geometry.ts`).
- **The clock is real time**, so a lap always takes the same wall-clock span regardless of frame rate.

The engine and the geometry both live behind pure functions with tests (`tests/race.test.ts`, `tests/geometry.test.ts`).

---

## Architecture

| Module | Role |
| --- | --- |
| `app/page.tsx` + `layout.tsx` | Static server shell, metadata, font, PWA registration |
| `app/components/Race.tsx` | Canvas + rAF loop, HUD, and every screen (one client component) |
| `app/components/RaceMarbles.tsx` | React-Three-Fiber 3D marble overlay |
| `lib/race.ts` | Pure, dependency-free, unit-tested race engine |
| `lib/geometry.ts` | Pure oval geometry + real-space anti-overlap solver (tested) |
| `lib/sound.ts` | Web Audio synth - SFX + per-stage music, zero files |
| `app/data/stages.ts` | The 8 themed stages (painters + palettes) |
| `app/data/countries.ts` | 194 countries (code, name, accent hue) |
| `public/flags/` | 194 self-hosted flag PNGs |
| `public/sw.js` | Hand-written service worker for offline play |

Everything is **self-hosted** - the 194 flags, the font (`next/font`), and all audio (synthesised) - so there is no third-party CDN and the Content-Security-Policy stays locked to `'self'`.

## Design decisions and trade-offs

| Decision | Chosen | Alternative | Why | Cost we accept |
| --- | --- | --- | --- | --- |
| Marbles | True 3D (three.js / R3F) | 2D sprites | Genuinely glossy, rolling spheres | A WebGL layer over the canvas |
| Field size | 10 racers per race | All 194 at once | Readable, uncrowded track | Only 10 flags per race |
| Overlap | Real-space solver | Flat (u, lane) solver | Correct on the tight oval bends | ~20 relaxation passes/frame |
| Audio | Synthesised (Web Audio) | Audio files | No licensing, works offline, CSP `'self'` | Chiptune, not produced tracks |
| Flags | Self-hosted PNGs | Remote flag CDN | No third-party dependency | ~800KB committed |
| Offline | Hand-written SW | next-pwa plugin | Plugin breaks on Next 16 / Turbopack | Maintain ~40 lines of SW |

---

## Tech stack

- **Framework:** Next.js 16 (App Router, static prerender)
- **UI runtime:** React 19
- **3D:** three.js + @react-three/fiber + drei (WebGL marbles)
- **Track + HUD:** HTML5 Canvas 2D
- **Language:** TypeScript (strict)
- **Styling:** Tailwind CSS 4
- **Audio:** Web Audio API (synth)
- **Tests:** node:test (unit) + Playwright (e2e)
- **Hosting:** Vercel - PWA, offline-capable

---

## Quick start

```bash
git clone https://github.com/bunlongheng/racer.git
cd racer
npm install
npm run dev            # http://localhost:3030
```

No environment variables are required - it is a fully static, self-contained game.

### Scripts

| Command | What it does |
| --- | --- |
| `npm run dev` | Start the dev server on port 3030 |
| `npm run build` | Production build |
| `npm run typecheck` | `tsc --noEmit` |
| `npm run lint` | ESLint (typescript-eslint + next core-web-vitals) |
| `npm test` | Unit tests: race engine, geometry, and country data |
| `npm run test:e2e` | Playwright end-to-end race |

---

## Project layout

```
app/
  page.tsx                 server shell
  layout.tsx               metadata, font, PWA registration
  components/Race.tsx       canvas race loop + HUD + screens
  components/RaceMarbles.tsx  3D marble overlay (R3F)
  data/stages.ts            8 themed stages
  data/countries.ts         194 countries (code, name, accent hue)
lib/
  race.ts                  pure, tested race engine
  geometry.ts              pure oval math + anti-overlap solver (tested)
  sound.ts                 Web Audio synth (SFX + per-stage music)
public/
  flags/                   194 self-hosted flag PNGs
  sw.js                    offline service worker
tests/                     unit tests (race, geometry, data)
e2e/                       Playwright race test
```

---

## License

MIT - see [LICENSE](LICENSE).

---

<p align="center">
  <sub>Built by <a href="https://bunlongheng.com">Bunlong Heng</a> &middot; <a href="https://bunlongheng.com/projects/racer">See it in my portfolio &rarr;</a></sub>
</p>

---

<div align="center">

<a href="https://bunlongheng.com"><img src="https://img.shields.io/badge/-bunlongheng.com-3A3A3C?style=for-the-badge&amp;labelColor=2A2A2C&amp;logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABwAAAAcCAMAAABF0y%2BmAAADAFBMVEXx8vLq6v%2F19fX09PT8%2Ff309PT5%2BfnAwMBMaXHx8vL19fX5%2Bfny8vL09PT39%2Ff6%2Bfn6%2Bvr09PTx8fH4%2BPn%2F8vLy8%2FP09PTz8%2FPy8vL%2F%2F%2F%2Fz9PTz9PP19fby8vP19PXz8%2FT19%2Fb08%2FTx8vLy8vL09vby8%2FL29vf19fTv8PDz9vb7%2Bvn%2F%2Ff%2F9%2Ff3w8vH29fb%2F%2FP339%2Ff5%2Bfn59%2Ff19PT09PR7rao3VF3%2F%2Fv8AKDR6sqsBV1vv8fL4%2BPiux8b8%2Bvq90NDy8%2FR%2BlpppnZyZqrACLTxclZJSkI9en5ssZ2luqaUAeWgqbW4BX1gCfG0%2Fa3QaVG0MN0UKNUKUpKsGpYMAKjoRQ1UDf3AQRloJqIgEm32d0cUBg3ElbnQIln8eVWgrmIdP0KzF5d5s2acdfHwppJCP5MMXl4Tz8%2FLv8%2FP39%2FiewL5%2BqahYf4IrTFZkk5KlxsREiIYAHiqDnKFmnJowc3IdVllvpKK7zc3D2tnk5%2BmuycdwpKKuub2yv8Ly%2B%2FkAMT7z%2Bvjq7e09gH93q6kGdWpGgoJZmJSlw8M7c3WwyceTw76w1M8mYmpFfX8lW2AFOEpGiYhxqqRrqaRb2rdIsZ8AVE91jpU7d3t3saqBoKhako4AWVEcUlg3aWwDupAHb2YwmIkPP1ERSVA7WmZ8tq8HT14oZW%2BYuLsMl3wQT10ROksENUc8Z3MJSGVLh4dqpJ%2BXr7U51awUgHM4v6ad1MkPuJZAiowVd3IEhnZH0aoYrI05p5sDtYwkzZ8EPViYxsRel5USbHAIgmwIM0gQamzU4uF2malQi4oWQFQ1zaYrp4UXlYwfiHoyxqFE1q4qeX4yn5JIxKkdrX1b1rAbWGvd5Oh91sRijpomsJEVc3Ukc34dqIcwuJVKwZIXkoQZcXlNv5g%2BqZUZvpNl2rPG9N0WkIoNqouW4sVPwp%2BM0b6h0ssfq4d%2F1r8hpo%2Fh9e4AamUjnI9avaBixqgklYWSwL4xtItUxplYl5qZ58mq78wMgnYXfnlOmJaR1L%2Bj78ny9fS%2FrXQPAAAAFXRSTlP7Brvx%2FsJhAgD87r4U72C4uGH8vhRDodYHAAAACXBIWXMAAAsTAAALEwEAmpwYAAAC6UlEQVQokS2Sd3BUVRSHb0KS3QRCiZw5t%2By9j33zXjZkyUs2uekhEAi9dwRCL4JUEQRFAQUs9A4WOopKU0GpFoqCBZUivaqoFKkWLIS5GX7%2FfnPO78yZj%2Fj8UTViK6HggvNgSGuluEZRKbZqlN9H%2FFWiWUzIFQK5fhjOIRTDoqv4ia86S4q3KSByHdSqIhxofBKr5iNxLMm2bQqWVkorJ%2BAkO46nkQJhceSRmCTDvKJIJBLJzMxMLU6POJ4eBPF1KpNEbdt2Cms4LCcnZ%2FX6D3d%2BsKJJ7VSlNLoqgbjUtmkKKxlVmpf3bsFXW3d1b77gueJAQIPQBCilkMKemNi23Ts9T35%2FaMe2rsvmTypylGUFCSIYWJLXrm2nUwOPfNO5w%2B73Okxnjqe0JigQEL3JizoeONr%2F2u99vju4fd1rdZnyspQmnGvAUBprWNqz348%2FhXN%2F6P1%2By4Uvs%2FJkpRQRwpJBRVn7vf16Dfitz5XevQqWt3yalXtKaRIUQmglWfsu5y7%2BMrDs0TMftWo1ZVy64xioOVpcIdt8%2Fvqvf9%2F859KJsrLmrzA%2BOKC0IkFEtJza7JOfr%2FYd0P9G98tdOxe0Hp9qJrOIBgDuOeEtp%2F%2B4d%2FvWX%2F%2F9u%2B%2FtlaUTmOWZg7SQEEyuFe52%2BML9O3cZO%2F5xl06zn2VcadOJUqJTzt7a82XfP7sx9vmGNm1m1MstV4IHCVAAoTLqL1372ddn%2Fz%2B2v%2FWq12c1fSpsaeQWAZAANntxzuI3e3zbo8XGpm%2FMnNqiXrh%2BljaPd6UUabkjps19tdGnX2zKb9b4pTFDCzOMLmYtpW5h%2BpCRTeYtyV%2BT36zx808OL4xYGoFrkqDTbJpSXLdRgwbPZL%2BQnT129OOPFVEEsAMJpHKdeKMJgJSAiALdjJANQKnRJI4R6boAKEwLCiGoBBegQjBftQo1KQWOlplFKSV9qGaF1NpAE7QQESTlMSy6pp%2F4%2FFFVYxMNRBTaiM0tkImxNaL8vgfDR8gvoYRaxgAAAABJRU5ErkJggg%3D%3D" alt="bunlongheng.com"></a>
<a href="https://www.linkedin.com/in/bunlongheng/"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn"></a>
<a href="https://www.instagram.com/ibunlong/"><img src="https://img.shields.io/badge/Instagram-C13584?style=for-the-badge&logo=instagram&logoColor=white" alt="Instagram"></a>
<a href="mailto:bheng.code@gmail.com"><img src="https://img.shields.io/badge/Email-2E7D32?style=for-the-badge&logo=gmail&logoColor=white" alt="Email"></a>

<br>

Built by **[Bunlong](https://bunlongheng.com)** &nbsp;&middot;&nbsp; [more apps](https://bunlongheng.com/projects)

</div>
