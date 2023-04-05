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
      show: true,
      form: {
        AbMetres: null,
        reliefStartPoint: null,
        typeOfModel: null,
        nks: null,
        heightOfWaterAtLeftSide: null,
      }
    }
  },
  mounted() {
    // Age categories
    var categories = [
      '0-4', '5-9', '10-14', '15-19', '20-24', '25-29', '30-34', '35-40', '40-45',
      '45-49', '50-54', '55-59', '60-64', '65-69', '70-74', '75-79', '80+'
    ];

    Highcharts.chart('pieChart', {
      chart: {
        type: 'bar'
      },
      title: {
        text: 'Population pyramid for Somalia, 2021',
        align: 'left'
      },
      subtitle: {
        text: 'Source: <a ' +
            'href="https://countryeconomy.com/demography/population-structure/somalia"' +
            'target="_blank">countryeconomy.com</a>',
        align: 'left'
      },
      accessibility: {
        point: {
          valueDescriptionFormat: '{index}. Age {xDescription}, {value}%.'
        }
      },
      xAxis: [{
        categories: categories,
        reversed: false,
        labels: {
          step: 1
        },
        accessibility: {
          description: 'Age (male)'
        }
      }, { // mirror axis on right side
        opposite: true,
        reversed: false,
        categories: categories,
        linkedTo: 0,
        labels: {
          step: 1
        },
        accessibility: {
          description: 'Age (female)'
        }
      }],
      yAxis: {
        title: {
          text: null
        },
        labels: {
          formatter: function () {
            return Math.abs(this.value) + '%';
          }
        },
        accessibility: {
          description: 'Percentage population',
          rangeDescription: 'Range: 0 to 5%'
        }
      },

      plotOptions: {
        series: {
          stacking: 'normal'
        }
      },

      tooltip: {
        formatter: function () {
          return '<b>' + this.series.name + ', age ' + this.point.category + '</b><br/>' +
              'Population: ' + Highcharts.numberFormat(Math.abs(this.point.y), 1) + '%';
        }
      },

      series: [{
        name: 'Male',
        data: [
          -8.98, -7.52, -6.65, -5.72, -4.85,
          -3.71, -2.76, -2.07, -1.70, -1.47,
          -1.22, -0.99, -0.81, -0.62, -0.41,
          -0.23, -0.15
        ]
      }, {
        name: 'Female',
        data: [
          8.84, 7.42, 6.57, 5.68, 4.83,
          3.74, 2.80, 2.14, 1.79, 1.59,
          1.34, 1.06, 0.83, 0.63, 0.43,
          0.25, 0.19
        ]
      }]
    });


    Highcharts.chart('container', {
      chart: {
        type: 'column'
      },
      title: {
        text: 'World\'s largest cities per 2021'
      },
      subtitle: {
        text: 'Source: <a href="https://worldpopulationreview.com/world-cities" target="_blank">World Population Review</a>'
      },
      xAxis: {
        type: 'category',
        labels: {
          rotation: -45,
          style: {
            fontSize: '13px',
            fontFamily: 'Verdana, sans-serif'
          }
        }
      },
      yAxis: {
        min: 0,
        title: {
          text: 'Population (millions)'
        }
      },
      legend: {
        enabled: false
      },
      tooltip: {
        pointFormat: 'Population in 2021: <b>{point.y:.1f} millions</b>'
      },
      series: [{
        name: 'Population',
        data: [
          ['Tokyo', 37.33],
          ['Delhi', 31.18],
          ['Shanghai', 27.79],
          ['Sao Paulo', 22.23],
          ['Mexico City', 21.91],
          ['Dhaka', 21.74],
          ['Cairo', 21.32],
          ['Beijing', 20.89],
          ['Mumbai', 20.67],
          ['Osaka', 19.11],
          ['Karachi', 16.45],
          ['Chongqing', 16.38],
          ['Istanbul', 15.41],
          ['Buenos Aires', 15.25],
          ['Kolkata', 14.974],
          ['Kinshasa', 14.970],
          ['Lagos', 14.86],
          ['Manila', 14.16],
          ['Tianjin', 13.79],
          ['Guangzhou', 13.64]
        ],
        dataLabels: {
          enabled: true,
          rotation: -90,
          color: '#FFFFFF',
          align: 'right',
          format: '{point.y:.1f}', // one decimal
          y: 10, // 10 pixels down from the top
          style: {
            fontSize: '13px',
            fontFamily: 'Verdana, sans-serif'
          }
        }
      }]
    });

  },
  methods: {
    async calculateResult() {
      let data = await axios.post('http://localhost:3000/calculate', {
        ...this.form
      });
      console.log(data);
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
