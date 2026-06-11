<script lang="ts">
    import * as d3 from "d3";
    import { Scatter, Line } from "$lib";
    import { onMount } from "svelte";
    import type { TMovie } from "../../types";

    let movies: TMovie[] = $state([]);
    let yearRange: [Date, Date] | undefined = $state();

    function getYearCountArray(movies: TMovie[]) {
        let yearCount: { [year: number]: number } = {};
        const allYears = [...new Set(movies.map((d) => d.year.getFullYear()))];
        for (let year of allYears) {
            yearCount[year] = movies.filter(
                (d) => d.year.getFullYear() == year,
            ).length;
        }
        const yearCountArray = Object.entries(yearCount).map(([year, count]) => ({
            x: new Date(year),
            y: count as number,
        }));
        yearCountArray.sort((a, b) => (a.x < b.x ? -1 : 1));
        return yearCountArray;
    }

    let yearCountArray = $derived(getYearCountArray(movies));

    type TAxisSelection = {
        x: keyof TMovie;
        y: keyof TMovie;
        size: keyof TMovie;
    };

    let axisSelection: TAxisSelection = $state({
        x: "year",
        y: "average_rating",
        size: "num_votes",
    });

    async function loadCsv() {
        try {
            const csvUrl = "./summer_movies.csv";
            movies = await d3.csv(csvUrl, (row) => {
                return {
                    ...row,
                    num_votes: Number(row.num_votes),
                    runtime_minutes: Number(row.runtime_minutes),
                    genres: row.genres.split(","),
                    year: new Date(row.year),
                    average_rating: Number(row.average_rating),
                } as unknown as TMovie;
            });
        } catch (error) {
            console.error("Error loading CSV:", error);
        }
    }

    onMount(loadCsv);

    const attrOptionsX = $derived(movies[0] ? Object.keys(movies[0]) : []);
    const attrOptionsY = $derived(
        movies[0] ? Object.keys(movies[0]).filter((d) => d != "genres") : [],
    );
    const attrOptionsS = $derived(
        movies[0] ? Object.keys(movies[0]).filter((d) => d != "genres") : [],
    );
</script>

<div class="page">

    <header class="page-header">
        <span class="badge">A2</span>
        <h1>Interactive Scatter & Timeline</h1>
        <p class="subtitle">
            A scatter plot with user-controlled axes — switch between year, rating,
            runtime, and vote count on the fly. Brush the timeline below to filter
            the scatter to a specific year range in real time.
        </p>
        <p class="movie-count">
            {movies.length === 0 ? "Loading..." : `${movies.length} movies loaded`}
        </p>
    </header>

    {#if movies.length > 0}
        <div class="chart-card">
            <div class="selectors">
                <label class="selector-item">
                    X Axis
                    <select bind:value={axisSelection.x}>
                        {#each attrOptionsX as key}
                            <option value={key}>{key}</option>
                        {/each}
                    </select>
                </label>
                <label class="selector-item">
                    Y Axis
                    <select bind:value={axisSelection.y}>
                        {#each attrOptionsY as key}
                            <option value={key}>{key}</option>
                        {/each}
                    </select>
                </label>
                <label class="selector-item">
                    Size
                    <select bind:value={axisSelection.size}>
                        {#each attrOptionsS as key}
                            <option value={key}>{key}</option>
                        {/each}
                    </select>
                </label>
            </div>

            <Scatter
                movies={yearRange
                    ? movies.filter(
                          (d) => d.year <= yearRange![1] && d.year >= yearRange![0],
                      )
                    : movies}
                x={axisSelection.x}
                y={axisSelection.y}
                size={axisSelection.size}
            />
        </div>

        <div class="chart-card">
            <p class="chart-label">Movies per year — brush to filter the scatter above</p>
            <Line data={yearCountArray} bind:yearRange />
        </div>
    {/if}

</div>

<style>
    .page {
        font-family: system-ui, -apple-system, sans-serif;
        max-width: 900px;
        margin: 0 auto;
        padding: 3rem 2rem;
    }

    /* ── Page header ── */
    .page-header {
        margin-bottom: 2.5rem;
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
        max-width: 580px;
    }

    .movie-count {
        font-size: 0.78rem;
        color: #2ab5a0;
        margin: 0;
    }

    /* ── Charts ── */
    .chart-card {
        background: #fff;
        border: 1.5px solid #dce8e4;
        border-radius: 10px;
        padding: 1.5rem;
        margin-bottom: 1.5rem;
    }

    .chart-label {
        font-size: 0.78rem;
        color: #8aa0b4;
        margin: 0 0 0.75rem;
    }

    /* ── Selectors ── */
    .selectors {
        display: flex;
        gap: 1.5rem;
        flex-wrap: wrap;
        margin-bottom: 1.25rem;
    }

    .selector-item {
        display: flex;
        flex-direction: column;
        gap: 4px;
        font-size: 0.72rem;
        font-weight: 600;
        letter-spacing: 0.06em;
        text-transform: uppercase;
        color: #5a7080;
    }

    select {
        padding: 5px 10px;
        border: 1.5px solid #dce8e4;
        border-radius: 6px;
        font-size: 0.85rem;
        color: #1a2535;
        background: #fff;
        cursor: pointer;
        font-family: system-ui, sans-serif;
    }

    select:focus {
        outline: none;
        border-color: #2ab5a0;
    }
</style>