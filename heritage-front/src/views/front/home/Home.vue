<template>
  <div style="display: flex; justify-content: center; padding-top: 30px; flex-direction: column">
    <el-card style="width: 100%; height: calc(100vh - 180px); margin-bottom: 10px" id="main"/>
  </div>
</template>

<script setup>
import {reactive, onMounted} from "vue";
import * as echarts from "echarts";
import {useRouter} from "vue-router";  // 使用 vue-router

const router = useRouter();  // 使用 vue-router

const cityData = [
  [1064.65516, 487.06251, '鄂州市', 7],
  [260.64371, 508.18175, '恩施土家族苗族自治州', 14],
  [1154.65644, 427.30465, '黄冈市', 11],
  [1105.93575, 549.22028, '黄石市', 3],
  [732.97044, 372.82661, '荆门市', 8],
  [727.21035, 538.42066, '荆州市', 10],
  [755.77076, 485.62256, '潜江市', 16],
  [403.68575, 292.4295, '神农架', 18],
  [398.64568, 158.7543, '十堰市', 4],
  [858.73223, 255.47083, '随州市', 13],
  [810.73154, 438.34426, '天门市', 17],
  [1000.81425, 441.46415, '武汉市', 2],
  [851.29212, 493.06229, '仙桃市', 15],
  [974.89388, 596.01859, '咸宁市', 12],
  [629.28896, 239.87139, '襄阳市', 6],
  [929.05323, 365.14689, '孝感市', 9],
  [505.6872, 422.50483, '宜昌市', 5],
];

onMounted(() => {
  const chartDom = document.getElementById("main");
  const myChart = echarts.init(chartDom);
  let option;
  const svgPath = new URL('@/assets/img/Map_of_Hubei.svg', import.meta.url).href;

  fetch(svgPath)
      .then((response) => response.text())
      .then((svg) => {
        echarts.registerMap("iceland_svg", {svg: svg});

        option = {
          tooltip: {},
          geo: {
            tooltip: {
              show: true,
            },
            map: "iceland_svg",
            roam: true,
          },
          series: {
            type: "effectScatter",
            coordinateSystem: "geo",
            geoIndex: 0,
            symbolSize: 10,
            itemStyle: {
              color: "#f56c6c",
            },
            encode: {
              tooltip: 2,
            },
            data: cityData.map(city => city.slice(0, 3)),  // 只使用坐标和城市名称
          },
        };

        myChart.setOption(option);

        // 点击事件
        myChart.getZr().on("click", function (params) {
          const pixelPoint = [params.offsetX, params.offsetY];
          const dataPoint = myChart.convertFromPixel({geoIndex: 0}, pixelPoint);

          console.log("点击的坐标:", dataPoint);  // 调试输出点击坐标

          // 比对坐标，允许浮动15的范围
          const matchCity = cityData.find((city) => {
            const [cityX, cityY, cityName, cityId] = city;
            return (
                Math.abs(cityX - dataPoint[0]) <= 15 && Math.abs(cityY - dataPoint[1]) <= 15
            );
          });

          if (matchCity) {
            const heritageId = matchCity[3];  // 获取匹配城市的ID
            console.log("匹配的城市ID:", heritageId);  // 调试输出匹配的城市ID
            router.push(`/front/heritage/${heritageId}`);  // 跳转到对应页面
          } else {
            console.log("未匹配到对应城市");
          }
        });
      })
      .catch((error) => console.error("Error loading SVG:", error));
});
</script>