<script>
  export let data = {};
</script>

<article>
  <h2>{data.title}</h2>
  <h3>{data.author}</h3>
  <img src={data.image} alt={data.title} />
  <p>
    {data.description}
  </p>
</article>

<style>
  :global(.reading) {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1em;
  }
  article {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  img {
    width: 50%;
    height: auto;
  }
  article h2 {
    margin: 0;
  }
</style>
