<div class="page">

    <section class="hero">
        <div class="hero-inner">
            <p class="eyebrow">CSCI 5609 · Data Visualization</p>
            <h1>Summer Movies:<br>A Data Visualization Journey</h1>
            <p class="description">
                A semester-long exploration of visualization techniques using a dataset of
                899 summer movies released between 1945 and 2023. Each assignment built on
                the last — starting with basic Svelte components, moving through interactive
                D3 charts, stepping into WebGL-powered 3D scenes, and finishing with a
                scroll-driven narrative visualization.
            </p>

            <div class="stats">
                <div class="stat">
                    <span class="stat-num">899</span>
                    <span class="stat-label">movies</span>
                </div>
                <div class="stat-divider"></div>
                <div class="stat">
                    <span class="stat-num">1945–2023</span>
                    <span class="stat-label">year range</span>
                </div>
                <div class="stat-divider"></div>
                <div class="stat">
                    <span class="stat-num">5</span>
                    <span class="stat-label">assignments</span>
                </div>
                <div class="stat-divider"></div>
                <div class="stat">
                    <span class="stat-num">3</span>
                    <span class="stat-label">main tools</span>
                </div>
            </div>

            <div class="tools-row">
                <span>SvelteKit</span>
                <span>D3.js</span>
                <span>Three.js</span>
                <span>TypeScript</span>
                <span>Svelte 5 Runes</span>
                <span>WebGL</span>
            </div>

            <p class="credit">By Ehtiram (Ezra) Shukurov</p>
        </div>
    </section>

    <section class="cards-section">
        <div class="cards-grid">

            <a href="/A0" class="card">
                <span class="card-bg-num">A0</span>
                <div class="card-content">
                    <p class="card-label">A0</p>
                    <h2>Getting Started</h2>
                    <p>First steps with Svelte 5's runes syntax. An interactive click counter with a dropdown to set the limit — a small intro to reactive state and event handling.</p>
                    <div class="tags">
                        <span>Svelte 5</span>
                        <span>TypeScript</span>
                    </div>
                </div>
            </a>

            <a href="/A1" class="card card-wide">
                <span class="card-bg-num">A1</span>
                <div class="card-content">
                    <p class="card-label">A1</p>
                    <h2>Summer Movies Dashboard</h2>
                    <p>A multi-chart dashboard built with D3. A bar chart for genre distribution, a bump chart tracking how genre rankings shifted year over year, and a heatmap showing which genres most often appear together in the same film.</p>
                    <div class="tags">
                        <span>D3.js</span>
                        <span>SVG</span>
                        <span>SvelteKit</span>
                    </div>
                </div>
            </a>

            <a href="/A2" class="card">
                <span class="card-bg-num">A2</span>
                <div class="card-content">
                    <p class="card-label">A2</p>
                    <h2>Interactive Scatter & Timeline</h2>
                    <p>A scatter plot where you can switch axes between year, rating, runtime, and vote count. A brushable line chart below filters the scatter in real time by year range.</p>
                    <div class="tags">
                        <span>D3.js</span>
                        <span>Brush</span>
                        <span>Svelte 5</span>
                    </div>
                </div>
            </a>

            <a href="/A3" class="card">
                <span class="card-bg-num">A3</span>
                <div class="card-content">
                    <p class="card-label">A3</p>
                    <h2>3D Beach Visualization</h2>
                    <p>Every summer movie as its own 3D shape rising from the sand — comedies as icosahedrons, romances as torus knots, dramas as octahedrons. Stacked by 5-year period in a beach scene with animated seagulls and a clapperboard.</p>
                    <div class="tags">
                        <span>Three.js</span>
                        <span>WebGL</span>
                        <span>D3.js</span>
                    </div>
                </div>
            </a>

            <a href="/A4" class="card">
                <span class="card-bg-num">A4</span>
                <div class="card-content">
                    <p class="card-label">A4</p>
                    <h2>Scrollytelling</h2>
                    <p>A scroll-driven narrative in two parts: a 2D bar chart comparing genre counts to 3 years prior as you scroll through every film, and a 3D scene where each era of summer cinema rises from the beach as you read the story.</p>
                    <div class="tags">
                        <span>Scrollytelling</span>
                        <span>Three.js</span>
                        <span>D3.js</span>
                    </div>
                </div>
            </a>

        </div>
    </section>

