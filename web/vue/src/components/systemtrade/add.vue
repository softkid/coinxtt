<template>
<div class="contain">
  <h6>Make your strategy</h6>
  Select Market<br> 
  <market-picker v-for="(market, e) in exchanges" :key="e" 
    v-on:market="updateMarketConfig"  :only-tradable="isTradebot">
    {{ e }}
  </market-picker>
<br>
  Select Current<br>
  <div class="mdl-grid"> 
  <label style="float:left;height:20px;margin:0;padding:0;" v-for="(curr, e) in currencies" :key="e">
    <input type="radio" name="fc" value="small" />
  </label>
  </div>
<br>
  Select Asset<br>
  <div class="mdl-grid">
  <label style="float:left;height:20px;margin:0;padding:0;" v-for="(ass, e) in assets" :key="e">
    <input type="radio" name="fd" value="small" /> 
  </label>
  </div>

 <br>
  Make Strategy<br>
  case:
  <div class="mdl-grid">
  <label style="float:left;height:20px;margin:0;padding:0;" v-for="(ass, e) in assets" :key="e">
    <input type="radio" name="fe" value="small" /> 
  </label>
  </div>
  candle size:
  <div class="mdl-grid">
  <label style="float:left;height:20px;margin:0;padding:0;" v-for="(ass, e) in assets" :key="e">
    <input type="radio" name="ff" value="small" /> 
  </label>
  </div>
  period:
  <div class="mdl-grid">
  <label style="float:left;height:20px;margin:0;padding:0;" v-for="(ass, e) in assets" :key="e">
    <input type="radio" name="fb" value="small" /> 
  </label>
  </div>

  <button class="a w100--s my1 btn--primary" v-on:click.prevent="start" >Start</button>
</div>
</template>

<!--template lang='jade'>
  div.contain.my2
    h4 :: Add system trading 
    h6 Select market 
    h6 Select current
    h6 Select asset
    h6 Make Strategy

    gekko-config-builder(v-on:config='updateConfig')
    .hr
    .txt--center(v-if='config.valid')
      a.w100--s.my1.btn--primary(href='#', v-on:click.prevent='start') Start
</template-->

<script>

import marketPicker from '../global/configbuilder/marketpicker.vue'
import typePicker from '../global/configbuilder/typepicker.vue'
import stratPicker from '../global/configbuilder/stratpicker.vue'
import paperTrader from '../global/configbuilder/papertrader.vue'

import _ from 'lodash'
import Vue from 'vue'
import { get } from '../../tools/ajax'
import gekkoConfigBuilder from './gekkoConfigBuilder.vue'

