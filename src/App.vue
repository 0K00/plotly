<template>
  <div id='app' class='container-fluid'>
    <div class="row">
      <div class="col-3">
        <Collapse v-for='(item, index) in this.list' :key='index' :user='item'>
          This is {{item.name}}, {{item.age}} years old
        </Collapse>
      </div>
      <div class="col-9">
        <tabs style="text-transform: capitalize;">
          <tab title='Jour'>
            <div class="d-flex align-items-baseline justify-content-between">
              <button>
                <b-icon-caret-left-fill></b-icon-caret-left-fill>
              </button>
              <p>{{dateMin}}</p>
              <button>
                <b-icon-caret-right-fill></b-icon-caret-right-fill>
              </button>
            </div>
          </tab>
          <tab title='Semaine'>
            <div class="d-flex align-items-baseline justify-content-between">
              <button>
                <b-icon-caret-left-fill></b-icon-caret-left-fill>
              </button>
              <p>{{dateMin}} - {{dateMax}}</p>
              <button>
                <b-icon-caret-right-fill></b-icon-caret-right-fill>
              </button>
            </div>
          </tab>
          <tab title='Mois'>
            <div class="d-flex align-items-baseline justify-content-between">
              <button>
                <b-icon-caret-left-fill></b-icon-caret-left-fill>
              </button>
              <p>{{dateMin}}</p>
              <button>
                <b-icon-caret-right-fill></b-icon-caret-right-fill>
              </button>
            </div>
          </tab>
          <tab title='Année'>
            <div class="d-flex align-items-baseline justify-content-between">
              <button>
                <b-icon-caret-left-fill></b-icon-caret-left-fill>
              </button>
              <p>{{dateMin}}</p>
              <button>
                <b-icon-caret-right-fill></b-icon-caret-right-fill>
              </button>
            </div>
          </tab>
          <!-- <tab title='Tous'>
            <p>{{dateMin}} - {{dateMax}}</p>
          </tab> -->
        </tabs>
        <div id='chart'></div>
      </div>
    </div>
  </div>
</template>

<script>
  import Plotly from 'plotly.js';
  import Tab from './components/Tab.vue'
  import Tabs from './components/Tabs.vue'
  import Collapse from "./components/Collapse.vue";


