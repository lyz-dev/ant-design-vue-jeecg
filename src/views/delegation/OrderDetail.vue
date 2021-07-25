<template>
  <a-descriptions title="">
    <a-descriptions-item label="实验室名称">
      {{ order.laboratoryId_dictText }}
    </a-descriptions-item>
    <a-descriptions-item label="器具名称">
      {{ order.deviceId_dictText }}
    </a-descriptions-item>
    <a-descriptions-item label="单价">
      {{ order.money }}
    </a-descriptions-item>
    <a-descriptions-item label="订单状态">
      {{ order.status_dictText }}
    </a-descriptions-item>
    <a-descriptions-item label="备注">
      {{ order.reamrk }}
    </a-descriptions-item>
  </a-descriptions>
</template>

<script>
export default {
  name: "OrderDetail",
  components: {
  },
  props:{order:{}},
}
</script>

<style scoped>

</style>