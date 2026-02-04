<script>
    let maxClick = 2;
    let cnt = maxClick; // tip: https://svelte.dev/docs/svelte/$state
    
    function onClick() {
      if (cnt > 0) cnt -= 1;
    }
</script>
  
<h1>[Your Name]'s VIS Site</h1>

<img
  width="200px"
  src="url to your favorite image"
/>

<div>
  You can click up to
  <select 
    bind:value={maxClick}
    on:change={() => (cnt = maxClick)}
  >
    {#each [2, 4, 6] as optionNum}
      <option value={optionNum}>
        {optionNum}
      </option>
    {/each}
  </select>
  times
</div>

<button on:click={onClick}> Click Me </button>

<!-- tip: use {#if...} template syntax here (https://svelte.dev/docs/svelte/if) so that the content below will automatically update when cnt value changes -->
<!-- `the content below should only show when cnt >0` -->
{#if cnt > 0}
  <p id="info">Remaining Number of Clicks: {cnt}</p>
{/if}

<!-- The content below should only shown when cnt =0 -->
{#if cnt === 0}
  <p>No more clicks allowed</p>
{/if}
  
<style>
  body {
    font-family: Arial, Helvetica, sans-serif;
  }
  button {
    background-color: #44aa66;
    /* background-color: blue; */
    color: white;
    font-size: xx-large;
    padding: 10px 20px;
    border: none;
    cursor: pointer;
    border-radius: 5px;
  }
</style>
