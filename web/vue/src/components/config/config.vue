<template> 
<div class="contain"> 

<h6>Exchange List : Add Available API keys</h6> 

  <apiConfigBuilder v-if="addApiToggle"></apiConfigBuilder> 
<table class="mdl-data-table mdl-js-data-table mdl-shadow--2dp">
  <thead>
    <tr>
      <th class="mdl-data-table__cell--non-numeric">Exchange</th>
      <th>API Key</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr v-for="(market, e) in exchanges" :key="e">
      <td class="mdl-data-table__cell--non-numeric">{{e}}</td>
      <td v-if="apiKeySets.includes(e)"><i class="material-icons mdl-button--accent">check</i></td>
      <td v-else ><i class="material-icons">cancel</i></td>
      <td>
        <button  v-if="apiKeySets.includes(e)" class="mdl-button mdl-js-button mdl-button--colored" @click.prevent="removeApiKey(e)">remove</button>
        <button  v-else class="mdl-button mdl-button--colored mdl-button--raised mdl-js-button mdl-js-ripple-effect" @click="openAddApi" >add</button>
      </td>
    </tr> 
  </tbody>
</table>
 

  
</div> 
</template>




<script>
 
import apiConfigBuilder from './apiConfigBuilder.vue';
import { post } from '../../tools/ajax'; 
import cache from '../../../../state/cache'; 

export default {
  components: {
    apiConfigBuilder
  },
  data: () => {
    return {
      addApiToggle: false,
    }
  },
  methods: {
    openAddApi: function() {
      this.addApiToggle = true;
    },
    removeApiKey: function(exchange) {
      if(!confirm('Are you sure you want to delete these API keys?'))
        return;

      post('removeApiKey', {exchange}, (error, response) => {
        if(error)
          return alert(error);
      });
    }
  },
  computed: { 
    apiKeySets: function() {
      return this.$store.state.apiKeys
    }, 
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
    }
  },
  watch: {
    apiKeySets: function() {
      this.addApiToggle = false;
    }
  }
}
</script>