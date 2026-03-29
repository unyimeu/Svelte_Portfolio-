<script>
  import * as d3 from "d3";

  let width = 600;
  let height = 200;
  let xAxis, yAxis;

  export let data = [];
  export let title = "";

  let margin = { top: 40, right: 130, bottom: 60, left: 100 };
  let innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  $: xScale = d3
    .scaleLinear()
    .domain([0, d3.max(data, (d) => d.value) || 1])
    .range([0, innerWidth]);

  $: yScale = d3
    .scaleBand()
    .domain(data.map((d) => d.label))
    .range([0, innerHeight])
    .padding(0.2);

  $: colorScale = d3
    .scaleOrdinal()
    .domain(["css", "html", "svelte", "js"])
    .range(["#4e79a7", "#76b7b2", "#59a14f", "#2d6a2d"]);

  $: maxBar = d3.greatest(data, (d) => d.value);

  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(
      d3.axisBottom(xScale).ticks(
        Math.min(
          d3.max(data, (d) => d.value),
          10,
        ),
      ),
    );
    d3.select(yAxis).call(d3.axisLeft(yScale));
  }
</script>

<div class="container">
  <svg viewBox="0 0 {width} {height}">
    <text
      x={margin.left + innerWidth / 2}
      y={margin.top / 2}
      text-anchor="middle"
      class="chart-title"
    >
      {title}
    </text>

    <g
      transform="translate({margin.left}, {margin.top + innerHeight})"
      bind:this={xAxis}
    />
    <g transform="translate({margin.left}, {margin.top})" bind:this={yAxis} />
    <g transform="translate({margin.left}, {margin.top})">
      {#each data as d}
        <rect
          x={0}
          y={yScale(d.label)}
          width={xScale(d.value)}
          height={yScale.bandwidth()}
          fill={colorScale(d.label)}
        />
      {/each}
      {#if maxBar}
        <!-- highlight outline around the longest bar -->
        <rect
          x={0}
          y={yScale(maxBar.label)}
          width={xScale(maxBar.value)}
          height={yScale.bandwidth()}
          fill="none"
          stroke="currentColor"
          stroke-width="2"
        />
        <!-- leader line -->
        <!-- <line
          x1={xScale(maxBar.value)}
          y1={yScale(maxBar.label) + yScale.bandwidth() / 2}
          x2={xScale(maxBar.value) + 30}
          y2={yScale(maxBar.label) + yScale.bandwidth() / 2}
          stroke="currentColor"
          stroke-width="1"
        /> -->
        <!-- annotation text -->
        <text
          x={xScale(maxBar.value) + 35}
          y={yScale(maxBar.label) + yScale.bandwidth() / 2}
          dominant-baseline="start"
          class="annotation"
        >
          Most lines of code
        </text>
      {/if}
      <!-- x-axis label -->
      <text
        x={innerWidth / 2}
        y={innerHeight + margin.bottom - 10}
        text-anchor="middle"
        class="axis-label"
      >
        Lines of Code
      </text>
      <!-- y-axis label -->
      <text
        x={-(innerHeight / 2)}
        y={-margin.left + 20}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label"
      >
        Language
      </text>
    </g>
  </svg>
  <ul class="legend">
    {#each data as d}
      <li style="--color: {colorScale(d.label)}">
        <span class="swatch"></span>
        {d.label} <em>({d.value})</em>
      </li>
    {/each}
  </ul>
</div>

<style>
  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
  }

  .container {
    display: flex;
  }

  .legend {
    flex: 1;
    list-style: none;
    padding: 0;
    margin: 0;
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  li {
    display: flex;
    align-items: center;
    gap: 4px;
  }

  .swatch {
    width: 12px;
    height: 12px;
    background-color: var(--color);
    flex-shrink: 0;
  }

  .chart-title {
    font-size: 0.9em;
    font-weight: bold;
    fill: currentColor;
  }

  .axis-label {
    font-size: 0.7em;
    fill: currentColor;
  }

  .annotation {
    font-size: 0.6em;
    fill: black;
    font-style: italic;
  }
</style>
