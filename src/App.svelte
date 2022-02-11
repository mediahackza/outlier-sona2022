<script>
  let tooltip = 'Tooltip'
  let colors = [
    '#d53e4f',
    '#f46d43',
    '#fdae61',
    '#fee08b',

    '#e6f598',
    '#abdda4',
    '#66c2a5',
    '#3288bd',
  ]
  let formatNumber = new Intl.NumberFormat()
  let promise = false
  import { tidy, pivotLonger, mutate, arrange, desc } from '@tidyjs/tidy'
  let width = 300 - 10
  let wordsWidth = 300
  let frameWidth = 800
  let speechMax = 0
  $: console.log(frameWidth)
  let allData = []

  let height = 400
  $: height = width * 0.3

  let margin = {
    top: 10,
    right: 30,
    bottom: 10,
    left: 0,
  }
  import * as d3 from 'd3'
  import { exclude_internal_props } from 'svelte/internal'
  let parseDate = d3.timeParse('%Y-%m-%d')
  let formatDate = d3.timeFormat('%-m/%y')
  let formatDateLong = d3.timeFormat('%e %B %Y')
  let data = []
  let maximum = []
  let max = 0
  $: console.log(max)
  let categories = [
    'corruption',
    'education',
    'employability',
    'energy_policy',
    'ret',
    'transport',
    'welfare',
  ]
  const labels = [
    { name: 'corruption', label: 'Corruption' },
    { name: 'education', label: 'Education' },
    { name: 'employability', label: 'Employment' },
    { name: 'energy_policy', label: 'Energy & Eskom' },
    { name: 'ret', label: 'RET' },
    { name: 'transport', label: 'transport' },
    { name: 'welfare', label: 'Welfare' },
  ]

  let sonas = []

  async function getData() {
    await fetch('https://api.datadesk.co.za/json.php?table=dd_sona2022_7232953')
      .then((response) => response.json())
      .then((response) => {
        response.forEach((d) => {
          categories.forEach((c) => {
            d[c] = +d[c]
            if (d[c] > max) {
              max = d[c]
            }
          })
          d.total = +d.total_speech_wo_greetings
          d.sona = parseDate(d.sona)
        })

        // get sona list
        response.forEach((f) => {
          sonas.push(f.sona)
        })
        allData = response
        console.log(allData)

        speechMax = d3.max(allData, (d) => d.total)
        console.log(speechMax)

        data = tidy(
          response,
          pivotLonger({
            cols: categories,
            namesTo: 'cat',
            valuesTo: 'val',
          })
        )
      })

    // get maximums
    await fetch(
      'https://api.datadesk.co.za/json.php?table=dd_sona_maximums_new_2679993'
    )
      .then((response) => response.json())
      .then((response) => {
        response.forEach((d) => {
          d.words = +d.words
          d.date = parseDate(d.date)
        })
        maximum = response

        promise = true
      })
  }

  $: xScale = d3
    .scaleTime()
    .range([margin.left, width - margin.right])
    .domain(d3.extent(data, (d) => d.sona))

  $: yScale = d3
    .scaleLinear()
    .range([height - margin.bottom, margin.top + 20])
    .domain([0, max])

  $: speechXScale = d3
    .scaleLinear()
    .range([0, wordsWidth - 200])
    .domain([0, speechMax])

  $: line_gen = d3
    .line()
    .x((d) => xScale(d.sona))
    .y((d) => yScale(d.val))

  $: area_gen = d3
    .area()
    .x((d) => xScale(d.sona))
    .y1(height - margin.bottom)
    .y0((d) => yScale(d.val))

  getData()

  function getLabel(cat) {
    let category = labels.filter((f) => f.name === cat)[0]
    return category.label
  }

  function getLabelX(cat) {
    let pos = maximum.filter((f) => f.topic === cat)[0]

    return xScale(pos.date)
  }
  function getLabelText(cat) {
    let pos = maximum.filter((f) => f.topic === cat)[0]
    return pos.words
  }

  $: xTicks = xScale.ticks(5)
  $: yTicks = yScale.ticks(6)
  $: console.log(width)

  function speechRemainder(ad) {
    let total = ad.total
    let remainder =
      ad.total -
      ad.corruption -
      ad.education -
      ad.employability -
      ad.energy_policy -
      ad.ret -
      ad.transport -
      ad.welfare
    let d = ad.sona
    console.log({ d, remainder, total })

    return remainder
  }
  let x = -1000
  let y = -1000

  function hover(e, ad) {
    x = e.clientX + 10
    y = e.clientY - 100
    tooltip = `<div class="tooltip-inner">
      <div class="tt-title">${formatDateLong(ad.sona)}</div></div>
      <div class="tt-label-row">
        <div class="tt-label" style="color: ${colors[0]};">Corruption</div>
        <div class="tt-detail">${formatNumber.format(ad.corruption)} words</div>
        </div>
        <div class="tt-label-row">
        <div class="tt-label" style="color: ${colors[1]};">Education</div>
        <div class="tt-detail">${formatNumber.format(ad.education)} words</div>
        </div>
        <div class="tt-label-row">
        <div class="tt-label" style="color: ${colors[2]};">Employment</div>
        <div class="tt-detail">${formatNumber.format(
          ad.employability
        )} words</div>
        </div>
        <div class="tt-label-row">
        <div class="tt-label" style="color: ${colors[3]};">Energy & Eskom</div>
        <div class="tt-detail">${formatNumber.format(
          ad.energy_policy
        )} words</div>
        </div>
        <div class="tt-label-row">
        <div class="tt-label" style="color: ${colors[4]};">RET</div>
        <div class="tt-detail">${formatNumber.format(ad.ret)} words</div>
        </div>
        <div class="tt-label-row">
        <div class="tt-label" style="color: ${colors[5]};">Transport</div>
        <div class="tt-detail">${formatNumber.format(ad.transport)} words</div>
        </div>
        <div class="tt-label-row">
        <div class="tt-label" style="color: ${colors[6]};">Welfare</div>
        <div class="tt-detail">${formatNumber.format(ad.welfare)} words</div>
        </div>
      `
  }
