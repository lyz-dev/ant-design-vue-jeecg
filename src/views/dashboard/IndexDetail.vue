<template>
  <a-card title="政策信息">
    <a-card v-for="information in informationList"
            v-bind:key="information.id"
            :style="{ marginTop: '16px' }">
     <div v-html="information.policyLink"></div>
    </a-card>
  </a-card>
</template>
<script>
export default {
  name: 'IndexDetail',
  data() {
    return {
      informationList:[]
    }
  },
  mounted() {
    if (!this.loaded) {
      this.list();
      console.log(this.informationList)
    }
  },
  methods: {
    list:function(){
      this.$http.get("/policy/policyInformation/list?column=istop,updateTime&order=desc&field=id,,,policyLink,istop,active,action&pageNo=1&pageSize=50&active=1").then((resp)=> {
        if (resp.code === 200) {
          for (let i = 0; i < resp.result.records.length; i++) {
            this.informationList.push(resp.result.records[i])
          }
        }
      })
    }
  }
}
</script>
<style></style>
