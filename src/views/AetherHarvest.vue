<template>
  <div class="aether-harvest-container">
    <iframe 
      src="http://localhost:8081" 
      frameborder="0"
      class="harvest-iframe"
      @load="onIframeLoad"
    ></iframe>
  </div>
</template>

<script>
export default {
  name: 'AetherHarvest',
  data() {
    return {
      loading: true
    }
  },
  methods: {
    onIframeLoad() {
      this.loading = false;
      console.log('AetherHarvest页面加载完成');
    }
  },
  mounted() {
    console.log('AetherHarvest组件已挂载');
  }
}
</script>

<style scoped>
.aether-harvest-container {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  background: #f5f7fa;
  position: relative;
}

.harvest-iframe {
  width: 100%;
  height: 100%;
  border: none;
  display: block;
}
</style>
