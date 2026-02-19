<script lang="ts">
  import * as d3 from "d3";
  import type { TMovie } from "../types";

  type Props = {
    movies: TMovie[];
    width?: number;
    height?: number;
    maxGenres?: number;
  };

  let { movies, width = 1200, height = 520, maxGenres = 12 }: Props = $props();

  let selectedGenre: string = $state("");

  const margin = { top: 20, right: 260, bottom: 45, left: 55 };

  const usableArea = $derived({
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left
  });

  const midX = $derived((usableArea.left + usableArea.right) / 2);
  const midY = $derived((usableArea.top + usableArea.bottom) / 2);

  type Point = { year: number; date: Date; rank: number | null; count: number | null };

  function inc<K>(map: Map<K, number>, key: K, by = 1) {
    map.set(key, (map.get(key) ?? 0) + by);
  }

  function prep(movies: TMovie[], maxGenres: number) {
    // 1) count movies per (year, genre)
    const byYear = new Map<number, Map<string, number>>();

    for (const m of movies) {
      const y = m.year.getFullYear();
      if (!byYear.has(y)) byYear.set(y, new Map());

      const gmap = byYear.get(y)!;
      for (const g of m.genres) {
        if (g) inc(gmap, g);
      }
    }

    const years = Array.from(byYear.keys()).sort((a, b) => a - b);

    // 2) top3 per year + how often each genre appears in top3
    const top3 = new Map<number, { genre: string; count: number; rank: number }[]>();
    const appearCount = new Map<string, number>();

    for (const y of years) {
      const entries = Array.from(byYear.get(y)!.entries()).sort((a, b) => b[1] - a[1]);

      const top: { genre: string; count: number; rank: number }[] = [];
      for (let i = 0; i < Math.min(3, entries.length); i++) {
        const [genre, count] = entries[i];
        top.push({ genre, count, rank: i + 1 });
        inc(appearCount, genre);
      }

      top3.set(y, top);
    }

    // 3) keep the most frequent top-3 genres (so the chart stays readable)
    const genreList = Array.from(appearCount.entries())
      .sort((a, b) => b[1] - a[1] || a[0].localeCompare(b[0]))
      .slice(0, maxGenres)
      .map(([g]) => g);

    const series: Record<string, Point[]> = {};
    for (const g of genreList) {
      series[g] = years.map((y) => {
        const hit = (top3.get(y) ?? []).find((d) => d.genre === g);
        return {
          year: y,
          date: new Date(y, 0, 1),
          rank: hit?.rank ?? null,
          count: hit?.count ?? null
        };
      });
    }

    return { years, genreList, series };
  }

  const data = $derived(prep(movies, maxGenres));
  const years = $derived(data.years.map((y) => new Date(y, 0, 1)));
  const genres = $derived(data.genreList);
  const series = $derived(data.series);

  const xScale = $derived<d3.ScaleTime<number, number>>(
    d3
      .scaleTime()
      .domain(years.length ? (d3.extent(years) as [Date, Date]) : [new Date(), new Date()])
      .range([usableArea.left, usableArea.right])
  );

  const yScale = $derived<d3.ScaleLinear<number, number>>(
    d3.scaleLinear().domain([1, 3]).range([usableArea.top, usableArea.bottom])
  );

  const color = $derived(d3.scaleOrdinal<string, string>().domain(genres).range(d3.schemeTableau10));

  const line = $derived(
    d3
      .line<Point>()
      .defined((d) => d.rank !== null)
      .x((d) => xScale(d.date))
      .y((d) => yScale(d.rank ?? 1))
  );

  let xAxis: SVGGElement | undefined = $state();
  let yAxis: SVGGElement | undefined = $state();

  function updateAxis() {
    if (!xAxis || !yAxis) return;

    d3.select(xAxis).call(
      d3
        .axisBottom(xScale)
        .ticks(8)
        .tickFormat((d) => (d as Date).getFullYear().toString())
    );

    d3.select(yAxis).call(d3.axisLeft(yScale).tickValues([1, 2, 3]).tickFormat((d) => `#${d}`));
  }

  $effect(() => {
    updateAxis();
  });
</script>

<h3>Top 3 Genres by Year (Bump Chart)</h3>

{#if movies.length}
  <svg {width} {height}>
    {#each [1, 2, 3] as r}
      <line
        x1={usableArea.left}
        x2={usableArea.right}
        y1={yScale(r)}
        y2={yScale(r)}
        stroke="#999"
        stroke-dasharray="4 4"
        opacity="0.45"
      />
    {/each}

    {#each genres as g}
      {@const pts = series[g]}
      <g opacity={selectedGenre === "" || selectedGenre === g ? 1 : 0.12}>
        <path
          d={line(pts) ?? ""}
          fill="none"
          stroke={color(g)}
          stroke-width={selectedGenre === g ? 4 : 2.5}
          onmouseover={() => (selectedGenre = g)}
          onmouseout={() => (selectedGenre = "")}
          onclick={() => (selectedGenre = selectedGenre === g ? "" : g)}
        />
        {#each pts as p (p.year)}
          {#if p.rank !== null}
            <circle
              cx={xScale(p.date)}
              cy={yScale(p.rank)}
              r={selectedGenre === g ? 5 : 4}
              fill={color(g)}
              stroke="white"
              stroke-width="1.5"
              onmouseover={() => (selectedGenre = g)}
              onmouseout={() => (selectedGenre = "")}
            >
              <title>{g} — {p.year}: rank {p.rank}, {p.count} movies</title>
            </circle>
          {/if}
        {/each}
      </g>
    {/each}

    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />

    <text x={midX} y={height - 6} text-anchor="middle" font-size="12">Year</text>
    <text
      x="14"
      y={midY}
      text-anchor="middle"
      font-size="12"
      transform={`rotate(-90, 14, ${midY})`}
    >
      Rank (Top 3)
    </text>

    <g transform={`translate(${usableArea.right + 18}, ${usableArea.top})`}>
      <text class="label" x="0" y="-6" font-size="12">Genres (click)</text>
      {#each genres as g, i}
        <g
          transform={`translate(0, ${i * 18})`}
          style="cursor:pointer"
          opacity={selectedGenre === "" || selectedGenre === g ? 1 : 0.35}
          onclick={() => (selectedGenre = selectedGenre === g ? "" : g)}
        >
          <rect width="12" height="12" fill={color(g)} rx="2" />
          <text class="label" x="18" y="11" font-size="12">{g}</text>
        </g>
      {/each}
    </g>
  </svg>
{/if}

<style>
  .label {
    paint-order: stroke;
    stroke: white;
    stroke-width: 4px;
    stroke-linejoin: round;
  }
</style>
