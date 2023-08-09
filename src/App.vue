<template>
  <div>
<!--    <G6TestForPaaS></G6TestForPaaS>-->
<!--    <GraphTestForPaaS></GraphTestForPaaS>-->
<!--    <GraphTestForPaaSByLocalJS></GraphTestForPaaSByLocalJS>-->
<!--    <G6test></G6test>-->
    <X6TestForPaaS></X6TestForPaaS>
  </div>
</template>

<script>
import G6test from "@/components/G6test.vue";
import G6TestForPaaS from "@/components/G6TestForPaaS.vue";
import GraphTestForPaaS from "@/components/GraphTestForPaaS.vue";
import X6TestForPaaS from "@/components/X6TestForPaaS.vue";
export default {
  name: 'App',
  components: {
    G6TestForPaaS,
    GraphTestForPaaS,
    G6test,
    X6TestForPaaS
  }
}
</script>

<style lang="less">

</style>