export default {
  created: function() {
    get('configPart/candleWriter', (error, response) => {
      this.candleWriter = toml.parse(response.part);
    });
    get('configPart/performanceAnalyzer', (error, response) => {
      this.performanceAnalyzer = toml.parse(response.part);
      this.performanceAnalyzer.enabled = true;
    });
  },
  components: {
    marketPicker,
    typePicker,
    stratPicker,
    paperTrader,
    gekkoConfigBuilder
  },
  data: () => {
    return {
      // defaults:
      market: {},
      range: {},
      type: '',
      strat: {},
      paperTrader: {},
      candleWriter: {},
      performanceAnalyzer: {},
      exchange: 'poloniex',
      currency: 'USDT',
      asset: 'BTC',
      pendingStratrunner: false,
      config: {}
    }
  },
  computed: {
    exchanges: function() {

      let exchanges = Object.assign({}, this.$store.state.exchanges);

      if(_.isEmpty(exchanges))
        return false;

      if(this.onlyTradable) {
        _.each(exchanges, (e, name) => {
          if(!e.tradable)
            delete exchanges[name];
        });
      }

      if(this.onlyImportable) {
        _.each(exchanges, (e, name) => {
          if(!e.importable)
            delete exchanges[name];
        });
      }
      return exchanges;
    }, 
    markets: function() {
      return this.exchanges ? this.exchanges[ this.exchange ] : null;
    }, 
    assets: function() {
      return this.exchanges ? this.exchanges[this.exchange].markets[this.currency] : null;
    }, 
    currencies: function() {
      return this.exchanges ? _.keys( this.exchanges[this.exchange].markets ) : null;
    },
    watchers: function() {
      return this.$store.state.watchers;
    },
    stratrunners: function() {
      return this.$store.state.stratrunners;
    },
    watchConfig: function() {
      let raw = _.pick(this.config, 'watch', 'candleWriter');
      let watchConfig = Vue.util.extend({}, raw);
      watchConfig.type = 'market watcher';
      watchConfig.mode = 'realtime';
      return watchConfig;
    },
    requiredHistoricalData: function() {
      if(!this.config.tradingAdvisor || !this.config.valid)
        return;

      let stratSettings = this.config.tradingAdvisor;
      return stratSettings.candleSize * stratSettings.historySize;
    },
    gekkoConfig: function() {
      var startAt;

      if(!this.existingMarketWatcher)
        return;

      if(!this.requiredHistoricalData)
        startAt = moment().utc().startOf('minute').format();
      else {
        // TODO: figure out whether we can stitch data
        // without looking at the existing watcher
        const optimal = moment().utc().startOf('minute')
          .subtract(this.requiredHistoricalData, 'minutes')
          .unix();

        const available = moment
          .utc(this.existingMarketWatcher.firstCandle.start)
          .unix();

        startAt = moment.unix(Math.max(optimal, available)).utc().format();
      }

      const gekkoConfig = Vue.util.extend({
        market: {
          type: 'leech',
          from: startAt
        },
        mode: 'realtime'
      }, this.config);
      return gekkoConfig;
    },
    existingMarketWatcher: function() {
      const market = Vue.util.extend({}, this.watchConfig.watch);
      return _.find(this.watchers, {watch: market});
    },
    exchange: function() {
      return this.watchConfig.watch.exchange;
    },
    existingTradebot: function() {
      return _.find(
        this.stratrunners.filter(s => s.trader === 'tradebot'),
        { watch: { exchange: this.exchange } }
      );
    },
    availableApiKeys: function() {
      return this.$store.state.apiKeys;
    }
  },
  watch: {
    // start the stratrunner
    existingMarketWatcher: function(val, prev) {
      if(!this.pendingStratrunner)
        return;

      if(val && val.firstCandle && val.lastCandle) {
        this.pendingStratrunner = false;

        this.startGekko((err, resp) => {
          this.$router.push({
            path: `/live-gekkos/stratrunner/${resp.id}`
          });
        });
      }
    }
  },
  methods: {
    updateConfig: function(config) {
      this.config = config;
    },
    start: function() {

      // if the user starts a tradebot we do some
      // checks first.
      if(this.config.type === 'tradebot') {
        if(this.existingTradebot) {
          let str = 'You already have a tradebot running on this exchange';
          str += ', you can only run one tradebot per exchange.';
          return alert(str);
        }

        if(!this.availableApiKeys.includes(this.exchange))
          return alert('Please first configure API keys for this exchange in the config page.')
      }

      // internally a live gekko consists of two parts:
      //
      // - a market watcher
      // - a live gekko (strat runner + (paper) trader)
      //
      // however if the user selected type "market watcher"
      // the second part won't be created
      if(this.config.type === 'market watcher') {

        // check if the specified market is already being watched
        if(this.existingMarketWatcher) {
          alert('This market is already being watched, redirecting you now...');
          this.$router.push({
            path: `/live-gekkos/watcher/${this.existingMarketWatcher.id}`
          });
        } else {
          this.startWatcher((error, resp) => {
            this.$router.push({
              path: `/live-gekkos/watcher/${resp.id}`
            });
          });
        }

      } else {

        if(this.existingMarketWatcher) {
          // the specified market is already being watched,
          // just start a gekko!
          this.startGekko(this.routeToGekko);
          
        } else {
          // the specified market is not yet being watched,
          // we need to create a watcher
          this.startWatcher((err, resp) => {
            this.pendingStratrunner = true;
            // now we just wait for the watcher to be properly initialized
            // (see the `watch.existingMarketWatcher` method)
          });
        }
      }
    },
    routeToGekko: function(err, resp) {
      if(err || resp.error)
        return console.error(err, resp.error);

      this.$router.push({
        path: `/live-gekkos/stratrunner/${resp.id}`
      });
    },
    startWatcher: function(next) {
      post('startGekko', this.watchConfig, next);
    },
    startGekko: function(next) {
      post('startGekko', this.gekkoConfig, next);
    }
  }
}
</script>

<style>
label > input{ /* HIDE RADIO */
  visibility: hidden; /* Makes input not-clickable */
  position: absolute; /* Remove input from document flow */
}
label > input + img{ /* IMAGE STYLES */
  cursor:pointer;
  border:2px solid transparent;
}
label > input:checked + img{ /* (RADIO CHECKED) IMAGE STYLES */
  border:2px solid #f00;
}
</style>
