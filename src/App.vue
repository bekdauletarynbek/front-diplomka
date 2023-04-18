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
    <div class="flex-grow px-5 py-4 h-full overflow-y-auto" v-loading="loading">
        <div id="XZsurface"></div>
        <div id="Roka"></div>
      <div id="nuxk"></div>
      <div id="nuxy"></div>
      <div id="nuWaterLeft"></div>
      <div id="nuWaterRight"></div>
    </div>
  </div>
  <el-dialog title="История вычислении" v-model="show" top="20px">
    <el-form :model="form" label-position="top">
      <el-form-item label="Model:">
        <el-select v-model="form.typeOfModel" class="w-full">
          <el-option v-for="model in models"  :key="model.id" :value="model.id" :label="model.title"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item v-for="inputs in inputLabels[form.typeOfModel]" :key="inputs.title" :label="inputs.title">
        <el-input v-model="form[inputs.key]"></el-input>
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
      labels: ['AbMetres', 'reliefStartPoint', 'nks', 'heightOfWaterAtLeftSide', 'heightOfWaterAtRightSide'],
      show: false,
      loading: false,
      form: {
        AbMetres: null,
        reliefStartPoint: null,
        typeOfModel: null,
        nks: null,
        heightOfWaterAtLeftSide: null,
        heightOfWaterAtRightSide: null,
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
      ],
      inputLabels: {
        1: [
          {
            title: 'Enter AB (meters)',
            key: 'AbMetres',
          },
          {
            title: 'Enter position of A point from relief start point (meters)',
            key: 'reliefStartPoint'
          },
          {
            title: 'Enter nks - number of segments between MN; 1<nks<=3',
            key: 'nks'
          },
        ],
        2: [
          {
            title: 'Enter AB (meters)',
            key: 'AbMetres',
          },
          {
            title: 'Enter position of A point from relief start point (meters)',
            key: 'reliefStartPoint'
          },
          {
            title: 'Enter nks - number of segments between MN; 1<nks<=3',
            key: 'nks'
          },
          {
            title: 'Heght of the water at left side(m) <   16.6227436',
            key: 'heightOfWaterAtLeftSide'
          },
        ],
        3: [
          {
            title: 'Enter AB (meters)',
            key: 'AbMetres',
          },
          {
            title: 'Enter position of A point from relief start point (meters)',
            key: 'reliefStartPoint'
          },
          {
            title: 'Enter nks - number of segments between MN; 1<nks<=3',
            key: 'nks'
          },
          {
            title: 'Heght of the water at left side(m) <   16.6227436',
            key: 'heightOfWaterAtLeftSide'
          },
          {
            title: 'Heght of the water at the right side(m)>2.00000000and<18.37763026',
            key: 'heightOfWaterAtRightSide'
          }
        ],
        4: [
          {
            title: 'Enter AB (meters)',
            key: 'AbMetres',
          },
          {
            title: 'Enter position of A point from relief start point (meters)',
            key: 'reliefStartPoint'
          },
          {
            title: 'Enter nks - number of segments between MN; 1<nks<=3',
            key: 'nks'
          },
          {
            title: 'Heght of the water at left side(m) <   16.6227436',
            key: 'heightOfWaterAtLeftSide'
          },
          {
            title: 'Heght of the water at the right side(m)>2.00000000and<18.37763026',
            key: 'heightOfWaterAtRightSide'
          }
        ],
        5: [],
        6: [],
        7: []
      },
      colorScale: null
    }
  },
  async mounted() {
    this.createColorScale();
    await this.getData();
    // Age categories
  },
  methods: {
    createColorScale() {
      // Create a color scale
      this.colorScale = anychart.scales.linearColor();
      this.colorScale.colors(['#FF0000', '#FFFF00', '#00FF00']); // Set the desired colors here
    },
    drawCategoryChart(id, {data, xAxis, title}) {
      console.log(data)
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
          useHTML: true,
          crosshair: true,
        },
        legend: {
          enabled: true
        },
        yAxis: {
          title: {
            text: 'Z'
          },
        },
        tooltip: {
          pointFormatter: function() {
            var point = this;
            return '<span style="color:' + point.color + '">\u25CF</span> ' + point.series.name + ': <b>' + point.y  + '</b><br/>';
          },
          crosshair: true, 
          shared: true,
          useHTML: true,
        },
        plotOptions: {
          series: {
            showInLegend: true
          },
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

      chart.colorScale(this.colorScale);
      chart.height(500);
      chart.title(title);
      chart.container(id);
      chart.draw();
    },
    async getData() {
      this.loading = true;
      let {data} = await axios.get('/api/get-response');
      this.loading = false;
      let xXZWater = [];
      for(let val of data.XZwater) {
        if(val[1])
        xXZWater.push(Math.round(val[0]))
      }
      let xXZSurface = [];
      for(let val of data.XZsurface) {
        if(val[1])
          xXZSurface.push(Math.round(val[0]))
      }
      let xElectrodes = [];
      for(let val of data.electrodes) {
        if(val[1])
          xElectrodes.push(Math.round(val[0]))
      }
      let xAxis = [...xXZSurface, ...xXZWater, ...xElectrodes];
      let XZwater = {
        data: data.XZwater,
        color: 'blue',
        name: 'XZSurface'
      };
      let firstChart = {
        data: data.XZsurface,
        color: 'red',
        name: 'XZSurface'
      };
      let secondChart = {
        data: data.electrodes,
        name: 'Electrodes',
        color: 'yellow'
      }
      let rokaChart = {
        data: data.Roka,
        color: 'blue',
        name: 'Roka'
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

      let surfaceDataWaterRight = data.nuWaterRight;
      let matrixWaterRight = [];
      for(let i = 0; i < surfaceDataWaterRight.length; i++) {
        for(let k = 0; k < surfaceDataWaterRight[i].length; k++) {
          matrixWaterRight.push([i, k, surfaceDataWaterRight[i][k]]);
        }
      }
      this.createAnyChart('nuxk', matrix, 'NuxK');
      this.createAnyChart('nuxy', matrixy, 'NuxY');
      this.createAnyChart('nuWaterLeft', matrixWaterLeft, 'NuWaterLeft');
      this.createAnyChart('nuWaterRight', matrixWaterRight, 'NuWaterRight');
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
