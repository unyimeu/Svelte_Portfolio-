<svelte:head>
  <title>Contact me</title>
</svelte:head>

<nav class="nav">
  <ol>
    <li class="nav"><a href="../">Home</a></li>
    <li class="nav"><a href="../projects">Projects</a></li>
    <li class="nav"><a class="current" href="./">Contact</a></li>
    <li class="nav">
      <a href="https://github.com/unyimeu" target="_blank">GitHub Profile</a>
    </li>
    <li class="nav"><a href="../resume">Resume</a></li>
  </ol>
</nav>
<h1>Contact</h1>
<form action="mailto:unyime@mit.edu" method="get">
  <label>
    Subject:
    <input type="text" name="subject" id="subject" required />
  </label>

  <label>
    Enter your email:
    <input type="email" name="email" id="email" required />
  </label>

  <label>
    Body:
    <textarea id="body" name="body"></textarea>
  </label>

  <button type="submit">Send</button>
</form>
