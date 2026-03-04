<script>
  import { page } from "$app/stores";
  import { base } from "$app/paths";

  let pages = [
    { url: "/", title: "About" },
    { url: "/projects", title: "Projects" },
    { url: "/resume", title: "Resume" },
    { url: "/contact", title: "Contact" },
    { url: "https://github.com/unyimeu", title: "Github" },
  ];

  let colorScheme = "light dark";

  let localStorage = globalThis.localStorage ?? {};

  if (localStorage.colorScheme) {
    // if localStorage has a colorScheme property
    colorScheme = localStorage.colorScheme; // override the default colorScheme
  }

  let root = globalThis.document?.documentElement;
  $: root?.style.setProperty("color-scheme", colorScheme);
  $: localStorage.colorScheme = colorScheme;

</script>

<label class="color-scheme-switch">
  Theme:
  <select bind:value={colorScheme}>
    <option value="light dark">Automatic</option>
    <option value="light">Light</option>
    <option value="dark">Dark</option>
  </select>
</label>

<nav class="nav">
  {#each pages as p}
    <a
      href={!p.url.startsWith("http") ? base + p.url : p.url}
      class:current={p.url === "/" // is this link the home page?
        ? $page.url.pathname === base + "/" // if yes - set current = true if the path name matches. Else, set current = true if the path name starts correctly
        : $page.url.pathname.startsWith(base + p.url)}
      target={p.url.startsWith("http") ? "_blank" : null}
    >
      {p.title}
    </a>
  {/each}
</nav>
<slot />

<style>
  nav {
    display: flex;
    margin-bottom: 1em;
    border-bottom-width: 1px;
    border-bottom-style: solid;
    border-bottom-color: oklch(80% 3% 200);
  }

  nav a {
    flex: 1;
    text-decoration: none;
    color: inherit;
    text-align: center;
    padding: 3em;
  }

  :root {
    color-scheme: light dark;
  }

  nav {
    --border-color: oklch(50% 10% 200 / 40%);
    border-bottom: 2px solid var(--border-color);
  }

  .current {
    border-bottom: 4px solid var(--border-color);
  }

  nav a:hover {
    background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
  }

  label,
  .color-scheme-switch {
    position: absolute;
    top: 1em;
    right: 1em;
  }

  .color-scheme-switch {
    display: inline-flex;
    gap: 4px;
    font-size: 80%;
  }
</style>
