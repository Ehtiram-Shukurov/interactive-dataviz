<script>
    let maxClick = 5;
    let cnt = maxClick;
    
    function onClick() {
        if (cnt > 0) cnt -= 1;
    }
</script>

<div class="page">

    <header class="page-header">
        <span class="badge">A0</span>
        <h1>Getting Started with Svelte</h1>
        <p class="subtitle">
            A first look at Svelte's component model — reactive state, event handling,
            and conditional rendering, all in one small interactive demo.
        </p>
    </header>

    <div class="demo">
        <img class="demo-img" width="180" src="image.jpg" alt="My Image" />

        <div class="controls">
            <label class="control-row">
                You can click up to
                <select bind:value={maxClick} on:change={() => (cnt = maxClick)}>
                    {#each [2, 4, 5, 6] as optionNum}
                        <option value={optionNum}>{optionNum}</option>
                    {/each}
                </select>
                times
            </label>

            <button on:click={onClick}>Click Me</button>

            {#if cnt > 0}
                <p class="info">Remaining clicks: <strong>{cnt}</strong></p>
            {:else}
                <p class="info exhausted">No more clicks allowed</p>
            {/if}
        </div>
    </div>

</div>

<style>
    .page {
        font-family: system-ui, -apple-system, sans-serif;
        max-width: 640px;
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
        margin: 0;
    }

    /* ── Demo area ── */
    .demo {
        display: flex;
        align-items: flex-start;
        gap: 2.5rem;
    }

    .demo-img {
        border-radius: 8px;
        object-fit: cover;
        flex-shrink: 0;
    }

    .controls {
        display: flex;
        flex-direction: column;
        gap: 1rem;
    }

    .control-row {
        font-size: 0.95rem;
        color: #1a2535;
        display: flex;
        align-items: center;
        gap: 0.5rem;
        flex-wrap: wrap;
    }

    select {
        padding: 4px 8px;
        border: 1.5px solid #dce8e4;
        border-radius: 6px;
        font-size: 0.9rem;
        color: #1a2535;
        background: #fff;
        cursor: pointer;
    }

    select:focus {
        outline: none;
        border-color: #2ab5a0;
    }

    button {
        align-self: flex-start;
        background: #2ab5a0;
        color: white;
        font-size: 1rem;
        font-family: system-ui, sans-serif;
        padding: 10px 24px;
        border: none;
        cursor: pointer;
        border-radius: 6px;
        transition: background 0.15s;
    }

    button:hover {
        background: #239e8c;
    }

    .info {
        font-size: 0.9rem;
        color: #5a7080;
        margin: 0;
    }

    .info.exhausted {
        color: #c05050;
    }
</style>