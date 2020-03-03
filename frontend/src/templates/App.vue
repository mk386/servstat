<template lang="pug">
    .ui.very.padded.basic.segment
      h1.ui.header ServStat
      table.ui.celled.unstackable.table
        thead
          tr
            th Host
            th CPU / Mem. / Swap
            th GPUs
        tbody
          tr(v-for='stat in stats')
            td
              div(v-if='stat.data')
                h3 {{ stat.data.host }}
                br
                span {{ stat.addr }}
              div(v-else)
                i {{ stat.addr }}
            td
              div(v-if='stat.data')
                .ui.small.reversed.progress(:data-percent='stat.data.cpu.usage')
                  .bar: .progress
                  .label {{ stat.data.cpu.info.brand }} ({{ stat.data.cpu.count }} Cores)
                div
                  span(v-for='percent, i in stat.data.cpu.percent')
                    .ui.basic.very.small.red.label(v-if='percent > 70') {{ percent }}% 
                    .ui.basic.very.small.orange.label(v-else-if='percent > 40') {{ percent }}% 
                    .ui.basic.very.small.yellow.label(v-else-if='percent > 20') {{ percent }}% 
                    .ui.basic.very.small.green.label(v-else) {{ percent }}% 
                hr.light
                .ui.small.reversed.progress(:data-value='stat.data.mem.used' :data-total='stat.data.mem.total')
                  .bar: .progress
                  .label Memory Usage ({{ stat.data.mem.used | size }} / {{ stat.data.mem.total | size }}) 
                .ui.small.reversed.progress(:data-value='stat.data.swap.used' :data-total='stat.data.swap.total')
                  .bar: .progress
                  .label Swap Usage ({{ stat.data.swap.used | size }} / {{ stat.data.swap.total | size }}) 

              div(v-else)
                i Loading...
            td
              div(v-if='stat.data')
                .ui.cards
                  .card(v-for='gpu in stat.data.gpu')
                    .content
                      .header [{{ gpu.index }}] {{ gpu.name }}
                      .meta
                        ul
                          li(v-for='p in gpu.processes')
                            | {{ p.username }} ({{ p.command }}:{{ p.pid }}, {{ p.gpu_memory_usage }}M)
                      .description
                        .ui.tiny.reversed.progress(:data-percent='gpu["utilization.gpu"]')
                          .bar: .progress
                          .label Util. {{ gpu['utilization.gpu'] }}% ({{ gpu['temperature.gpu'] }}°C, fan: {{ gpu['fan.speed'] }})
                        .ui.tiny.reversed.progress(:data-value='gpu["memory.used"]' :data-total='gpu["memory.total"]')
                          .bar: .progress
                          .label Mem. ({{ gpu['memory.used'] }}M / {{ gpu['memory.total'] }}M) 
                      .extra.content
              div(v-else)
                i Loading...
</template>


<script>
import axios from 'axios';
import $ from 'jquery';


function round(n, f=2) {
  let x = (10 ** f);
  return Math.round(n * x) / x;
}

function formatSize(n) {
  const units = ['', 'K', 'M', 'G', 'T'];
  let p = 1024;
  for (let unit of units) {
    if (n < p) return `${round(n)}${unit}`;
    n /= p;
  }
  return `${round(n)}P`;
}


export default {
  created() {
    axios.get('config.json').then(res => {
      for (let [i, link] of res.data.links.entries()) {
        let addr;
        try {
          addr = /https?:\/\/([a-zA-Z0-9\.]+)(:[0-9]+)?(\/.*)?/.exec(link)[1];
        } catch (e) {
          addr = '(unknown address)';
        }
        this.stats.push({ addr });
        this.update(i, link, res.data.interval || 5000);
      }
    }).catch(err => {
      alert('Failed to load config.json');
    });
  },
  data() {
    return {
      stats: []
    };
  },
  methods: {
    update(index, link, interval) {
      axios.get(link).then(res => {
        this.$set(this.stats[index], 'data', res.data);
      }).finally(() => {
        setTimeout(() => {
          this.update(index, link, interval);
        }, interval);
      });
    },
  },
  watch: {
    stats() {
      this.$nextTick(() => {
        $('.ui.progress').progress();
      });
    }
  },
  filters: {
    size(n) {
      return formatSize(n);
    }
  }
};
</script>



<style lang="less" scoped>
hr.light {
  border-top: 1px solid rgba(34,36,38,.1);
  border-bottom: 0px;
  border-left: 0px;
  border-right: 0px;
}

.ui.reversed.progress[data-percent^="1"] .bar,
.ui.reversed.progress[data-percent^="2"] .bar {
  background-color: #66DA81;
}

.ui.reversed.progress[data-percent^="3"] .bar {
  background-color: #B4D95C;
}

.ui.reversed.progress[data-percent^="4"] .bar,
.ui.reversed.progress[data-percent^="5"] .bar {
  background-color: #DDC928;
}

.ui.reversed.progress[data-percent^="6"] .bar {
  background-color: #E6BB48;
}

.ui.reversed.progress[data-percent^="7"] .bar,
.ui.reversed.progress[data-percent^="8"] .bar {
  background-color: #EFBC72;
}

.ui.reversed.progress[data-percent^="9"] .bar,
.ui.reversed.progress[data-percent^="100"] .bar {
  background-color: #D95C5C;
}

/* Indicating Label */

.ui.reversed.progress[data-percent^="1"] .label,
.ui.reversed.progress[data-percent^="2"] .label {
  color: rgba(0, 0, 0, 0.87);
}

.ui.reversed.progress[data-percent^="3"] .label {
  color: rgba(0, 0, 0, 0.87);
}

.ui.reversed.progress[data-percent^="4"] .label,
.ui.reversed.progress[data-percent^="5"] .label {
  color: rgba(0, 0, 0, 0.87);
}

.ui.reversed.progress[data-percent^="6"] .label {
  color: rgba(0, 0, 0, 0.87);
}

.ui.reversed.progress[data-percent^="7"] .label,
.ui.reversed.progress[data-percent^="8"] .label {
  color: rgba(0, 0, 0, 0.87);
}

.ui.reversed.progress[data-percent^="9"] .label,
.ui.reversed.progress[data-percent^="100"] .label {
  color: rgba(0, 0, 0, 0.87);
}

/* Single Digits */

.ui.reversed.progress[data-percent="1"] .bar,
.ui.reversed.progress[data-percent="2"] .bar,
.ui.reversed.progress[data-percent="3"] .bar,
.ui.reversed.progress[data-percent="4"] .bar,
.ui.reversed.progress[data-percent="5"] .bar,
.ui.reversed.progress[data-percent="6"] .bar,
.ui.reversed.progress[data-percent="7"] .bar,
.ui.reversed.progress[data-percent="8"] .bar,
.ui.reversed.progress[data-percent="9"] .bar {
  background-color: #66DA81;
}

.ui.reversed.progress[data-percent="1"] .label,
.ui.reversed.progress[data-percent="2"] .label,
.ui.reversed.progress[data-percent="3"] .label,
.ui.reversed.progress[data-percent="4"] .label,
.ui.reversed.progress[data-percent="5"] .label,
.ui.reversed.progress[data-percent="6"] .label,
.ui.reversed.progress[data-percent="7"] .label,
.ui.reversed.progress[data-percent="8"] .label,
.ui.reversed.progress[data-percent="9"] .label {
  color: rgba(0, 0, 0, 0.87);
}

/* Indicating Success */

.ui.reversed.progress.success .label {
  color: rgb(83, 26, 26);
}

</style>