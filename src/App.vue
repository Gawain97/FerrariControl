<template>
  <div id="app">
    <Ferrari></Ferrari>
  </div>
</template>
<script>
// import THREE from "./components/Threejs.vue";
import CONTROLS from "./components/Controls.vue";
import CONTROLTEST from './components/ControlTest.vue' 
import CONTROLS3 from './components/Controls3.vue' 
import SHADOW from './components/Shadow.vue' 
import CHECKBOX from './components/CheckBox.vue' 
import FerrariCustomized from './views/FerrariCustomized.vue' 
import Ferrari from './views/Ferrari.vue' 






export default {
  components: {
    CONTROLS,
    CONTROLTEST,
    SHADOW,
    CONTROLS3,
    CHECKBOX,
    FerrariCustomized,
    Ferrari

  },
};
</script>
<style lang="less" scoped>



</style>
