<script>
  let formatNumber = new Intl.NumberFormat()
  let promise = false
  import { tidy, pivotLonger, mutate, arrange, desc } from '@tidyjs/tidy'
  let width = 300
  let height = 400
  $: height = width * 0.3
  $: width = width - 10
  let margin = {
    top: 10,
    right: 5,
    bottom: 10,
    left: 10,
  }
  import * as d3 from 'd3'
  let parseDate = d3.timeParse('%Y-%m-%d')
  let formatDate = d3.timeFormat('%b%y')
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
    .range([height - margin.bottom, margin.top + 10])
    .domain([0, max])

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
</script>

<main>
  <div class="page-title">Words spoken in each of the past 5 SONAs</div>
  <div class="sub-title">
    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Praesent ante
    dolor, tincidunt non tempus a, congue in tortor. Donec lacinia laoreet
    sapien ut rutrum. Morbi feugiat eleifend ligula, vitae mollis erat varius a.
    Nam nec lorem dolor. Suspendisse potenti. Ut vitae dui et est fringilla
    hendrerit.
  </div>
  {#if promise}
    <div class="charts">
      {#each categories as cat, i}
        <div class="chart" bind:clientWidth={width}>
          <div class="chart-title">{getLabel(cat)}</div>
          <svg {width} {height}>
            <path
              class="chart-area"
              d={area_gen(data.filter((c) => c.cat === cat))}
            />
            <path
              class="chart-line"
              d={line_gen(data.filter((c) => c.cat === cat))}
            />
            <line
              x1={margin.left}
              y1={height - margin.bottom}
              x2={width - margin.right}
              y2={height - margin.bottom}
              stroke="gray"
            />

            <line
              class="marker"
              x1={getLabelX(cat) - 5}
              y1={margin.top}
              x2={getLabelX(cat) - 5}
              y2={height - margin.bottom}
            />
            <text x={getLabelX(cat) - 10} y={margin.top + 5}
              >{formatNumber.format(getLabelText(cat))} words</text
            >
            {#each sonas as sona}
              <!-- <line
                y1="0"
                y2={height - margin.bottom}
                x1={xScale(sona)}
                x2={xScale(sona)}
                class="x-grid"
              /> -->

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
  {/if}
</main>

<style>
  svg {
    overflow: visible;
  }
  main {
    padding-bottom: 200px;
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
    width: 90%;
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
    width: 100%;
    border: solid 1px lightgray;
    /* background: rgb(243, 243, 243); */
  }
  .spacer {
    display: none;
  }
  .chart-title {
    padding: 10px 20px;
    font-weight: 700;
    text-transform: uppercase;
  }
  text {
    font-size: 0.9rem;
    fill: #000;
    /* font-weight: 700; */
    text-anchor: end;
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
    font-size: 0.7rem;
    text-anchor: middle;
    transform: translate(0, 3px);
  }

  @media only screen and (max-width: 800px) {
    .charts {
      grid-template-columns: 1fr 1fr;
      max-width: 500px;
    }
  }

  @media only screen and (max-width: 600px) {
    .charts {
      grid-template-columns: 1fr;
      max-width: 300px;
    }
  }
</style>
