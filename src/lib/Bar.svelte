<script lang="ts">
  import type { TMovie } from "../types";
  import * as d3 from "d3";

  type Props = {
    movies: TMovie[];
    progress?: number;
    width?: number;
    height?: number;
  };
  let { movies, progress = 100, width = 500, height = 400 }: Props = $props();

  let selectedGenre: string = $state("");

  const yearRange = $derived(d3.extent(movies.map((d) => d.year)));

  function getUpYear(yearRange: [undefined, undefined] | [Date, Date]) {
    if (!yearRange[0]) return new Date();
    const timeScale = d3.scaleTime().domain(yearRange).range([0, 100]);
    return timeScale.invert(progress);
  }
  const upYear: Date = $derived(getUpYear(yearRange!));

  const ghostYear: Date = $derived(
    new Date(upYear.getFullYear() - 3, upYear.getMonth(), upYear.getDate())
  );

  function getGenreNums(movies: TMovie[], upYear: Date) {
    let res: { [genre: string]: number } = {};
    movies
      .filter((movie) => movie.year <= upYear)
      .forEach((movie) => {
        movie.genres.forEach((genre: string) => {
          res[genre] = (res[genre] || 0) + 1;
        });
      });
    return res;
  }

  const genreNums = $derived(getGenreNums(movies, upYear));
  const ghostGenreNums = $derived(getGenreNums(movies, ghostYear));

  const fullGenreNums = $derived(getGenreNums(movies, yearRange[1] ?? new Date()));

  const margin = { top: 15, bottom: 50, left: 30, right: 10 };

  const usableArea = $derived({
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left,
  });

  const xScale = $derived(
    d3.scaleBand()
    .domain(Object.keys(fullGenreNums))
    .range([usableArea.left, usableArea.right])
    .padding(0.1)
  );

  const yScale = $derived(
    d3.scaleLinear()
    .range([usableArea.bottom, usableArea.top])
    .domain([0, d3.max(Object.values(fullGenreNums)) ?? 0])
  );

  const xBarwidth: number = $derived(xScale.bandwidth());

  let xAxis: any = $state(), yAxis: any = $state();

  function updateAxis() {
    d3.select(xAxis)
      .call(d3.axisBottom(xScale))
      .selectAll("text")
      .attr("transform", "rotate(45)")
      .style("text-anchor", "start");
    d3.select(yAxis).call(d3.axisLeft(yScale));
  }

  $effect(() => {
    updateAxis();
  });
</script>

<h3>
  The Distribution of Genres {yearRange[0]?.getFullYear()} - {yearRange[1]?.getFullYear()}
</h3>

{#if movies.length > 0}
  <svg {width} {height}>
    <g class="bars">
      {#each Object.keys(fullGenreNums) as genre}
        {@const cnt = genreNums[genre] ?? 0}
        {@const ghostCnt = ghostGenreNums[genre] ?? 0}
        {@const halfWidth = xBarwidth / 2}
        <g class={genre}>
          <rect
            width={halfWidth}
            height={usableArea.bottom - yScale(ghostCnt)}
            x={xScale(genre)!}
            y={yScale(ghostCnt)}
            fill="#cccccc"
            class="bar"
            opacity={selectedGenre === "" || selectedGenre === genre ? 1 : 0.2}
          />
          <rect
            role="button"
            tabindex="0"
            width={halfWidth}
            height={usableArea.bottom - yScale(cnt)}
            x={xScale(genre)! + halfWidth}
            y={yScale(cnt)}
            fill="#449900"
            class="bar"
            opacity={selectedGenre === "" || selectedGenre === genre ? 1 : 0.2}
            onmouseover={() => { selectedGenre = genre; }}
            onmouseout={() => { selectedGenre = ""; }}
            onfocus={() => { selectedGenre = genre; }}
            onblur={() => { selectedGenre = ""; }}
          />
          <text
            x={xScale(genre)! + xBarwidth * 0.75}
            y={yScale(cnt) - 5}
            font-size="12"
            text-anchor="middle"
          >
            {selectedGenre === genre ? cnt : ""}
          </text>
        </g>
      {/each}
    </g>

    <g transform="translate({usableArea.right - 130}, {usableArea.top + 5})">
      <rect width="12" height="12" fill="#cccccc" />
      <text x="16" y="11" font-size="11" fill="#555">3 years prior ({ghostYear.getFullYear()})</text>
      <rect width="12" height="12" fill="#449900" y="18" />
      <text x="16" y="29" font-size="11" fill="#555">Current ({upYear.getFullYear()})</text>
    </g>

    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
  </svg>
{/if}

<style>
  .bar {
    transition:
      y 0.1s ease,
      height 0.1s ease,
      width 0.1s ease;
  }
</style>