<script>
  import { base } from "$app/paths";
  import { onMount } from "svelte";
  import * as d3 from "d3";
  import BarHorizontal from "$lib/BarHorizontal.svelte";
  import LineChart from "$lib/LineChart.svelte";
  import { computePosition, autoPlacement, offset } from "@floating-ui/dom";

  let locData = [];
  let commits = [];

  onMount(async () => {
    locData = await d3.csv(`${base}/loc.csv`, (row) => ({
      ...row,
      line: Number(row.line),
      length: Number(row.length),
      depth: Number(row.depth),
      date: new Date(row.date + "T00:00" + row.timezone),
      datetime: new Date(row.datetime),
    }));

    commits = d3
      .groups(locData, (d) => d.commit)
      .map(([commit, lines]) => {
        let first = lines[0];
        let { author, date, time, timezone, datetime } = first;
        let ret = {
          id: commit,
          url: "https://github.com/unyimeu/Svelte_Portfolio-/commit/" + commit,
          author,
          date,
          time,
          timezone,
          datetime,
          hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
          totalLines: lines.length,
          lines: lines,
        };

        return ret;
      });

    commits = d3.sort(commits, (d) => -d.totalLines);
  });

  let width = 1000,
    height = 600;

  $: [minDate, maxDate] = d3.extent(commits.map((d) => d.date));
  $: maxDatePlusOne = new Date(maxDate);
  $: maxDatePlusOne.setDate(maxDatePlusOne.getDate() + 1);

  let margin = { top: 20, right: 20, bottom: 5, left: 5 };

  let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left,
  };
  usableArea.width = usableArea.right - usableArea.left;
  usableArea.height = usableArea.bottom - usableArea.top;

  $: xScale = d3
    .scaleTime()
    .domain([minDate, maxDatePlusOne])
    .range([usableArea.left, usableArea.right])
    .nice();

  $: yScale = d3
    .scaleLinear()
    .domain([24, 0])
    .range([usableArea.bottom, usableArea.top]);

  $: rScale = d3
    .scaleSqrt()
    .domain([0, d3.max(commits, (d) => d.totalLines)])
    .range([5, 30]);

  $: {
    d3.select(yAxisGridlines).call(
      d3.axisLeft(yScale).tickFormat("").tickSize(-usableArea.width),
    );
  }

  let xAxis, yAxis;
  let yAxisGridlines;
  $: {
    d3.select(xAxis).call(d3.axisBottom(xScale));
    d3.select(yAxis).call(
      d3
        .axisLeft(yScale)
        .tickFormat((d) => String(d % 24).padStart(2, "0") + ":00"),
    );
  }

  let hoveredIndex = -1;
  $: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};

  let commitTooltip;
  let tooltipPosition = { x: 0, y: 0 };

  async function dotInteraction(index, evt) {
    let hoveredDot = evt.target;
    if (evt.type === "mouseenter") {
      hoveredIndex = index;
      tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
        strategy: "fixed", // because we use position: fixed
        middleware: [
          offset(5), // spacing from tooltip to dot
          autoPlacement(), // see https://floating-ui.com/docs/autoplacement
        ],
      });
    } else if (evt.type === "mouseleave") {
      hoveredIndex = -1;
    } else if (evt.type === "click") {
      let commit = commits[index];
      if (!clickedCommits.includes(commit)) {
        // Add the commit to the clickedCommits array
        clickedCommits = [...clickedCommits, commit];
      } else {
        // Remove the commit from the array
        clickedCommits = clickedCommits.filter((c) => c !== commit);
      }
    }
  }

  let clickedCommits = [];

  function isCommitBrushed(commit) {
    if (!brushSelection) {
      return false;
    }
    let min = { x: brushSelection[0][0], y: brushSelection[0][1] };
    let max = { x: brushSelection[1][0], y: brushSelection[1][1] };
    let x = xScale(commit.date);
    let y = yScale(commit.hourFrac);
    return x >= min.x && x <= max.x && y >= min.y && y <= max.y;
  }

  $: brushedCommits = brushSelection ? commits.filter(isCommitBrushed) : [];

  $: selectedCommits = Array.from(
    new Set([...clickedCommits, ...brushedCommits]),
  );

  $: selectedLines =
    selectedCommits.length > 0
      ? selectedCommits.flatMap((d) => d.lines)
      : locData;

  $: selectedCounts = d3.rollup(
    selectedLines,
    (v) => v.length,
    (d) => d.type,
  );

  $: allTypes = Array.from(new Set(locData.map((d) => d.type)));

  $: barData = allTypes.map((type) => ({
    label: String(type),
    value: selectedCounts.get(type) || 0,
  }));

  export let title = "";
  let svg;
  $: {
    d3.select(svg).call(
      d3
        .brush()
        .extent([
          [usableArea.left, usableArea.top],
          [usableArea.right, usableArea.bottom],
        ])
        .on("start brush end", brushed),
    );

    d3.select(svg).selectAll(".dots, .overlay ~ *").raise();
  }

  $: brushSelection = null;

  function brushed(evt) {
    brushSelection = evt.selection;
  }

  let linesByDate = [];

  $: {
    // 1. Get the count for each date in the data
    const rolled = d3
      .rollups(
        locData,
        (v) => v.length,
        (d) => d3.timeDay.floor(d.datetime),
      )
      .map(([date, count]) => ({ date, count }));

    // 2. Get an array of all days covered by the data
    const [minDate, maxDate] = d3.extent(rolled, (d) => d.date);
    const allDays = d3.timeDays(minDate, d3.timeDay.offset(maxDate, 1));

    // 3. Build linesByDate by filling all undefined dates with 0 counts
    linesByDate = allDays.map((date) => ({
      date,
      count:
        rolled.find((d) => d.date.getTime() === date.getTime())?.count ?? 0,
    }));
  }
