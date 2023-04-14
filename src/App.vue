<template>
  <div class="flex h-full" style="height: 100vh">
    <div class="text-left py-3 w-81 border-0 border-r border-solid border-gray-200 h-full flex-shrink-0">
      <div class="border-0 border-b border-solid border-gray-200 px-3 py-3">
        <p class="w-72">Добро пожаловать в первый онлайн систему контроля дамбами</p>
        <el-button type="text">Показать историю вычислении</el-button>
      </div>
      <div class="px-3 py-5 flex flex-col">
      <el-upload>
        <el-button type="primary" round size="small" plain>Выбрать файл для входных данных</el-button>
      </el-upload>
      <el-upload style="margin-top: 20px">
        <el-button type="primary" round size="small" plain>Выбрать второй файл для вычисления</el-button>
      </el-upload>
        <div class="mt-3">
        <el-button @click="show=!show" type="success" size="small" round>Начать вычисление</el-button>
        </div>
      </div>
    </div>
    <div class="flex-grow px-5 py-4 h-full overflow-y-auto">
        <div id="XZsurface"></div>
        <div id="Roka"></div>
      <div id="nuxk"></div>
      <div id="nuxy"></div>
      <div id="nuWaterLeft"></div>
    </div>
  </div>
  <el-dialog title="История вычислении" v-model="show" top="20px">
    <el-form :model="form" label-position="top">
      <el-form-item label="Model:">
        <el-select v-model="form.typeOfModel" class="w-full">
          <el-option v-for="model in models"  :key="model.id" :value="model.id" :label="model.title"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label=" Enter AB (meters)">
        <el-input v-model="form.AbMetres"></el-input>
      </el-form-item>
      <el-form-item label="Enter position of A point from relief start point (meters)">
        <el-input v-model="form.reliefStartPoint"></el-input>
      </el-form-item>
      <el-form-item label=" Enter nks - number of segments between MN; 1<nks<=3">
        <el-input v-model="form.nks"></el-input>
      </el-form-item>
      <el-form-item label="Heght of the water at left side(m) <   16.6227436 ">
        <el-input v-model="form.heightOfWaterAtLeftSide"></el-input>
      </el-form-item>
      <el-button @click="calculateResult" type="success" round size="small">Вычислить</el-button>
    </el-form>
  </el-dialog>
</template>

<script>
import Highcharts from 'highcharts';
import anychart from 'anychart'
import axios from 'axios'

export default {
  name: 'App',
  components: {
  },
  data() {
    return {
      show: false,
      form: {
        AbMetres: null,
        reliefStartPoint: null,
        typeOfModel: null,
        nks: null,
        heightOfWaterAtLeftSide: null,
      },
      xAxis: [],
      models: [
        {title: 'relief', id: 1},
        {title: 'water on the left', id: 2},
        {title: 'two water surfaces', id: 3},
        {title: 'two water surfaces, damb on the base', id: 4},
        {title: 'water on the rigth and base', id: 5},
        {title: 'one water surfaces, damb on the base', id: 6},
        {title: 'two water spaces, leaking and base', id: 7}
      ]
    }
  },
  async mounted() {
    await this.getData();
    // Age categories
  },
  methods: {
    drawCategoryChart(id, {data, xAxis, title}) {
      Highcharts.chart(id, {
        chart: {
          type: 'spline'
        },
        title: {
          text: title
        },
        xAxis: {
          accessibility: {
            description: 'X'
          },
          categories: xAxis.sort((a, b)=>a - b),
        },
        yAxis: {
          title: {
            text: 'Z'
          },
        },
        tooltip: {
          crosshairs: true,
          shared: true
        },
        plotOptions: {
          spline: {
            marker: {
              radius: 4,
              lineColor: '#666666',
              lineWidth: 1
            }
          }
        },
        series: data
      });
    },
    async calculateResult() {
      let data = await axios.post('/api/calculate', {
        ...this.form
      });
      console.log(data);
      this.getData();
      this.show = false;
    },
    async createAnyChart(id, chartData, title) {
      let chart = anychart.surface(chartData);
      chart.colorRange().enabled(true)
          .colorLineSize(20)
          .stroke('#000000')
          .ticks()
          .enabled(true)
          .stroke('#000000')
          .position('inside');

      // Set color scale
      const colorScale = anychart.scales.linearColor();
      colorScale.colors(['#FF0000', '#FFFF00', '#2a9d8f']); // Set the desired colors here
      chart.colorScale(colorScale);
      chart.height(500);
      chart.title(title);
      chart.container(id);
      chart.draw();
    },
    async getData() {
      let {data} = await axios.get('/api/get-response');
      let xAxis = data.XZsurface.map(k=> k[0]);
      let XZwater = {
        data: data.XZwater,
        color: 'blue',
        title: 'XZSurface'
      };
      let firstChart = {
        data: data.XZsurface,
        color: 'red',
        title: 'XZSurface'
      };
      let secondChart = {
        data: data.electrodes,
        title: 'Electrodes',
        color: 'yellow'
      }
      let rokaChart = {
        data: data.Roka,
        color: 'blue',
        title: 'Roka'
      }
      let xAxisRoka = data.Roka.map(k=> k[0]);
      let surfaceData = data.nuxk;
      let matrix = [];
      for(let i = 0; i < surfaceData.length; i++) {
        for(let k = 0; k < surfaceData[i].length; k++) {
          matrix.push([i, k, surfaceData[i][k]]);
        }
      }
      let surfaceDataY = data.nuxy;
      let matrixy = [];
      for(let i = 0; i < surfaceDataY.length; i++) {
        for(let k = 0; k < surfaceDataY[i].length; k++) {
          matrixy.push([i, k, surfaceDataY[i][k]]);
        }
      }
      let surfaceDataWater = data.nuWaterLeft;
      let matrixWaterLeft = [];
      for(let i = 0; i < surfaceDataWater.length; i++) {
        for(let k = 0; k < surfaceDataWater[i].length; k++) {
          matrixWaterLeft.push([i, k, surfaceDataWater[i][k]]);
        }
      }
      this.createAnyChart('nuxk', matrix, 'NuxK');
      this.createAnyChart('nuxy', matrixy, 'NuxY');
      this.createAnyChart('nuWaterLeft', matrixWaterLeft, 'NuWaterLeft');
      console.log(xAxis.length, firstChart.length)
      this.drawCategoryChart('XZsurface', {data: [firstChart, secondChart, XZwater], xAxis, title: 'XZsurface + Electrodes'})
      this.drawCategoryChart('Roka', {data: [rokaChart], xAxis: xAxisRoka, title: 'Roka Chart'})
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.anychart-ui-support {
  height: 500px !important;
}
</style>
