<script>
  import { onMount } from "svelte";

  let results = null;
  let specs = null;
  let runningTime = 0;
  let timer;

  function updateRunningTime() {
    runningTime += 0.2;
  }

  async function runTest() {
    runningTime = 0;
    timer = setInterval(updateRunningTime, 200);

    try {
      const response = await fetch("/api/runTest");
      results = await response.json();
    } catch (error) {
      results = { error: error.message };
    } finally {
      clearInterval(timer);
    }
  }

  async function getSpecs() {
    try {
      const response = await fetch("/api/getSpecs");
      specs = await response.json();
    } catch (error) {
      specs = { error: error.message };
    }
  }

  onMount(() => {
    runTest();
    getSpecs();
  });
</script>

<main>
  <nav class="navbar">
    <a href="https://php-sqlite.xpressionist.com">PHP / SQLite Test</a>
    <a href="https://node-sqlite.xpressionist.com">Node / SQLite Test</a>
    <a href="https://nextjs-sqlite.xpressionist.com">Next.js / SQLite Test</a>
    <a href="https://sveltekit-sqlite.xpressionist.com" class="active"
      >SvelteKit / SQLite Test</a
    >
  </nav>
  <h1>SvelteKit 2.5.24 / SQLite Performance Test</h1>
  <p>
    Built by <a href="https://x.com/fedeandri">@fedeandri</a> starting from
    <a
      href="https://x.com/ashleyrudland/status/1826991719646179583"
      target="_blank"
      rel="noopener noreferrer">@ashleyrudland</a
    >'s PHP code. See the source of this
    <a
      href="https://github.com/fedeandri/sveltekit-sqlite"
      target="_blank"
      rel="noopener noreferrer">SvelteKit / SQLite Test on Github</a
    >.
    <br /> From my SQLite tests
    <strong>PHP+JS writes ~10x faster and reads ~40x faster</strong> than Node based
    apps.
  </p>
  <div class="container">
    <div class="card">
      <h2>SvelteKit / SQLite Test</h2>
      <small>(5 minutes cache)</small>
      <div id="results-box">
        {#if !results}
          <div class="flex-row">
            <div class="spinner"></div>
            <span>Running test ({runningTime.toFixed(1)}s)...</span>
          </div>
        {:else if results.error}
          <p>Error: {results.error}</p>
        {:else}
          <ul>
            <li>
              Duration: {results.duration
                ? results.duration.toLocaleString()
                : "N/A"} seconds
            </li>
            <li>
              DB size: {results.dbSizeInMb
                ? results.dbSizeInMb >= 1024
                  ? (results.dbSizeInMb / 1024).toLocaleString(undefined, {
                      maximumFractionDigits: 1,
                    }) + "GB"
                  : results.dbSizeInMb.toLocaleString() + "MB"
                : "N/A"}
            </li>
            <li>
              Records processed: {results.total
                ? results.total.toLocaleString()
                : "N/A"}
            </li>
            <li>
              Reads/sec: {results.readsPerSecond
                ? results.readsPerSecond.toLocaleString()
                : "N/A"}
            </li>
            <li class="font-medium">
              Writes/sec: {results.writesPerSecond
                ? results.writesPerSecond.toLocaleString()
                : "N/A"}
            </li>
            {#if results.failureRate && results.failureRate > 0}
              <li>Failure rate: {results.failureRate}%</li>
            {/if}
          </ul>
        {/if}
      </div>
    </div>
    <div class="card">
      <h2>VPS Specs</h2>
      <small
        >(<a href="https://www.vultr.com/pricing/" target="_blank"
          >Vultr High Performance Intel $12/mo</a
        >
        with
        <a href="https://www.cloudpanel.io/" target="_blank">CloudPanel</a
        >)</small
      >
      <div id="specs-box">
        {#if specs}
          <ul>
            {#each Object.entries(specs) as [key, value]}
              <li>{key}: {value}</li>
            {/each}
          </ul>
        {:else}
          <div class="flex-row">
            <div class="spinner"></div>
            <span>Loading specs data...</span>
          </div>
        {/if}
      </div>
    </div>
  </div>
</main>

<style>
  main {
    display: flex;
    min-height: 100vh;
    width: 100%;
    max-width: 1200px;
    flex-direction: column;
    padding: 2.5rem;
  }
  h1 {
    font-weight: bold;
    font-size: 1.25rem;
    margin-bottom: 1.5rem;
  }
  p {
    margin-bottom: 2.5rem;
    line-height: 1.4em;
  }
  a {
    color: #2563eb;
    text-decoration: none;
    text-underline-offset: 4px;
  }
  .container {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }
  @media (min-width: 640px) {
    .container {
      flex-direction: row;
      gap: 2.5rem;
    }
  }
  .card {
    background-color: white;
    padding: 1.5rem;
    border-radius: 0.5rem;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
    flex: 1;
    margin-bottom: 1rem;
  }
  .card h2 {
    font-size: 1.125rem;
    font-weight: 600;
    margin-bottom: 0;
  }
  .spinner {
    width: 20px;
    height: 20px;
    margin-top: 1rem;
    border: 2px solid #f3f3f3;
    border-top: 2px solid #2563eb;
    border-radius: 50%;
    animation: spin 1s linear infinite;
  }
  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
  .flex-row {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 0.25rem;
  }
  .navbar {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    background-color: #f3f4f6;
    padding: 0.5rem;
    border-radius: 0.25rem;
    border: 1px solid #d3d3d3;
    margin-bottom: 1.5rem;
  }
  .navbar a {
    text-decoration: none;
    color: #4b5563;
    padding: 0.5rem 1rem;
    border-radius: 0.25rem;
    border: 1px solid #d3d3d3;
    transition: background-color 0.3s ease;
    margin: 0.25rem;
  }
  .navbar a:hover {
    background-color: #d1d5db;
    color: black !important;
  }
  .navbar a.active {
    background-color: #2563eb;
    border: 1px solid #2563eb;
    color: white !important;
  }
</style>
