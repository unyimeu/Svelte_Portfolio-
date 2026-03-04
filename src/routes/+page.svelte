<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";
</script>

<h1>Unyimeabasi Usua</h1>
<p>
  Hi, my name is Unyi. I am a junior at MIT studying Artificial Intelligence and
  Decision Making. This summer I will be interning as a SWE @ Adobe on their
  Digital Experience Data Insights Team.
  
</p>
<img src="images/drinking boba.jpg" alt="Unyimeabasi Usua" width="600" />

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
