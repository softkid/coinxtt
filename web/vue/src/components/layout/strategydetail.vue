<template>
<div class="contain py2"> 
<div class="demo-card-wide mdl-card mdl-shadow--2dp">
  <div class="mdl-card__title">
    <h2 class="mdl-card__title-text">ID</h2>
  </div>
  <div class="mdl-card__supporting-text">
    Lorem ipsum dolor sit amet, consectetur adipiscing elit.
    Mauris sagittis pellentesque lacus eleifend lacinia...
  </div>
  <div class="mdl-card__actions mdl-card--border">
    <a class="mdl-button mdl-button--colored mdl-js-button mdl-js-ripple-effect">
      List
    </a>
  </div>
  <div class="mdl-card__menu">
    <button class="mdl-button mdl-button--icon mdl-js-button mdl-js-ripple-effect">
      <i class="material-icons">share</i>
    </button>
  </div>
</div> 
</div>
</template>


<script>

import Vue from 'vue'
import _ from 'lodash'

import { post } from '../../tools/ajax'
import spinner from '../global/blockSpinner.vue'
import chart from '../backtester/result/chartWrapper.vue'
import roundtrips from '../backtester/result/roundtripTable.vue'
import paperTradeSummary from '../global/paperTradeSummary.vue'
// global moment

export default {
  created: function() {
    if(!this.isLoading)
      this.getCandles();
  },
  components: {
    spinner,
    chart,
    paperTradeSummary,
    roundtrips
  },
  data: () => {
    return {
      candleFetch: 'idle',
      candles: false
    }
  },
  computed: {
    stratrunners: function() {
      return this.$store.state.stratrunners;
    },
    data: function() {
      return _.find(this.stratrunners, {id: this.$route.params.id});
    },
    chartData: function() {
      return {
        candles: this.candles,
        trades: this.trades
      }
    },
    trades: function() {
      if(!this.data)
        return [];

      return this.data.trades;
    },
    report: function() {
      if(!this.data)
        return;

      return this.data.report;
    },
    stratName: function() {
      if(this.data)
        return this.data.strat.tradingAdvisor.method;
    },
    stratParams: function() {
      if(!this.data)
        return '';

      let stratParams = Vue.util.extend({}, this.data.strat.params);
      delete stratParams.__empty;

      if(_.isEmpty(stratParams))
        return 'No parameters'

      return JSON.stringify(stratParams, null, 4);
    },
    isLoading: function() {
      if(!this.data)
        return true;
      if(!_.isObject(this.data.firstCandle))
        return true;
      if(!_.isObject(this.data.lastCandle))
        return true;

      return false;
    },
    watchers: function() {
      return this.$store.state.watchers;
    },
    watcher: function() {
      let watch = Vue.util.extend({}, this.data.watch);
      return _.find(this.watchers, { watch });
    },
  },
  watch: {
    'data.lastCandle.start': function() {
      this.candleFetch = 'dirty';
    },
    data: function(val, prev) {
      if(this.isLoading)
        return;

      if(this.candleFetch !== 'fetched' )
        this.getCandles();
    }
  },
  methods: {
    round: n => (+n).toFixed(5),
    humanizeDuration: (n) => window.humanizeDuration(n),
    moment: mom => moment.utc(mom),
    fmt: mom => moment.utc(mom).format('YYYY-MM-DD HH:mm'),
    getCandles: function() {
      this.candleFetch = 'fetching';

      let to = this.data.lastCandle.start;
      let from = this.data.firstCandle.start;
      let candleSize = this.data.strat.tradingAdvisor.candleSize;

      let config = {
          watch: this.data.watch,
          daterange: {
            to, from
          },
          // hourly candles
          candleSize
        };

      post('getCandles', config, (err, res) => {
        this.candleFetch = 'fetched';
        // todo
        if(!res || res.error || !_.isArray(res))
          console.log(res);

        this.candles = res.map(c => {
          c.start = moment.unix(c.start).utc().format();
          return c;
        });
      })
    }
  }
}
</script>

<style>
</style>
