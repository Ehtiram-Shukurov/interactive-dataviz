<div class="home">
  <h1>My Projects</h1>

  <nav>
    <a href="A0" class="card">
      <span class="label">A0</span>
      <span class="desc">Getting Started</span>
    </a>
    <a href="A1" class="card">
      <span class="label">A1</span>
      <span class="desc">Bar Chart</span>
    </a>
    <a href="A2" class="card">
      <span class="label">A2</span>
      <span class="desc">Scatter & Line</span>
    </a>
    <a href="A3" class="card">
      <span class="label">A3</span>
      <span class="desc">3D Visualization</span>
    </a>
  </nav>
</div>

<style>
  .home {
    padding: 2rem;
    font-family: Georgia, serif;
  }

  h1 {
    font-weight: normal;
    color: #c0541a;
    margin-bottom: 1.5rem;
  }

  nav {
    display: flex;
    gap: 1rem;
    flex-wrap: wrap;
  }

  .card {
    display: flex;
    flex-direction: column;
    gap: 0.3rem;
    padding: 1rem 1.5rem;
    border: 1.5px solid #e0d5c8;
    border-radius: 8px;
    text-decoration: none;
    background: white;
    transition: border-color 0.2s, box-shadow 0.2s;
  }

  .card:hover {
    border-color: #c0541a;
    box-shadow: 0 4px 14px rgba(192, 84, 26, 0.1);
  }

  .label {
    font-size: 1.2rem;
    font-weight: bold;
    color: #c0541a;
  }

  .desc {
    font-size: 0.8rem;
    color: #999;
  }
</style>