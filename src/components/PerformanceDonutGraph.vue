<template>
  <div class='graph'>
    <template v-if="graphs.length">
      <div class='legend'>
        <div class="stat">
          <label>Prev <input type="checkbox" v-model="includePrevious" /></label>
          <p>Good: </p>
          <p v-if="includePrevious" class="prev">prev</p>
          <p>Bad: </p>
          <p v-if="includePrevious" class="prev">prev</p>
          <p>Total: </p>
          <p v-if="includePrevious" class="prev">prev</p>
        </div>
        <div class='stat' v-for="ele in legend" :key="ele.id">
          <label>{{ ele.key }}</label>
          <template v-for="e in ele.value" :key="e.key">
            <p :class="[{ prev: e.key.includes('-prev') }]" :style="{ color: e.color }"
              @mouseenter="hovered(ele.key + '-' + e.key + '-graph')"
              @mouseleave="hovered(ele.key + '-' + e.key + '-graph')" :id="ele.key + '-' + e.key">{{ e.val
              }}</p>
          </template>
        </div>
      </div>
      <div class="donut-wrapper">
        <svg v-for="(graph, index) in graphs" xmlns='http://www.w3.org/2000/svg' :key="index">
          <g v-for="ele in graph" :key="ele.id">
            <circle v-if="ele.type === 'circle'" @mouseenter="hovered(ele.key)" @mouseleave="hovered(ele.key)"
              :id="ele.id" :stroke="ele.stroke" :stroke-width="ele.strokeWidth" fill='none'
              xmlns='http://www.w3.org/2000/svg' :cx="ele.cx" :cy="ele.cy" :r="ele.r" />
            <path v-if="ele.type === 'path'" @mouseenter="hovered(ele.key)" @mouseleave="hovered(ele.key)" :id="ele.id"
              :stroke="ele.stroke" :stroke-width="ele.strokeWidth" fill='none' xmlns='http://www.w3.org/2000/svg'
              :d="ele.d" />
          </g>
        </svg>
      </div>
    </template>
    <p v-else>No Statistics available</p>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { IvoleyStats, IgraphElement, IgraphLegend, IabititiStat, Istat } from '@/components/interfaces'
interface Idata {
  key: string,
  value: Istat,
}
const props = defineProps<{ stats: IvoleyStats | undefined }>()
const settings = { centerX: 125, centerY: 125, radius: 100, strokeWidth: 20 }

const includePrevious = ref(false)

const data = computed(() => (props.stats) ? props.stats as IvoleyStats : null)

const abilityColor = (ability: string): string => {
  const abi: [string, string][] = [
    ['attack-good', '#F6BB42'], ['attack-bad', '#b2811f'],
    ['recieve-good', '#8CC152'], ['recieve-bad', '#689e2d'],
    ['set-good', '#3BAFDA'], ['set-bad', '#1d83a8'],
    ['block-good', '#967ADC'], ['block-bad', '#6142af',],
    ['serve-good', '#DA4453'], ['serve-bad', '#a6222f']
  ]
  const r = new Map(abi).get(ability)
  return r !== undefined ? r : '#fff'
}
function flaterData(data: IvoleyStats, isCurrent: boolean): Idata[] {
  return Object.entries(data).map(([key, value]: [string, IabititiStat]) => ({ key: key, value: isCurrent ? value.current : value.prev }))
}

const graphs = computed(() => {
  const { centerX, centerY, radius, strokeWidth } = settings
  const graphList: IgraphElement[][] = []

  if (data.value === null) return graphList

  const currData = flaterData(data.value, true)
  graphList.push(generateGraph(currData, centerX, centerY, radius, strokeWidth, false))

  if (includePrevious.value) {
    const prevData = flaterData(data.value, false)
    graphList.push(generateGraph(prevData, centerX, centerY, radius - (strokeWidth * 2), strokeWidth, true))
  }
  return graphList
})
const legend = computed(() => {
  if (data.value === null) return []

  const currData = flaterData(data.value, true)
  const leg = generateLegend(currData)

  if (includePrevious.value) {
    const prevData: Idata[] = flaterData(data.value, false)
    leg.map((e, index) => {
      e.value = e.value.reduce((accum, curr) => {
        accum.push(curr)
        accum.push({ key: curr.key + '-prev', val: prevData[index].value[curr.key as keyof Istat] })
        return accum
      }, [] as { key: string; val: number; color?: string; }[])
    })
  }

  return leg
})

