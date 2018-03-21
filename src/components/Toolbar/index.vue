<template>
  <div class="schedule-menu">
      <button class="success" @click="save">保存</button>
      <button class="primary" @click="check">其他按钮</button>
      <button class="normal">按钮3</button>
      <button class="normal">按钮4</button>
  </div>
</template>
<script>
export default {
  name: 'Toolbar',
  methods: {
    save () {
      this.$emit('save')
    },
    check () {
      console.log(this.$parent)
    }
  }
}
</script>
