<template>
  <div class="min-h-screen flex items-center justify-center">
    <h1 class="text-center font-sans text-5xl font-bold text-purple">{{ msg }}</h1>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  }
}
</script>