</script>

<svelte:head>
  <title>Meta</title>
</svelte:head>

<h1>Meta</h1>

<p>Meta page to visualize project data.</p>
<h3>Commits by Time of Day</h3>
<svg viewBox="0 0 {width} {height} " bind:this={svg}>
  <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
  <g
    class="gridlines"
    transform="translate({usableArea.left}, 0)"
    bind:this={yAxisGridlines}
  />

  <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />

  <g class="dots">
    {#each commits as commit, index}
      <circle
        class:selected={selectedCommits.includes(commit)}
        on:click={(evt) => dotInteraction(index, evt)}
        on:mouseenter={(evt) => dotInteraction(index, evt)}
        on:mouseleave={(evt) => dotInteraction(index, evt)}
        cx={xScale(commit.datetime)}
        cy={yScale(commit.hourFrac)}
        r={rScale(commit.totalLines)}
        fill="steelblue"
        fill-opacity="0.7"
      />
    {/each}
  </g>
</svg>

<dl
  class="info tooltip"
  hidden={hoveredIndex === -1}
  style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px"
  bind:this={commitTooltip}
>
  <dt>Commit</dt>
  <dd><a href={hoveredCommit.url} target="_blank">{hoveredCommit.id}</a></dd>

  <dt>Date</dt>
  <dd>{hoveredCommit.datetime?.toLocaleString("en", { dateStyle: "full" })}</dd>

  <!-- Add: Time, author, lines edited -->
  <dt>Time</dt>
  <dd>
    {hoveredCommit.datetime?.toLocaleTimeString("en", { timeStyle: "short" })}
  </dd>
  <dt>Author</dt>
  <dd>{hoveredCommit.author}</dd>
  <dt>Lines Edited</dt>
  <dd>{hoveredCommit.totalLines}</dd>
</dl>

<BarHorizontal
  data={barData}
  title={selectedCommits.length > 0
    ? `Lines of Code: ${selectedCommits.length} Selected Commits`
    : "Lines of Code: Website Breakdown"}
/>
<LineChart data={linesByDate} />

<!-- <dl class="stats">
  <dt>Total <abbr title="Lines of code">LOC</abbr></dt>
  <dd>{locData.length}</dd>
</dl>

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
</style> -->

<style>
  svg {
    overflow: visible;
  }
  .gridlines {
    stroke-opacity: 0.2;
  }

  circle {
    transition: 200ms;
    &:hover {
      fill: darkgreen;
    }
  }

  dl.info {
    display: grid;
    grid-template-columns: max-content 1fr;
    gap: 0.5rem 1rem;
    background: #f9f9f9;
    padding: 1rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    width: max-content;
    background-color: oklch(100% 0% 0 / 85%);
    box-shadow: inset 0 0 0 1px oklch(0% 0% 0 / 10%);
    backdrop-filter: blur(3px);

    transition-duration: 500ms;
    transition-property: opacity, visibility;

    &[hidden]:not(:hover, :focus-within) {
      opacity: 0;
      visibility: hidden;
    }
  }

  dl.info dt {
    font-size: 0.75rem;
    text-transform: uppercase;
    color: #666;
  }

  .tooltip {
    position: fixed;
    top: 1em;
    left: 1em;
  }

  .selected {
    fill: var(--color-accent);
  }

  @keyframes marching-ants {
    to {
      stroke-dashoffset: -8; /* 5 + 3 */
    }
  }

  svg :global(.selection) {
    fill-opacity: 10%;
    stroke: black;
    stroke-opacity: 70%;
    stroke-dasharray: 5 3;
    animation: marching-ants 2s linear infinite;
  }
</style>
