<template>
  <div id="app">
    <view-channels-page/>
  </div>
</template>

<script>
import ViewChannelsPage from './components/ViewChannelsPage.vue'

export default {
  name: 'app',
  components: {
    'view-channels-page': ViewChannelsPage
  }
}
</script>

<style>
@tailwind preflight;
@tailwind components;
@tailwind utilities;
</style>
