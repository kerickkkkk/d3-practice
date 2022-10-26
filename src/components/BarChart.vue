<template>
  <div>
    <select
      v-model="selectValue"
      @change="changeHandler"
    >
      <option
        value=""
        disabled
      >
        請選擇
      </option>
      <option value="data1">
        Data1
      </option>
      <option value="data2">
        Data2
      </option>
      <option
        value="data5"
        selected
      >
        Data5
      </option>
    </select>
    <hr>
    <h2>實作：</h2>
    <ol>
      <li>動態載入</li>
      <li>顏色深色表示數值比較大</li>
      <li>響應式圖表， y 軸於銀幕寬 500px以下只顯示三筆科度</li>
      <li>tooltip 跟隨滑鼠顯示 bar 資料</li>
      <li>bar 上加上每一個高度的值</li>
    </ol>
    <div ref="canvasDom" />
  </div>
</template>

<script setup>
import * as d3 from 'd3'
import { ref, onMounted, onUnmounted } from 'vue'

const data = {
  data1: [
    {
      height: 100,
      name: 'a'
    }
  ],
  data2: [
    {
      height: 100,
      name: 'a'
    },
    {
      height: 200,
      name: 'b'
    }
  ],
  data5: [
    {
      height: 800,
      name: 'a'
    },
    {
      height: 100,
      name: 'b'
    },
    {
      height: 500,
      name: 'c'
    },
    {
      height: 1000,
      name: 'd'
    },
    {
      height: 200,
      name: 'ffff'
    },
    {
      height: 1100,
      name: 'g'
    },
    {
      height: 1500,
      name: 'h'
    }
  ]
}

const canvasDom = ref(null)
const canvas = ref(null)
const selectValue = ref('')

let height, width, margin, svg, mainGroup, xGroup, yGroup, x, y, tooltips

const rwdChart = () => {
  // 移除 svg
  canvas.value.select('svg').remove()
  // 取得當下寬度
  width = canvas.value.node().getBoundingClientRect().width
  height = width / 1.618 // golden
  // console.log(width, height)
  margin = 50

  svg = canvas.value
    .append('svg')
    .attr('width', width)
    .attr('height', height)
    .style('border', '1px solid black')

  mainGroup = svg
    .append('g')
    .attr('transform', `translate( ${margin}, ${margin})`)
  // .append('text')
  // .text('maingroup text');

  xGroup = svg
    .append('g')
    .attr('transform', `translate( ${margin}, ${height - margin})`)

  yGroup = svg.append('g').attr('transform', `translate(${margin}, ${margin})`)
  x = d3
    .scaleBand()
    .range([0, width - margin * 2])
    .paddingInner(0.2)
    .paddingOuter(0.2)

  y = d3
    .scaleLinear()
    .range([height - margin * 2, 0])
    .nice()

  // tooltips
  tooltips = d3
    .select('body')
    .append('div')
    .style('position', 'absolute')
    .attr('class', 'tooltips')
    .attr('fill', 'pink')
}

const drawSvg = function (data) {
  // 生出對應的大小的顏色
  const gernerateColor = d3
    .scaleSequential()
    .domain([0, d3.max(data, (d) => d.height)])
    .interpolator(d3.interpolateBlues)

  x.domain(data.map((item) => item.name))

  y.domain([0, d3.max(data, (d) => d.height)])

  const { innerWidth: screenWidth } = window
  const xAxis = d3.axisBottom(x)
  const yAxis = d3
    .axisLeft(y)
    .ticks(screenWidth > 500 ? null : 3)
    .tickFormat((d) => `${d}加字`)

  xGroup.call(xAxis)
  yGroup.call(yAxis)

  // console.log(y(200))
  const mainWithData = mainGroup.selectAll('rect').data(data)
  // I. 更新 已存在 的dom
  mainWithData
    .transition()
    .attr('width', x.bandwidth)
    .attr('height', (d) => height - margin * 2 - y(d.height))
    .attr('fill', (d) => gernerateColor(d.height))
    .attr('x', (d) => x(d.name))
    .attr('y', (d) => y(d.height))

  // II. enter 產出 dom 來對應沒有綁定 dom 的資料
  mainWithData
    .enter()
    .append('rect')
    .attr('width', x.bandwidth)
    .attr('height', (d) => height - margin * 2 - y(d.height))
    .attr('fill', (d) => gernerateColor(d.height))
    .attr('x', (d) => x(d.name))
    .attr('y', (d) => y(d.height))
    .on('mouseover', (e, d) => {
      const target = d3.select(e.target)
      target.attr('opacity', 0.5)
      //   const x = target.attr('x')
      //   const y = target.attr('y')
      // const[x, y] = d3.pointer(event);

      // const width = target.attr('width')
      // const height = target.attr('height')
      tooltips
        .html(d.height)
        .transition()
        // .style("background", gernerateColor(d.height))
        .style('opacity', 1)
        .style('left', e.pageX + 'px')
        .style('top', e.pageY + 'px')
    })
    .on('mouseleave', (e, d) => {
      const target = d3.select(e.target)
      target.attr('opacity', 1)
      tooltips
        .transition()
        .style('opacity', 0)
    })

  // III. 沒有資料綁定的 dom 移除
  mainWithData.exit().remove()

  // 如果有exit 後 mouseover 會抓不到...
  //   mainWithData.selectAll("rect").on('mouseover', e =>{

  //     console.log(e.target)
  //   })
  // 加上個 bar 的值
  mainGroup.selectAll('text')
    .data(data)
    .enter()
    .append('text')
    .text(d => d.height)
    .attr('x', (d) => x(d.name) + x.bandwidth() / 2)
    .attr('y', (d) => y(d.height) - 5)
    .attr('text-anchor', 'middle')
}

const changeHandler = (e) => {
  drawSvg(data[selectValue.value])
}

const reszieHandler = () => {
  rwdChart()
  if (selectValue.value) drawSvg(data[selectValue.value])
}

onMounted(() => {
  canvas.value = d3.select(canvasDom.value)
  reszieHandler()
  window.addEventListener('resize', reszieHandler)
})

onUnmounted(() => {
  window.removeEventListener('resize', reszieHandler)
})
</script>

<style lang="scss" scoped>

:deep(.tooltips) {
    position: absolute;
    text-align: center;
    width: 60px;
    height: 28px;
    padding: 2px;
    font-size: 12px ;
    background: lightsteelblue;
    border: none;
    border-radius: 4px;
    pointer-events: none;
}
</style>
