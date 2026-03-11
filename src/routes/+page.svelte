<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";

  import { onMount } from "svelte";

  let githubData = null; // This will eventually hold our Github stats
  let loading = true; // This will be true *until* the fetch's promise resolves to a value
  let error = null; // If the API call resulted in an error, it will go into this variable

  onMount(async () => {
    // code from your old retrieveGithubData function
    try {
      console.log("Page has been mounted!");
      let response = await fetch("https://api.github.com/users/unyimeu");
      console.log(response);
      githubData = await response.json();
      console.log(githubData);
    } catch (err) {
      // if the "try" block runs into an error, cancel excecution and run this code instead
      error = err;
    }
    loading = false; // don't forget to add this line!
  });

  let data = fetch("https://api.github.com/users/unyimeu");
</script>

<h1>Unyimeabasi Usua</h1>
<p>
  Hi, my name is Unyi. I am a junior at MIT studying Artificial Intelligence and
  Decision Making. This summer I will be interning as a SWE @ Adobe on their
  Data and AI Team.
</p>
<img src="images/drinking boba.jpg" alt="Unyimeabasi Usua" width="600" />

{#if loading}
  <p>Loading...</p>
{:else if error}
  <p>Something went wrong: {error.message}</p>
{:else}
  <!-- The data is {JSON.stringify(githubData)} -->
  <section>
    <h2>My GitHub Stats</h2>
    <dl>
      <dt>Followers</dt>
      <dd>{githubData.followers}</dd>
      <dt>Following</dt>
      <dd>{githubData.following}</dd>
      <dt>Public Repositories</dt>
      <dd>{githubData.public_repos}</dd>
    </dl>
  </section>
{/if}

<h2>Projects</h2>

<div class="projects">
  {#each projects.slice(0, 3) as p}
    <Project data={p} />
  {/each}
</div>

<h2>What I'm Reading</h2>

<div class="reading">
  {#each reading.slice(0, 3) as r}
    <ReadingItem data={r} />
  {/each}
</div>

<style>
  dl {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
  }

  dt {
    grid-row: 1;
    font-size: 0.75rem;
    text-transform: uppercase;
    color: #666;
  }

  dd {
    grid-row: 2;
    font-size: 2rem;
    font-weight: bold;
    margin: 0;
  }
</style>
