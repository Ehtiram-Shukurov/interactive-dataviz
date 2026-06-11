# Summer Movies — A Data Visualization Journey

A collection of five data visualization assignments built for **CSCI 5609: Data Visualization** at the University of Minnesota. Each assignment explores a different technique, all centered on the same dataset: 899 summer movies released between 1945 and 2023.

**[Live Demo](https://ehtiram-shukurov.github.io/interactive-dataviz/)**

---

## Assignments

### A0 — Getting Started
A first look at Svelte 5's runes syntax. A small interactive click counter with a dropdown to set the limit — the goal was to get comfortable with reactive state and event handling before the real work began.

**Tools:** Svelte 5, TypeScript

---

### A1 — Summer Movies Dashboard
Three charts built with D3 exploring the full dataset:

- **Bar chart** — genre distribution across all movies, animated as you move through years
- **Bump chart** — how genre rankings shifted year over year from 1945 to 2023
- **Heatmap** — a co-occurrence matrix showing which genres most often appear together in the same film

**Tools:** D3.js, SVG, SvelteKit

---

### A2 — Interactive Scatter & Timeline
A scatter plot where you control the axes — switch between year, rating, runtime, and vote count on the fly. A brushable line chart below filters the scatter to a specific year range in real time, so you can zoom into any decade.

**Tools:** D3.js, Svelte 5 Runes, Brush interaction

---

### A3 — 3D Beach Visualization
Every summer movie represented as its own 3D shape rising from the sand, grouped into 5-year columns:

- 🟠 **Comedy** → Icosahedron
- 🔴 **Romance** → Torus Knot
- 🟢 **Drama** → Octahedron

The scene runs in a full beach environment with palm trees, an animated seagull, and a clapperboard. Hit Play to bring it to life.

**Tools:** Three.js, WebGL, D3.js, GLTFLoader

---

### A4 — Scrollytelling
A scroll-driven narrative visualization in two parts:

1. **2D scrolly** — a bar chart comparing each year's genre counts to 3 years prior as you scroll through every movie in the dataset
2. **3D scrolly** — the same beach scene from A3, but now scroll-controlled. Each era of summer cinema rises from the sand as you read through the story, with camera movement and decade-by-decade reveals tied directly to scroll position

**Tools:** Three.js, D3.js, Svelte 5 Runes, Scrollytelling

---

## Dataset

The dataset contains 899 summer movies from IMDb, spanning 1945 to 2023. Each entry includes title, year, genres, runtime, average rating, and vote count. It was the consistent thread across all five assignments — the same data, seen through five different lenses.

---

## Tech Stack

| Tool | Used for |
|---|---|
| SvelteKit | App framework and routing |
| Svelte 5 Runes | Reactive state (`$state`, `$derived`, `$effect`) |
| D3.js | Data processing and SVG chart rendering |
| Three.js | 3D scenes and WebGL rendering |
| TypeScript | Type safety throughout |

---

## Running Locally

```bash
npm install
npm run dev
```

Then open [http://localhost:5173](http://localhost:5173).

---

## Author

**Ehtiram (Ezra) Shukurov**  
CSCI 5609 — Data Visualization  
University of Minnesota