</script>

<main>
  <div class="tooltip" style="top: {y}px; left: {x}px;">{@html tooltip}</div>
  <div class="page-title">Words spoken in each of the past 6 SONAs</div>
  <div class="sub-title">
    In the chart below we look at seven key topics and how many words each has
    received in the past six State of the Nation Addresses. The maximum number
    of words for each is indicated by the red markers. <em>
      Note, this is not how many times the word has been used but how many words
      were dedicated to each of the topics in general.
    </em>
  </div>
  {#if promise}
    <div class="charts" bind:clientWidth={frameWidth}>
      <!-- <div class="chart" bind:clientWidth={width}>Chart</div>
        <div class="chart" bind:clientWidth={width}>Chart</div>
        <div class="chart" bind:clientWidth={width}>Chart</div> -->
      {#each categories as cat, i}
        <div class="chart">
          <div class="chart-title">{getLabel(cat)}</div>
          <svg width={width - margin.left - margin.right} {height}>
            <path
              class="chart-area"
              d={area_gen(data.filter((c) => c.cat === cat))}
            />
            <path
              class="chart-line"
              d={line_gen(data.filter((c) => c.cat === cat))}
            />

            <line
              class="marker"
              x1={getLabelX(cat)}
              y1={margin.top}
              x2={getLabelX(cat)}
              y2={height - margin.bottom}
            />
            <text x={getLabelX(cat) - 5} y={margin.top + 5}
              >{formatNumber.format(getLabelText(cat))} words</text
            >
            {#each sonas as sona}
              <text class="x-tick" y={height} x={xScale(sona)}
                >{formatDate(sona)}</text
              >
            {/each}
          </svg>
        </div>
        {#if i === 5}
          <div class="chart {width > 750 ? '' : 'spacer'}" />
        {/if}
      {/each}
    </div>

    <!-- Words Spoken -->
    <div class="words" bind:clientWidth={wordsWidth}>
      <div class="page-title">The longest (and shortest) speeches</div>
      <div class="sub-title">
        The past six SONA speeches have ranged in length from 4,500 words to
        more than 8,000 words. This excludes the often lengthy greetings that
        preced the speech. This year's SONA speech was the second longest in
        recent times, just 133 words shy of the longest in February 2019.
      </div>
      <div class="legend">
        {#each categories as cat, i}
          <div
            class="legend-block {i === 0 || i === 1 || i === 6 ? 'light' : ''}"
            style="background: {colors[i]};"
          >
            {getLabel(cat)}
          </div>
        {/each}
        <div
          class="legend-block block-extra"
          style="
        background: {colors[7]};"
        >
          other
        </div>
      </div>
      {#each allData as ad}
        <div class="bar-label">
          {formatDateLong(ad.sona)}<br />
          <div class="word-count">
            SONA speech was <strong
              >{formatNumber.format(ad.total)} words</strong
            > in total
          </div>
        </div>
        <div
          class="sona-row"
          on:mouseover={(event) => {
            hover(event, ad)
          }}
          on:focus={(event) => {
            hover(event, ad)
          }}
          on:mousemove={(event) => {
            hover(event, ad)
          }}
          on:mouseout={(event) => {
            x = -1000
            y = -1000
          }}
          on:blur={(event) => {
            x = -1000
            y = -1000
          }}
        >
          <div
            class="block"
            style="width: {speechXScale(
              ad.corruption
            )}px; background: {colors[0]};"
          >
            &nbsp;
          </div>
          <div
            class="block"
            style="width: {speechXScale(
              ad.education
            )}px; background: {colors[1]};"
          >
            &nbsp;
          </div>
          <div
            class="block"
            style="width: {speechXScale(
              ad.employability
            )}px; background: {colors[2]};"
          >
            &nbsp;
          </div>
          <div
            class="block"
            style="width: {speechXScale(
              ad.energy_policy
            )}px; background: {colors[3]};"
          >
            &nbsp;
          </div>
          <div
            class="block"
            style="width: {speechXScale(ad.ret)}px; background: {colors[4]};"
          >
            &nbsp;
          </div>

          <div
            class="block"
            style="width: {speechXScale(
              ad.transport
            )}px; background: {colors[5]};"
          >
            &nbsp;
          </div>
          <div
            class="block"
            style="width: {speechXScale(
              ad.welfare
            )}px; background: {colors[6]};"
          >
            &nbsp;
          </div>
          <div
            class="block block-extra block-extra-bold"
            style="width: {speechXScale(
              speechRemainder(ad)
            )}px; background: {colors[7]};"
          >
            {formatNumber.format(speechRemainder(ad))} words
          </div>
        </div>
      {/each}
    </div>
  {/if}
