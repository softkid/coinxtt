<template>
  <section class="contain grd-row">
    <div class="grd-row-col-6-6 p6 " v-html="intro"></div> 
  </section>
</template>

<script>
import marked from '../../tools/marked';

const intro = marked(` 
# TradingBot

      TradingBot is a Bitcoin trading bot.

      1.Backtesting platform
      2.Self Setting of strategy. 
      3.popular Exchange.
# Version Info

      04061512  1.1.1
      04061510  1.0.0 Opening
`); 

export default {
  data: () => {
    return {
      intro
    }
  }
}
</script>
<style> 
</style>