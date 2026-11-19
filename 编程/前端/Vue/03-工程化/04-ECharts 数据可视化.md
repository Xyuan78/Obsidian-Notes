---
title: ECharts 数据可视化
tags:
  - 前端
  - Vue
  - ECharts
  - 可视化
---

# ECharts 数据可视化

> [!source] 课程对照
> `vue2026.pdf` 第 150–155 页。

## 安装与引入

```bash
npm install echarts
```

```js
import * as echarts from "echarts";
```

如果项目只需要少量图表类型，也可以使用按需引入来减少包体积。

## 准备图表容器

```vue
<template>
  <div ref="chartRef" class="chart"></div>
</template>

<style scoped>
.chart {
  width: 100%;
  height: 400px;
}
</style>
```

图表容器必须具有明确高度，否则 ECharts 通常无法正确显示。宽度可以使用百分比，但父容器也需要有可计算宽度。

## 初始化图表

```vue
<script setup>
import { onBeforeUnmount, onMounted, ref } from "vue";
import * as echarts from "echarts";

const chartRef = ref(null);
let chart;

onMounted(() => {
  chart = echarts.init(chartRef.value);

  chart.setOption({
    title: {
      text: "销量统计",
    },
    tooltip: {
      trigger: "axis",
    },
    xAxis: {
      type: "category",
      data: ["一月", "二月", "三月"],
    },
    yAxis: {
      type: "value",
    },
    series: [
      {
        name: "销量",
        type: "bar",
        data: [120, 200, 150],
      },
    ],
  });
});

onBeforeUnmount(() => {
  chart?.dispose();
});
</script>
```

## 使用响应式数据更新图表

```js
const chartData = ref({
  labels: [],
  values: [],
});

function updateChart() {
  chart?.setOption({
    xAxis: {
      data: chartData.value.labels,
    },
    series: [
      {
        data: chartData.value.values,
      },
    ],
  });
}
```

接口请求成功后先更新数据，再调用 `setOption()`。不要在同一 DOM 上重复 `echarts.init()`，否则可能产生多个实例和警告。

## 根据数据变化自动刷新

```js
import { watch } from "vue";

watch(
  chartData,
  () => {
    updateChart();
  },
  { deep: true },
);
```

更推荐在数据请求完成的明确位置调用 `updateChart()`，避免无意义的深度监听。

## 响应式尺寸

```js
const resizeObserver = new ResizeObserver(() => {
  chart?.resize();
});

onMounted(() => {
  resizeObserver.observe(chartRef.value);
});

onBeforeUnmount(() => {
  resizeObserver.disconnect();
  chart?.dispose();
});
```

也可以在窗口尺寸变化时调用 `chart.resize()`，但组件被隐藏或容器大小变化时，`ResizeObserver` 通常更可靠。

## 常见图表类型

```js
// 柱状图
series: [{ type: "bar", data: [10, 20, 30] }]

// 折线图
series: [{ type: "line", smooth: true, data: [10, 20, 30] }]

// 饼图
series: [
  {
    type: "pie",
    radius: "60%",
    data: [
      { name: "直接访问", value: 335 },
      { name: "搜索引擎", value: 679 },
    ],
  },
]
```

## 从接口加载数据

```js
async function loadChart() {
  const result = await getStatistics();
  const list = result.data;

  chartData.value.labels = list.map((item) => item.name);
  chartData.value.values = list.map((item) => item.value);

  updateChart();
}
```

应处理：

- 加载状态。
- 空数据。
- 接口错误。
- 数据单位。
- 窗口缩放。
- 组件卸载后的实例销毁。

## 常见错误

- 容器没有高度。
- 直接使用 `document.getElementById()` 操作组件内部 DOM，优先使用模板引用。
- 数据刷新时反复初始化实例。
- 组件卸载时没有 `dispose()`。
- 把大量原始数据直接暴露给图表，未先转换数据。
- 图表颜色没有统一主题，多个页面风格不一致。