</div>

<style>
    .page {
        font-family: system-ui, -apple-system, sans-serif;
        color: #1a2535;
    }

    /* ── Hero ── */
    .hero {
        background: #111d2b;
        padding: 4.5rem 2rem 4rem;
    }

    .hero-inner {
        max-width: 760px;
        margin: 0 auto;
    }

    .eyebrow {
        font-size: 0.72rem;
        letter-spacing: 0.14em;
        text-transform: uppercase;
        color: #2ab5a0;
        margin: 0 0 1rem;
    }

    h1 {
        font-family: Georgia, serif;
        font-size: clamp(1.9rem, 4vw, 3rem);
        font-weight: normal;
        color: #eef2f5;
        line-height: 1.2;
        margin: 0 0 1.5rem;
    }

    .description {
        font-size: 0.95rem;
        color: #8aa0b4;
        line-height: 1.75;
        margin: 0 0 2rem;
        max-width: 640px;
    }

    /* stats row */
    .stats {
        display: flex;
        align-items: center;
        gap: 1.5rem;
        margin-bottom: 1.5rem;
        flex-wrap: wrap;
    }

    .stat {
        display: flex;
        flex-direction: column;
        gap: 2px;
    }

    .stat-num {
        font-size: 1.3rem;
        font-weight: 600;
        color: #eef2f5;
        line-height: 1;
    }

    .stat-label {
        font-size: 0.68rem;
        text-transform: uppercase;
        letter-spacing: 0.08em;
        color: #4e7080;
    }

    .stat-divider {
        width: 1px;
        height: 28px;
        background: #1e3045;
        flex-shrink: 0;
    }

    /* tools row */
    .tools-row {
        display: flex;
        flex-wrap: wrap;
        gap: 0.4rem;
        margin-bottom: 2rem;
    }

    .tools-row span {
        font-size: 0.72rem;
        padding: 4px 10px;
        border-radius: 20px;
        background: #1a2e40;
        color: #2ab5a0;
        border: 1px solid #1e3a50;
    }

    .credit {
        font-size: 0.8rem;
        color: #3d5a6e;
        margin: 0;
    }

    /* ── Cards ── */
    .cards-section {
        background: #f3f7f6;
        padding: 3rem 2rem 4rem;
    }

    .cards-grid {
        max-width: 960px;
        margin: 0 auto;
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        gap: 1.25rem;
    }

    .card-wide {
        grid-column: span 2;
    }

    .card {
        position: relative;
        display: flex;
        flex-direction: column;
        background: #fff;
        border: 1.5px solid #dce8e4;
        border-radius: 10px;
        padding: 1.5rem;
        text-decoration: none;
        color: inherit;
        overflow: hidden;
        transition: border-color 0.2s, box-shadow 0.2s, transform 0.2s;
    }

    .card:hover {
        border-color: #2ab5a0;
        box-shadow: 0 6px 20px rgba(42, 181, 160, 0.1);
        transform: translateY(-2px);
    }

    .card-bg-num {
        position: absolute;
        right: -0.25rem;
        bottom: -1rem;
        font-family: Georgia, serif;
        font-size: 7rem;
        font-weight: bold;
        color: #eef4f2;
        line-height: 1;
        pointer-events: none;
        user-select: none;
    }

    .card-content {
        position: relative;
        z-index: 1;
        display: flex;
        flex-direction: column;
        gap: 0.5rem;
        height: 100%;
    }

    .card-label {
        font-size: 0.68rem;
        font-weight: 700;
        letter-spacing: 0.1em;
        text-transform: uppercase;
        color: #2ab5a0;
        margin: 0;
    }

    .card h2 {
        font-size: 1.05rem;
        font-weight: 600;
        margin: 0;
        color: #111d2b;
    }

    .card p {
        font-size: 0.875rem;
        color: #5a7080;
        line-height: 1.65;
        margin: 0;
        flex: 1;
    }

    .tags {
        display: flex;
        flex-wrap: wrap;
        gap: 0.4rem;
        margin-top: 0.75rem;
    }

    .tags span {
        font-size: 0.68rem;
        background: #eaf4f2;
        color: #2a8a7e;
        padding: 3px 9px;
        border-radius: 20px;
    }
</style>