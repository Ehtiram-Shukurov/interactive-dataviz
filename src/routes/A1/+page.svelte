<script lang="ts">
  import * as d3 from "d3";
  import { onMount } from "svelte";
  import type { TMovie } from "../../types";
  import Bar from "$lib/Bar.svelte";
  import BumpChart from "$lib/BumpChart.svelte";
  import HeatMap from "$lib/HeatMap.svelte";

  let movies: TMovie[] = [];

  async function loadCsv() {
    try {
      const csvUrl = "./summer_movies.csv";
      movies = await d3.csv(csvUrl, (row): TMovie => {
        return {
          tconst: row.tconst ?? "",
          title_type: row.title_type ?? "",
          primary_title: row.primary_title ?? "",
          original_title: row.original_title ?? "",
          year: new Date(`${row.year}-01-01`),
          runtime_minutes: Number(row.runtime_minutes),
          genres: row.genres ? row.genres.split(",") : [],
          simple_title: row.simple_title ?? "",
          average_rating: Number(row.average_rating),
          num_votes: Number(row.num_votes),
        };
      });
    } catch (error) {
      console.error("Error loading CSV:", error);
    }
  }

  onMount(loadCsv);
</script>

<div class="page">

  <header class="page-header">
    <span class="badge">A1</span>
    <h1>Summer Movies Dashboard</h1>
    <p class="subtitle">
      Three charts exploring 899 summer movies from 1945 to 2023 — how genres
      are distributed, how their rankings shifted year over year, and which
      genres most often appear together in the same film.
    </p>
    <p class="movie-count">
      {movies.length === 0 ? "Loading..." : `${movies.length} movies loaded`}
    </p>
  </header>

  <div class="charts">
    <section class="chart-section">
      <Bar {movies} />
    </section>

    <section class="chart-section">
      <BumpChart {movies} width={1200} height={520} />
    </section>

    <section class="chart-section">
      <HeatMap {movies} />
    </section>
  </div>

</div>

<style>
  .page {
    font-family: system-ui, -apple-system, sans-serif;
    max-width: 1280px;
    margin: 0 auto;
    padding: 3rem 2rem;
  }

  /* ── Page header ── */
  .page-header {
    margin-bottom: 3rem;
    padding-bottom: 2rem;
    border-bottom: 1px solid #dce8e4;
  }

  .badge {
    display: inline-block;
    font-size: 0.7rem;
    font-weight: 700;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    color: #2ab5a0;
    margin-bottom: 0.6rem;
  }

  h1 {
    font-family: Georgia, serif;
    font-size: 1.8rem;
    font-weight: normal;
    color: #111d2b;
    margin: 0 0 0.75rem;
  }

  .subtitle {
    font-size: 0.9rem;
    color: #5a7080;
    line-height: 1.65;
    margin: 0 0 0.75rem;
    max-width: 640px;
  }

  .movie-count {
    font-size: 0.78rem;
    color: #2ab5a0;
    margin: 0;
  }

  /* ── Charts ── */
  .charts {
    display: flex;
    flex-direction: column;
    gap: 3rem;
  }

  .chart-section {
    background: #fff;
    border: 1.5px solid #dce8e4;
    border-radius: 10px;
    padding: 1.5rem;
  }
</style>