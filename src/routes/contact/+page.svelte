<svelte:head>
  <title>Contact me</title>
</svelte:head>

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
