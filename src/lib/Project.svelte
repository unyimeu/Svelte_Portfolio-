<script>
  export let data = {};
</script>

<article>
  <h2>{data.title}</h2>
  <img src={data.image} alt={data.title} />
  <p>
    <strong>{data.year}</strong> | {data.url} | {data.description}
  </p>
</article>

<style>
  :global(.projects) {
    display: grid;
    gap: 1em;
    grid-template-columns: repeat(auto-fill, minmax(15em, 1fr));
  }

  article {
    grid-template-rows: subgrid;
    grid-row: span 3;
  }

  article h2 {
    margin: 0;
  }
</style>