export default {
  name: 'plotly',
  inheritAttrs: false,
  props: {


  },
  components: {
    Tab,
    Tabs,
    Collapse,
  },

  data() {
    return {
      scheduled: null,
      range: [],
      selectorOptions: undefined,
      dateMin: '',
      dateMax: '',
      data: [],
      list: [
        {name: "ola", age: 16, visible: false},
        {name: "sam", age: 18, visible: false},
        {name: "lop", age: 45, visible: false}
      ],
    };
  },
  mounted() {
    var rawDataURL = 'https://raw.githubusercontent.com/plotly/datasets/master/2016-weather-data-seattle.csv';
    var xField = 'Date';
    var yField = 'Mean_TemperatureC';
    let range = this.range
    console.log(range)

    Plotly.d3.csv(rawDataURL, function(err, rawData) {
        if(err) throw err;

        var data = prepData(rawData);
        let min = Math.min(...data[0].y)
        let max = Math.max(...data[0].y)
        range = [min, max]
        var layout = {
            title: 'Time series with range slider and selectors',
            height: 700,
            xaxis: {
                rangeslider: {}
            },
            yaxis: {
                fixedrange: false,
                range: [min, max],
            }
        };
        var config = {
          scrollZoom: true,
          locale: 'fr',
          displaylogo: false,
          modeBarButtonsToRemove: ['autoScale2d', 'hoverClosestCartesian', 'hoverCompareCartesian'],
          displayModeBar: true,
          responsive: true,
        }

        Plotly.plot('chart', data, layout, config);
        
    });

    function prepData(rawData) {
        var x = [];
        var y = [];

        rawData.forEach(function(datum, i) {
            if(i % 100) return;

            x.push(new Date(datum[xField]));
            y.push(datum[yField]);
        });


        return [{
            mode: 'lines',
            x: x,
            y: y
        }];
    }
  },
  watch: {
    data: {
      handler() {
        this.schedule({ replot: true });
      },
      deep: true
    },
    options: {
      handler(value, old) {
        if (JSON.stringify(value) === JSON.stringify(old)) {
          return;
        }
        this.schedule({ replot: true });
      },
      deep: true
    },
    layout(layout) {
      this.innerLayout = { ...layout };
      this.schedule({ replot: false });
    }
  },


  methods: {
    onResize() {
      Plotly.Plots.resize(this.$el);
    },
    schedule(context) {
      const { scheduled } = this;
      if (scheduled) {
        scheduled.replot = scheduled.replot || context.replot;
        return;
      }
      this.scheduled = context;
      this.$nextTick(() => {
        const {
          scheduled: { replot }
        } = this;
        this.scheduled = null;
        if (replot) {
          this.react();
          return;
        }
        this.relayout(this.innerLayout);
      });
    },
    toImage(options) {
      const allOptions = Object.assign(this.getPrintOptions(), options);
      return Plotly.toImage(this.$el, allOptions);
    },
    downloadImage(options) {
      const filename = `plot--${new Date().toISOString()}`;
      const allOptions = Object.assign(this.getPrintOptions(), { filename }, options);
      return Plotly.downloadImage(this.$el, allOptions);
    },
    getPrintOptions() {
      const { $el } = this;
      return {
        format: 'png',
        width: $el.clientWidth,
        height: $el.clientHeight
      };
    },
    react() {
      Plotly.react(this.$el, this.data, this.innerLayout, this.options);
    },

    resetY() { 
      var layout = {
        title: 'Time series with range slider and selectors',
        xaxis: {
            rangeselector: this.selectorOptions,
            rangeslider: {}
        },
        yaxis: {
            fixedrange: false,
            range: this.range,
        }
      };
      Plotly.update('chart', this.data, layout)
    },

    updateDate(timeline) {
      console.log(this.range)
      let layout = {
         title: 'Time series with range slider and selectors',
        xaxis: {
            rangeselector: this.selectorOptions,
            rangeslider: {},
            range: []
        },
        yaxis: {
            fixedrange: false,
            range: this.range,
        }
      }

      this.selectedDate = timeline
      switch(timeline) {
        case "Jour":  {
          let dayMin = new Date()
          dayMin.setHours(0,0,0,0)
          let dayMax = new Date()
          dayMax.setHours(23,59,59,999)
          this.dateMin = dayMin.toLocaleDateString('default', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })
          this.dateMax = dayMax
          let rangeDay = [dayMin, dayMax]
          layout.xaxis.range = rangeDay
          Plotly.update('chart', this.data, layout)
          break
        }
        case "Semaine": {
          let currWeek = new Date();
          let firstWeek = currWeek.getDate() - currWeek.getDay()
          let lastWeek = firstWeek + 6
          let weekMin = new Date(currWeek.setDate(firstWeek));
          let weekMax = new Date(currWeek.setDate(lastWeek));
          this.dateMin = weekMin.toLocaleDateString('default', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })
          this.dateMax = weekMax.toLocaleDateString('default', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })
          let rangeWeek = [weekMin, weekMax]
          layout.xaxis.range = rangeWeek
          Plotly.update('chart', this.data, layout)
          break
        }
        case "Mois": {
          let dateMonth = new Date(), y = dateMonth.getFullYear(), m = dateMonth.getMonth();
          let monthMin = new Date(y, m, 1);
          let monthMax = new Date(y, m + 1, 0);
          this.dateMin = monthMin.toLocaleDateString('default', { year: 'numeric', month: 'long' })
          this.dateMax = monthMax.toLocaleDateString('default', { year: 'numeric', month: 'long' })
          let rangeMonth = [monthMin, monthMax]
          layout.xaxis.range = rangeMonth
          Plotly.update('chart', this.data, layout)
          break
        }
        case "Année": {
          let yearMin = new Date(new Date().getFullYear(), 0, 1);
          let yearMax = new Date(new Date().getFullYear(), 11, 31)
          this.dateMin = yearMin.toLocaleDateString('default', { year: 'numeric' })
          this.dateMax = yearMax.toLocaleDateString('default', { year: 'numeric' })
          let rangeYear = [yearMin, yearMax]
          layout.xaxis.range = rangeYear
          Plotly.update('chart', this.data, layout)
          break
        }
        // case "Tous": {
        //   console.log(this.data)
        //   let allMin = new Date(Math.min.apply(null, this.data.x));
        //   let allMax = new Date(Math.max.apply(null, this.data.x));
        //   this.dateMin = allMin
        //   this.dateMax = allMax
        //   let rangeAll = [allMin, allMax]
        //   layout.xaxis.range = rangeAll
        //   Plotly.update('chart', this.data, layout)
        //   break
        // }
        default :
      }
    },
    
  },

};
</script>

<style lang='css' scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
