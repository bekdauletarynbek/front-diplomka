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
        <el-button type="success" size="small" round>Начать вычисление</el-button>
        </div>
      </div>
    </div>
    <div class="flex-grow px-5 py-4 h-full overflow-y-auto">
        <div id="container"></div>
        <div id="pieChart"></div>
    </div>
  </div>
  <el-dialog title="История вычислении" v-model="show">
    <el-form :model="form" label-position="top">
      <el-form-item label=" Enter AB (meters)">
        <el-input v-model="form.AbMetres"></el-input>
      </el-form-item>
      <el-form-item label="Enter position of A point from relief start point (meters)">
        <el-input v-model="form.reliefStartPoint"></el-input>
      </el-form-item>
      <el-form-item label="Model: 1 - relief, 2 - water on the left, 3 - two water surfaces,
 4- two water surfaces, damb on the base , 5- water on the rigth and base
 6- one water surfaces, damb on the base , 7- two water spaces, leaking and base">
        <el-input v-model="form.typeOfModel"></el-input>
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
      }
    }
  },
  async mounted() {
    await this.getData();
    // Age categories
  },
  methods: {
    drawCategoryChart(id, data) {
      Highcharts.chart('container', {
        chart: {
          type: 'spline'
        },
        title: {
          text: 'Monthly Average Temperature'
        },
        subtitle: {
          text: 'Source: ' +
              '<a href="https://en.wikipedia.org/wiki/List_of_cities_by_average_temperature" ' +
              'target="_blank">Wikipedia.com</a>'
        },
        xAxis: {
          accessibility: {
            description: 'Months of the year'
          }
        },
        yAxis: {
          title: {
            text: 'Temperature'
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
      let data = await axios.post('http://localhost:3000/calculate', {
        ...this.form
      });
      console.log(data);
    },
    async getData() {
      let {data} = await axios.get('http://localhost:3000/api/get-response');
      console.log(data);
      let firstChart = {
        data: data.XZsurface.data,
        color: 'red',
        title: 'XZSurface'
      };
      let secondChart = {
        data: data.electrodes.data,
        title: 'Electrodes',
        color: 'blue'
      }
      this.drawCategoryChart('', [firstChart, secondChart])
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
</style>
