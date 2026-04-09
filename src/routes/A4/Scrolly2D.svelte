<script lang="ts">
  import type { TMovie } from "../../types";
  import { Scroll, Bar } from "$lib";
  import * as d3 from "d3";

  type Props = {
    movies: TMovie[];
  };
  let { movies }: Props = $props();

  let myProgress = $state(0);

  let yearRange = $derived(d3.extent(movies.map((d) => d.year)));

  function getYearsBetweenDates(startDate?: Date, endDate?: Date) {
    if (startDate && endDate) {
      return Array.from(
        { length: endDate.getFullYear() - startDate.getFullYear() + 1 },
        (_, i) => new Date(startDate.getFullYear() + i, 0, 1),
      );
    }
    return [];
  }

  let years: Date[] = $derived(getYearsBetweenDates(yearRange[0], yearRange[1]));

  const barChartHeight = 500;
</script>

<h2>Genre Distribution Over Time</h2>
<p>
  Scroll through the years on the left to see how genre counts changed over time.
  The chart shows the current year alongside 3 years prior so you can compare directly.
</p>

<Scroll bind:progress={myProgress}>
  {#each years as date}
    <div class="year-block {date.getFullYear().toString()}">
      <h3 class="year">{date.getFullYear()}</h3>
      {#each movies.filter((d) => d.year.getFullYear() == date.getFullYear()) as movie}
        <b class="movie-title">{movie.primary_title}:</b>
        {movie.genres.join(" | ")}
        <br />
      {/each}
    </div>
  {/each}

  <div slot="viz">
    <Bar {movies} progress={myProgress} height={barChartHeight} width={600} />
  </div>
</Scroll>

<style>
  .movie-title {
    color: #449900;
  }
  h3.year {
    margin-bottom: 0px;
  }
  /* min-height keeps each year roughly the same scroll height so the
     chart on the right stays in sync with the left side content */
  .year-block {
    min-height: 200px;
  }
</style>