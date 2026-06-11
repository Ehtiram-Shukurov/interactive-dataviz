<script lang="ts">
    import favicon from '$lib/assets/favicon.svg';

    let { children } = $props();
</script>

<svelte:head>
    <link rel="icon" href={favicon} />
</svelte:head>

<header class="nav">
    <a href="" class="monogram">Homepage</a>
    <nav>
        {#each ['A0', 'A1', 'A2', 'A3', 'A4'] as id}
            <a href="{id}">{id}</a>
        {/each}
    </nav>
</header>

{@render children()}

<style>
    .nav {
        position: sticky;
        top: 0;
        z-index: 100;
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 0 2rem;
        height: 48px;
        background: #111d2b;
        border-bottom: 1px solid #1e3045;
    }

    .monogram {
        font-family: Georgia, serif;
        font-size: 1rem;
        font-weight: bold;
        color: #2ab5a0;
        text-decoration: none;
        letter-spacing: 0.06em;
    }

    nav {
        display: flex;
        gap: 0.1rem;
    }

    nav a {
        color: #6b8a9e;
        text-decoration: none;
        font-family: system-ui, sans-serif;
        font-size: 0.78rem;
        padding: 5px 10px;
        border-radius: 4px;
        transition: color 0.15s;
    }

    nav a:hover {
        color: #2ab5a0;
    }
</style>