const generateLegend = (data: Idata[]): IgraphLegend[] => {
  const legend: IgraphLegend[] = []
  const abi = ['good', 'bad', 'total']
  if (!data.length) return legend

  return data.map((e) => {
    const v = abi.reduce((accum, curr) => {
      (curr === 'total')
        ? accum.push({ key: curr, val: e.value[curr as keyof Istat] })
        : accum.push({ key: curr, val: e.value[curr as keyof Istat], color: abilityColor(e.key + '-' + curr) })
      return accum
    }, [] as { key: string; val: number; color?: string; }[])

    return {
      key: e.key,
      value: v
    }
  })
}

const generateGraph = (data: Idata[], centerX: number, centerY: number, radius: number, strokeWidth: number, isPrev: boolean) => {
  const graph: IgraphElement[] = []

  if (!data.length) return graph

  const totalCumulative = data.reduce((accum, curr) => accum += curr.value.total, 0);

  let count = 0
  let beg = 0
  let end = 0
  data.map((e, index) => {
    const { total, ...val } = e.value

    Object.entries(val).map(([key, pr]) => {
      let percent = 0
      if (totalCumulative !== 0) percent = roundToDecimals(((pr) / totalCumulative) * 100)

      count = roundToDecimals(count + percent)
      if ((index == data.length) && (count < 100)) percent += (100 - count)

      end = roundToDecimals(beg + ((360 / 100) * percent))

      const element = createElement({ key: e.key + '-' + key, value: pr }, beg, end, { x: centerX, y: centerY }, radius, strokeWidth, isPrev)
      if (element) graph.push(element)

      beg = end
    })
  })
  return graph

}

function arcRadius(cx: number, cy: number, radius: number, degrees: number) {
  const radians = (degrees - 90) * Math.PI / 180.0;
  return { x: cx + (radius * Math.cos(radians)), y: cy + (radius * Math.sin(radians)) }
}
function createElement(data: { key: string, value: number }, beg: number, end: number, c: { x: number, y: number }, radius: number, strokeWidth: number = 50, isPrev: boolean = false): IgraphElement | null {
  const id = `${data.key}${isPrev ? '-prev' : ''}-graph`

  if (end === 0) return null

  const newElement = { key: data.key + (isPrev ? '-prev' : ''), id: id, stroke: abilityColor(data.key), strokeWidth: strokeWidth }

  if (beg === 0 && end === 360) return { ...newElement, type: 'circle', cx: c.x, cy: c.y, r: radius }

  const b = arcRadius(c.x, c.y, radius, end)
  const e = arcRadius(c.x, c.y, radius, beg)
  const la = (end - beg) <= 180 ? 0 : 1
  return {
    ...newElement,
    type: 'path',
    d: `M ${roundToDecimals(b.x)} ${roundToDecimals(b.y)} A ${radius} ${radius} 0 ${la} 0 ${roundToDecimals(e.x)} ${roundToDecimals(e.y)}`
  }
}
function roundToDecimals(x: number): number {
  return Math.round((x + Number.EPSILON) * 100) / 100
}

function hovered(id: string) {
  const ele = document.getElementById(id)
  if (ele) ele.classList.toggle('selected')
}
</script>

<style scoped>
.graph {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: var(--card-bg-color);
  grid-area: graph;
  padding: 8px 4px;
}

.graph .donut-wrapper {
  width: 250px;
  height: 250px;
  display: block;
  position: relative;
}

.graph svg {
  width: 250px;
  height: 250px;
  display: block;
  margin: 0 auto;
  position: absolute;
  /* makes it transparent for mouse */
  pointer-events: none;
}

.graph svg path,
.graph svg circle {
  pointer-events: fill;
  cursor: pointer;
}

.graph svg path.selected,
.graph svg circle.selected {
  stroke-width: 30;
}

.graph .select {
  margin-bottom: 1em;
  align-self: flex-end;
}

.graph fieldset {
  display: flex;
  justify-content: space-between;
  gap: 15px;
}

.graph .legend {
  display: flex;
  flex-wrap: wrap;
  text-transform: capitalize;
  justify-content: center;
  text-transform: capitalize;
}

.graph .legend .stat {
  text-align: center;
  transition: transform .2s ease-in-out;
}

.graph .legend .stat label {
  color: var(--text-color);
  padding: 0 1ch;
}

.graph .legend .stat p {
  margin: 0;
  cursor: pointer;
}

.graph .legend .stat p:nth-child(2n + 1) {
  background-color: var(--position-bg-color);
  color: var(--text-color);
}

.graph .legend .stat p.prev {
  font-weight: 100;
  font-size: 0.7em;
}

.graph .legend .stat p.selected {
  font-weight: 900;
}
</style>
