<script lang="ts">
  import * as d3 from "d3";
  import type { TMovie } from "../types";

  type Props = {
    movies: TMovie[];
    width?: number;
    height?: number;
    maxGenres?: number;
  };

  let { movies, width = 900, height = 650, maxGenres = 12 }: Props = $props();

  let hoverRow: string = $state("");
  let hoverCol: string = $state("");

  const margin = { top: 20, right: 140, bottom: 90, left: 110 };

  const usableArea = $derived({
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left
  });

  const midX = $derived((usableArea.left + usableArea.right) / 2);
  const midY = $derived((usableArea.top + usableArea.bottom) / 2);

  type Cell = { row: string; col: string; value: number };

  function inc(map: Map<string, number>, key: string, by = 1) {
    map.set(key, (map.get(key) ?? 0) + by);
  }

  function getTopGenres(movies: TMovie[], maxGenres: number) {
    const freq = new Map<string, number>();
    for (const m of movies) {
      for (const g of m.genres) if (g) inc(freq, g);
    }
    return Array.from(freq.entries())
      .sort((a, b) => b[1] - a[1] || a[0].localeCompare(b[0]))
      .slice(0, maxGenres)
      .map(([g]) => g);
  }

  function getCells(movies: TMovie[], genres: string[]) {
    const set = new Set(genres);

    const counts = new Map<string, Map<string, number>>();
    for (const r of genres) {
      const row = new Map<string, number>();
      for (const c of genres) row.set(c, 0);
      counts.set(r, row);
    }

    for (const m of movies) {
      const gs = Array.from(new Set(m.genres.filter((g) => set.has(g))));
      for (let i = 0; i < gs.length; i++) {
        for (let j = i + 1; j < gs.length; j++) {
          const a = gs[i],
            b = gs[j];
          counts.get(a)!.set(b, (counts.get(a)!.get(b) ?? 0) + 1);
          counts.get(b)!.set(a, (counts.get(b)!.get(a) ?? 0) + 1);
        }
      }
    }

    const cells: Cell[] = [];
    for (const r of genres) {
      for (const c of genres) {
        const v = r === c ? 0 : (counts.get(r)!.get(c) ?? 0);
        cells.push({ row: r, col: c, value: v });
      }
    }
    return cells;
  }

  const genres = $derived(getTopGenres(movies, maxGenres));
  const cells = $derived(getCells(movies, genres));
  const maxVal = $derived(d3.max(cells, (d) => d.value) ?? 0);

  const xScale = $derived(
    d3
      .scaleBand()
      .domain(genres)
      .range([usableArea.left, usableArea.right])
      .padding(0.06)
  );

  const yScale = $derived(
    d3
      .scaleBand()
      .domain(genres)
      .range([usableArea.top, usableArea.bottom])
      .padding(0.06)
  );

  const colorScale = $derived(
    d3.scaleSequential(d3.interpolateBlues).domain([0, maxVal || 1])
  );

  let xAxis: SVGGElement | undefined = $state();
  let yAxis: SVGGElement | undefined = $state();

  function updateAxis() {
    if (!xAxis || !yAxis) return;

    d3.select(xAxis)
      .call(d3.axisBottom(xScale))
      .selectAll("text")
      .attr("transform", "rotate(-45)")
      .style("text-anchor", "end");

    d3.select(yAxis).call(d3.axisLeft(yScale));
  }

  $effect(() => {
    updateAxis();
  });
</script>

<h3>Genre Co-occurrence Heatmap</h3>

{#if movies.length}
  <svg {width} {height}>
    <defs>
      <linearGradient id="hmGrad" x1="0%" y1="0%" x2="100%" y2="0%">
        <stop offset="0%" stop-color={colorScale(0)} />
        <stop offset="100%" stop-color={colorScale(maxVal || 1)} />
      </linearGradient>
    </defs>

    {#each cells as d (d.row + "-" + d.col)}
      {@const x = xScale(d.col)}
      {@const y = yScale(d.row)}
      {#if x != null && y != null}
        <rect
          x={x}
          y={y}
          width={xScale.bandwidth()}
          height={yScale.bandwidth()}
          fill={d.row === d.col ? colorScale(maxVal || 1) : colorScale(d.value)}
          stroke="#fff"
          opacity={
            hoverRow === "" && hoverCol === ""
              ? 1
              : d.row === hoverRow || d.col === hoverCol
                ? 1
                : 0.15
          }
          onmouseover={() => {
            hoverRow = d.row;
            hoverCol = d.col;
          }}
          onmouseout={() => {
            hoverRow = "";
            hoverCol = "";
          }}
        >
          <title>
            {d.row} + {d.col}: {d.value}
          </title>
        </rect>
      {/if}
    {/each}

    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />

    <text x={midX} y={height - 6} text-anchor="middle" font-size="12">Genre</text>
    <text
      x="14"
      y={midY}
      text-anchor="middle"
      font-size="12"
      transform={`rotate(-90, 14, ${midY})`}
    >
      Genre
    </text>

    <g transform={`translate(${usableArea.right + 20}, ${usableArea.top + 10})`}>
      <text x="0" y="-6" font-size="12">Co-occurrence</text>
      <rect width="100" height="10" fill="url(#hmGrad)" stroke="#ccc" />
      <text x="0" y="26" font-size="11">Low</text>
      <text x="76" y="26" font-size="11">High</text>
    </g>
  </svg>
{/if}