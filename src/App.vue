<template>
  <div id="multiselect">
    <MultiSelect :selectType="'select-multiple'"></MultiSelect>
  </div>
</template>

<script>
import MultiSelect from './components/multi-select'

export default {
  name: 'app',
  components: {
    MultiSelect
  }
}
</script>

<style lang="scss">

</style>
