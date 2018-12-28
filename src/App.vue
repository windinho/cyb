<template>
  <div id="app" class="container">
    <center>

      <h1>Critical Illness Insurance Calculator</h1>
      <span>Adjust the sliders to estimate the coverage amount you may need.</span>
    </center>
    <div class="row">
      <div class="col-md-3">
        <form>
          <div v-for="(n, i) in expenses.length" class="form-group">
            <label>{{ expenses[i].name }}
              <span class="text-secondary">&#9432;</span>
            </label>
            <div class="input-group mb-3">
              <div class="input-group-prepend">
                <span class="input-group-text">$</span>
              </div>
              <input type="number" step="100" v-model.number="expenses[i].value" class="form-control">
              <div v-if="expenses[i].month" class="input-group-append">
                <span class="input-group-text">/month</span>
              </div>
            </div>
            <div class="slidecontainer text-center">
              <input v-model.number="expenses[i].value" type="range" min="0" step="100" max="2500" class="slider">
            </div>
          </div>
          <div class="form-group">
            <label id="add">Additional Info</label>
          </div>
        </form>
      </div>
      <div class="col-md-8 offset-md-1">
        <div class="card col-md-10">
          <div class="form-group row">
            <label class="col-md-4 offset-md-1 col-form-label">
              Recovery period <span class="text-secondary">&#9432;</span>
            </label>
            <div class="col-md-6">
              <div class="input-group mb-3">
                <input v-model.number="months" class="form-control">
                <div class="input-group-append">
                  <span class="input-group-text">months</span>
                </div>
              </div>
              <div class="slidecontainer text-center">
                <input v-model.number="months" type="range" min="6" step="1" max="36" class="slider">
              </div>
            </div>
          </div>
        </div>
        <div id="chart" class="col-md-10">
          <canvas id="myChart" height="200"></canvas>
        </div>
        <div class="col-md-8 offset-md-1 text-center">
          <p class="text-center">
            A serious illness with recovery lasting 12 months could put your finances down by $359,000 today and by $467,000 in 10 years.
          </p>
          <button class="btn btn-sm btn-primary">START COMPARING QUOTES</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import Chart from 'chart.js/dist/Chart.min.js';

  export default {
    name: 'app',
    data () {
      return {
        months: 12,
        expenses: [
        { name: "Desired replacement income (after-tax)", value: 500, month: true },
        { name: "Out-of-pocket healthcare expenses", value: 100 },
        { name: "Home modification expenses", value: 200 },
        { name: "Medical homecare expenses", value: 300, month: true },
        { name: "Other expenses (transport, childcare, etc)", value: 400, month: true },
        ],
        chart: null
      }
    },
    watch: {
      expenses: {
        handler: function (val, oldVal) {
          setTimeout(() => {
            console.log(val, oldVal)
            this.updateChart()
          }, 3000);
        },
        deep: true
      }
    },
    methods: {
      updateChart() {
        var net = 0;
        for (var i = 0; i < this.expenses.length-1; i++) {
          net += Number(this.expenses[i].value);
          // console.log(net)
        }
        var total = net - Number(this.expenses[this.expenses.length-1].value)
        var in10 = total*1.2 / this.months;

        if(this.chart === null) {
          this.drawChart([total, in10]);
        } else {
          this.chart.data.datasets[0].data = [total, in10];
          this.chart.update();
        }
      },
      drawChart(val) {
        var ctx = document.getElementById("myChart").getContext('2d');
        this.chart = new Chart(ctx, {
          type: 'bar',
          data: {
            labels: ["Estimated Cost today", "Estimated Cost in 10 years"],
            datasets: [{
              label: false,
              data: val,
              backgroundColor: [
              '#49DCFA',
              '#00B5E1'
              ],
              borderWidth: 1,
              responsive: true
            }]
          },
          options: {
            legend: {
              display: false
            },
            scales: {
              yAxes: [{
                ticks: {
                  beginAtZero:true,
                  callback: function(value, index, values) {
                    return '$' + value + 'k';
                  }
                },
              }],
              xAxes: [{
                scaleLabel: {
                  labelString: 'Assumptions',
                  display: true,
                  fontColor: '#02B6E1'
                }
              }]
            }
          },
        });
      }
    },
    mounted() {
      this.updateChart()
    }
  }
</script>

<!-- <style lang="scss"> -->
  <style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
    margin-top: 60px;
  }

  .row {
    margin-top: 50px;
  }

  label {
    font-size: 12px;
  }

  #add {
    width: 100%;
  }

  #add:after {
    content: '\229E';
    color: #17B3E4;
    font-size: 16px;
    float: right;
  }

  .input-group-text {
    background: none;
  }

  #chart {
    margin-top: 50px;
  }

  /*Chrome*/
  @media screen and (-webkit-min-device-pixel-ratio:0) {
    input[type='range'] {
      overflow: hidden;
      -webkit-appearance: none;
      outline: 0;
      border: 0;
      cursor: pointer;
    }

    input[type=range]::-moz-focus-outer {
      border: 0;
    }

    input[type='range']::-webkit-slider-runnable-track {
      height: 10px;
      -webkit-appearance: none;
      color: #13bba4;
      margin-top: -1px;
    }

    input[type='range']::-webkit-slider-thumb {
      width: 10px;
      -webkit-appearance: none;
      height: 10px;
      cursor: ew-resize;
      background: #434343;
      box-shadow: -80px 0 0 80px #43e5f7;
    }

  }
  /** FF*/
  input[type="range"]::-moz-range-progress {
    background-color: #43e5f7;
  }
  input[type="range"]::-moz-range-track {
    background-color: #ddd;
  }
  input[type='range']::-moz-range-thumb {
    background: #17B3E4;
    border-color: #17B3E4;
    border-radius: 50%;
    width: 10px;
    height: 10px;
  }
  /* IE*/
  input[type="range"]::-ms-fill-lower {
    background-color: #43e5f7;
  }
  input[type="range"]::-ms-fill-upper {
    background-color: #ddd;
  }

</style>
