<template> 
  <div class="mdl-layout mdl-js-layout mdl-layout--fixed-header" id="app">
    <header class="mdl-layout__header">
      <div class="mdl-layout__header-row"> 
        <span class="mdl-layout-title" > {{title}} </span>
        <div class="mdl-layout-spacer"></div>
        <nav class="mdl-navigation mdl-layout--large-screen-only">
          <router-link class="mdl-navigation__link" to="/system-trade" @click.native="hideMenu">Trade</router-link>
          <router-link class="mdl-navigation__link" to="/strategy-info" @click.native="hideMenu">Strategy</router-link>
          <router-link class="mdl-navigation__link" to="/config" @click.native="hideMenu">Exchange API</router-link> 
          <router-link class="mdl-navigation__link" to="/home" @click.native="hideMenu">CoinXTT</router-link>    
          <!--router-link class="mdl-navigation__link" to="/live-gekkos" @click.native="hideMenu">Live Bot</router-link>
          <router-link class="mdl-navigation__link" to="/data" @click.native="hideMenu">Data</router-link>
          <router-link class="mdl-navigation__link" to="/backtest" @click.native="hideMenu">Backtest</router-link-->
        </nav> 
      </div>
    </header>
    
    <div class="mdl-layout__drawer"> 
      <span class="mdl-layout-title"> {{title}} </span> 
      <nav class="mdl-navigation"> 
        <router-link class="mdl-navigation__link" to="/system-trade" @click.native="hideMenu">Trade</router-link>
        <router-link class="mdl-navigation__link" to="/strategy-info" @click.native="hideMenu">Strategy</router-link>
        <router-link class="mdl-navigation__link" to="/config" @click.native="hideMenu">Exchange API</router-link> 
        <router-link class="mdl-navigation__link" to="/home" @click.native="hideMenu">CoinXTT</router-link>    
        <!--router-link class="mdl-navigation__link" to="/live-gekkos" @click.native="hideMenu">Live Bot</router-link>
        <router-link class="mdl-navigation__link" to="/data" @click.native="hideMenu">Data</router-link>
        <router-link class="mdl-navigation__link" to="/backtest" @click.native="hideMenu">Backtest</router-link-->
      </nav>
    </div>

    <main class="docs-layout-content mdl-layout__content mdl-color-text--grey-600">
      <div class="content mdl-grid mdl-grid--no-spacing" id="content">
        <router-view></router-view> 
      </div>
    </main>
  </div>
</template>

<script>
require('material-design-lite')
 

export default {
  name: 'app', 
  data: () => {
    return {
      title : "TradingBot"
    }
  },
  methods: {
      hideMenu: function () {
        document.getElementsByClassName('mdl-layout__drawer')[0].classList.remove('is-visible')
        document.getElementsByClassName('mdl-layout__obfuscator')[0].classList.remove('is-visible')
      }
  },
  components: {
  }
}
</script>


<style>
@import url('https://fonts.googleapis.com/icon?family=Material+Icons');
@import url('https://code.getmdl.io/1.3.0/material.red-deep_orange.min.css');

#app {
  display: flex;
  min-height: 100vh;
  flex-direction: column;
}

.fill {
  flex: 1;
}

.text {
  max-width: 500px;
}

input {
  background: none;
  margin-top: 0.5em;
}

.params {
  min-height: 235px;
  line-height: 1.3em;
}

.hr {
  margin-top: 2rem;
  margin-bottom: 2rem;
  height: 10px;
  background-color: rgba(250,250,250,.99);
}

.btn--primary {
  display: inline-block;
  margin-right: 12px;
  margin-bottom: 12px;
  height: 40px;
  padding: 0 18px;
  border-radius: 4px;
  background-color: #3498db;
  text-shadow: 0 1px 3px rgba(36,180,126,.4);
  box-shadow: 0 4px 6px rgba(50,50,93,.11), 0 1px 3px rgba(0,0,0,.08);
  color: #fff;
  text-decoration: none;
  line-height: 40px;
  transition: transform 250ms;
}

.btn--primary:hover {
  transform: translateY(-1px);
  box-shadow: 0 7px 14px rgba(50,50,93,.1), 0 3px 6px rgba(0,0,0,.08);
  background-color: #3498db;
  color: #fff;
  text-decoration: none;
}

.btn--primary:active,
.btn--primary:focus {
  background-color: #3498db;
  color: #fff;
  text-decoration: none;
}

.btn--primary:active {
  transform: translateY(1px);
}
</style>