</main>

<style>
  .legend-block {
    padding: 5px 10px;
    margin: 0px;
    color: #000;
    font-weight: 400;
    font-family: 'Roboto Condensed', Arial, Helvetica, sans-serif;
    border: solid 1px #fff;
    text-transform: uppercase;
    font-size: 0.8rem;
  }
  .block {
    padding: 5px;
    background: #eee;
    margin: 0px !important;

    /* margin: 3px; */
    /* border: solid 0px #fff; */
  }
  .block-extra {
    /* margin: none !important; */
    color: #fff;
    padding: 6px;
    transform: translate(0px, -1px);
  }
  .block-extra-bold {
    font-weight: 700;
    font-size: 0.9rem;
  }
  .light {
    color: #fff;
  }
  .legend {
    margin-bottom: 20px;
  }
  .legend * {
    display: inline-block;
  }
  .bar-label {
    /* font-size: 0.9rem; */
    font-weight: 700;
    margin-top: 15px;
    margin-bottom: 5px;
    font-family: 'Roboto Condensed', Arial, Helvetica, sans-serif;
    text-transform: uppercase;
  }
  .words {
    /* border: solid 1px red; */
    margin-top: 50px;
    width: 100%;
    max-width: 900px;
    margin-left: auto;
    margin-right: auto;
    /* text-align: center;  */
  }
  .sona-row {
    /* width: 100%; */
    opacity: 0.6;
    text-align: left;
    /* border: solid 1px #fff; */
  }
  .sona-row:hover {
    opacity: 1;
    cursor: pointer;
  }
  .sona-row * {
    display: inline-block;
  }

  svg {
    overflow: visible;
  }
  main {
    padding-bottom: 100px;
  }
  .page-title {
    width: 90%;

    text-align: center;
    font-weight: 700;
    font-size: 2rem;
    margin-bottom: 10px;
    margin-left: auto;
    margin-right: auto;
  }
  .sub-title {
    width: 90%;
    max-width: 800px;
    margin-left: auto;
    margin-right: auto;
    margin-bottom: 20px;
    text-align: center;
  }
  .charts {
    /* border: solid 1px red; */
    width: 100%;
    max-width: 900px;
    margin-left: auto;
    margin-right: auto;
    /* border: solid 1px gray; */
    /* padding: 10px; */
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    grid-gap: 10px;
  }
  path {
    stroke: indianred;
    stroke-width: 3px;
  }
  .chart-line {
    fill: none;
  }
  .chart-area {
    fill: rgb(211, 211, 211);
    stroke: none;
  }
  .chart {
    border: solid 1px lightgray;
    text-align: center;
    max-width: 110%;
    padding: 10px;

    /* background: rgb(243, 243, 243); */
  }
  .spacer {
    display: none;
  }
  .chart-title {
    padding: 0px 20px 10px 5px;
    font-weight: 700;
    text-transform: uppercase;
    text-align: left;
    font-family: 'Roboto Condensed', Arial, Helvetica, sans-serif;
  }
  text {
    font-size: 0.9rem;
    font-weight: 700;
    fill: #000;
    /* font-weight: 700; */
    text-anchor: end;
    fill: indianred;
  }
  .marker {
    stroke-width: 2px;
    stroke-dasharray: 7, 5;
    stroke: indianred;
  }
  line {
    stroke: lightgray;
    stroke-width: 2px;
  }
  .x-grid {
    stroke: #000;
    shape-rendering: crispEdges;
    stroke-width: 1px;
    stroke-dasharray: 3, 3;
  }
  .x-tick {
    font-size: 0.65rem;
    text-anchor: middle;
    transform: translate(0, 3px);
    font-family: 'Roboto Condensed', Arial, Helvetica, sans-serif;
    fill: #000;
  }
  .word-count {
    font-weight: 400;
    color: gray;
    font-size: 0.9rem;
    margin-top: 5px;
  }
  strong {
    color: #000;
  }

  @media only screen and (max-width: 900px) {
    .charts {
      grid-template-columns: 1fr 1fr;
      max-width: 700px;
    }
  }

  @media only screen and (max-width: 600px) {
    .charts {
      grid-template-columns: 1fr;
      max-width: 400px;
    }
  }

  .chart-container {
    width: 90%;
  }
  .tooltip {
    position: fixed;
    z-index: 300;
    padding: 5px 10px;
    background: #fff;
    border: solid 1px lightgray;
    font-size: 0.9rem;
    -webkit-box-shadow: 1px 1px 5px 0px rgba(0, 0, 0, 0.22);
    -moz-box-shadow: 1px 1px 5px 0px rgba(0, 0, 0, 0.22);
    box-shadow: 1px 1px 5px 0px rgba(0, 0, 0, 0.22);
    background: rgb(49, 49, 49);
    color: #fff;
    top: -1000px;
    left: -1000px;
  }
</style>
