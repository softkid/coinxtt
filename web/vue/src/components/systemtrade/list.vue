<template lang='jade'>
  .contain.py2
    
    .text(v-if='!stratrunners.length')
      p You are currently not running any strategies.
    table.full.mdl-data-table.mdl-js-data-table.mdl-shadow--2dp(v-if='stratrunners.length')
      thead
        tr.mdl-data-table__cell--non-numeric
          th exchange
          th currency-asset 
          th last update
          th duration
          th strategy
          th profit
      tbody
        tr.mdl-data-table__cell--non-numeric.clickable(v-for='gekko in stratrunners', v-on:click='$router.push({path: `system-trade/stratrunner/${gekko.id}`})')
          td {{ gekko.watch.exchange }}
          td {{ gekko.watch.currency }}-{{ gekko.watch.asset }} 
          td
            template(v-if='gekko.lastCandle') {{ fmt(gekko.lastCandle.start) }}
          td
            template(v-if='gekko.firstCandle && gekko.lastCandle') {{ timespan(gekko.lastCandle.start, gekko.firstCandle.start) }}
          td {{ gekko.strat.name }}
          td
            template(v-if='!gekko.report') 0
            template(v-if='gekko.report') {{ round(gekko.report.profit) }} {{ gekko.watch.currency }}
    .hr 
    router-link.btn--primary(to='/system-trade/add') Add your startegy !!
</template>

<script>

import marked from '../../tools/marked'
// global moment
// global humanizeDuration

const text = marked(`

## Live Gekko

Run your strategy against the live market!

`);

export default {
  data: () => {
    return { 
      text
    }
  },
  created: function() {
    this.timer = setInterval(() => {
      this.now = moment();
    }, 1000)
  },
  destroyed: function() {
    clearTimeout(this.timer);
  },
  data: () => {
    return {
      text,
      timer: false,
      now: moment()
    }
  },
  computed: {
    stratrunners: function() {
      return this.$store.state.stratrunners
    },
    watchers: function() {
      return this.$store.state.watchers
    }
  },
  methods: {
    humanizeDuration: (n) => window.humanizeDuration(n),
    moment: mom => moment.utc(mom),
    fmt: mom => moment.utc(mom).format('YYYY-MM-DD HH:mm'),
    round: n => (+n).toFixed(3),
    timespan: function(a, b) {
      return this.humanizeDuration(this.moment(a).diff(this.moment(b)))
    }
  }
}
</script>

<style>
table.clickable {
  border-collapse: separate;
}

tr.clickable td:nth-child(1) {
  padding-left: 5px;
}

tr.clickable {
  cursor: pointer;
}
tr.clickable:hover {
  background: rgba(216,216,216,.99);
}
</style>
