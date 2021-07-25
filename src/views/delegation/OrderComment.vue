<template>
  <a-list
    class="comment-list"
    :header="`${logs.length} 条评论`"
    item-layout="horizontal"
    :data-source="logs"
  >
    <a-list-item slot="renderItem" slot-scope="item, index">
      <a-comment :author="item.author">
        <p slot="content">
          {{ item.content }}
        </p>
        <a-tooltip slot="datetime" :title="item.datetime">
          <span>{{ item.datetime }}</span>
        </a-tooltip>
      </a-comment>
    </a-list-item>
  </a-list>
</template>

<script>

import moment from 'moment';
import {httpAction} from '@/api/manage'

export default {
  name: "OrderComment",
  components: {},
  props:{order:{}},
  data() {
    return {
      logs: [],
      moment,
    };
  },
  created() {
    this.initLogs();
  },
  methods:{
    initLogs(){
      // alert(this.order.id);
      httpAction("/delegation/orderLogs/list?orderId="+this.order.id,{},"get").then(res => {
        if (res.success) {
          this.logs = res.result.map((item, index, arr) => {
            let c = {author: item.createBy, content: item.commentString,datetime:item.createTime}
            return c;
          })
          console.log('this.tenantsOptions: ', this.tenantsOptions)
        }
      })
    },
  },

}
</script>

<style scoped>

</style>