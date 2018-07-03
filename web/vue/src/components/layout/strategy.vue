<template> 
<div class="contain"> 

<h6>Strategy List : Make your investment strategy</h6> 
 
<table class="mdl-data-table mdl-js-data-table mdl-shadow--2dp">
  <thead>
    <tr>
      <th class="mdl-data-table__cell--non-numeric">Strategy</th>
      <th>Rank By Profit</th>
      <th>Comment</th>
    </tr>
  </thead>
  <tbody>
    <tr v-for="(strat,e) in strategies" :key="e" >
      <td class="mdl-data-table__cell--non-numeric">{{strat.name}}</td>
      <td v-if="strat.empty">{{strat.empty}}</td>
      <td v-else ><i class="material-icons">cancel</i></td>
      <td>
        <button class="mdl-button mdl-button--colored mdl-button--raised mdl-js-button mdl-js-ripple-effect" v-on:click='$router.push({path: `strategy-info/${strat.name}`})' >Detail</button>
      </td>
    </tr> 
  </tbody>
</table>
 

  
</div> 
</template>

<script>
import _ from 'lodash'
import { get } from '../../tools/ajax' 
import cache from '../../../../state/cache'; 

export default { 
  data: () => {
    return {
      strategies: [],
      strategy: 'MACD',
      rawStratParams: '',
      rawStratParamsError: false,

      emptyStrat: false,
      stratParams: {}
    }
  },
  created: function () {
    get('strategies', (err, data) => {
        this.strategies = data;

        _.each(this.strategies, function(s) {
          s.empty = s.params === '';
        });

        this.rawStratParams = _.find(this.strategies, { name: this.strategy }).params;
        this.emptyStrat = _.find(this.strategies, { name: this.strategy }).empty;
        this.emitConfig();
    });
  },
  watch: { 
    strategy: function(strat) {
      strat = _.find(this.strategies, { name: strat });
      this.rawStratParams = strat.params;
      this.emptyStrat = strat.empty;

      this.emitConfig();
    }, 
    rawStratParams: function() { this.emitConfig() }
  },
  methods: {
    emitConfig: function() {
      this.parseParams();
      this.$emit('stratConfig', this.config);
    },
    parseParams: function() {
      try {
        this.stratParams = toml.parse(this.rawStratParams);
        this.rawStratParamsError = false;
      } catch(e) {
        this.rawStratParamsError = e;
        this.stratParams = {};
      }
    }
  },
  computed: { 
    config: function() {
      let config = {
        tradingAdvisor: {
          enabled: true,
          method: this.strategy
        }
      }

      if(this.emptyStrat)
        config[this.strategy] = {__empty: true}
      else
        config[this.strategy] = this.stratParams;

      return config;
    }
  }
} 
</script> 