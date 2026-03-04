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
</script>

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
</style>
