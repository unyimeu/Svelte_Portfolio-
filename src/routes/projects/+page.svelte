<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>
<nav class="nav">
  <ol>
    <li class="nav"><a href="../">Home</a></li>
    <li class="nav"><a class="current" href="./">Projects</a></li>
    <li class="nav"><a href="../contact">Contact</a></li>
    <li class="nav">
      <a href="https://github.com/unyimeu" target="_blank">GitHub Profile</a>
    </li>
    <li class="nav"><a href="../resume">Resume</a></li>
  </ol>
</nav>
<h1>{projects.length} Projects</h1>
<div class="projects">
  {#each projects as p}
    <Project data={p} />
  {/each}
</div>
