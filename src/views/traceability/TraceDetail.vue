<template>
  <a-descriptions title="溯源信息" bordered>
    <a-descriptions-item label="检测机构">{{ track.testingFacility }}</a-descriptions-item>
    <a-descriptions-item label="检测类型">{{ track.testingType }}</a-descriptions-item>
    <a-descriptions-item label="检测地点">{{ track.detectionLocation }}</a-descriptions-item>
    <a-descriptions-item label="检测日期">{{ track.detectionDate }}</a-descriptions-item>
    <a-descriptions-item label="到期日期">{{ track.dateDue }}</a-descriptions-item>
    <a-descriptions-item label="证书编号">{{ track.certificateNumber }}</a-descriptions-item>
    <a-descriptions-item label="检测项目">{{ track.testItem }}</a-descriptions-item>
    <a-descriptions-item label="检测费">{{ track.testFee }}</a-descriptions-item>
    <a-descriptions-item v-if="track.certificateBackup" label="电子证书">
      <!--      <j-upload v-model="track.certificateBackup"></j-upload>-->
      <a target="_blank" rel="noopener noreferrer"
         title="电子证书"
         :href="this.action+track.certificateBackup"
         class="ant-upload-list-item-name ant-upload-list-item-name-icon-count-1">
        <span><a-icon type="download"/>&nbsp;下载</span>
      </a>
    </a-descriptions-item>
    <a-descriptions-item label="备注">{{ track.remark }}</a-descriptions-item>
  </a-descriptions>
</template>

<script>
export default {
  name: "TraceDetail",
  props: {
    applianceInformationId: {
      type: String,
      required: true
    },
  },
  data() {
    return {
      description: '溯源信息管理页面',
      action: window._CONFIG['domianURL'] + '/sys/common/static/',
      myTempObj: {},
    }
  },
  computed: {
    track: {
      get: function () {
        return this.myTempObj; // 在这里把临时对象的值通过计算属性赋值给页面中用到的对象
      }
    }
  },
  created() {
    this.getInfo(this.applianceInformationId);
  },
  methods: {
    getInfo(id) { // 发起get请求
      this.$http({
        method: "get",
        url: "/traceability/traceabilityInformation/byApplianceInformationIdAndFlag?applianceInformationId=" + id,
      }).then(res => {
        this.myTempObj = res.result; // 在这里用临时变量接受服务端返回值
      })
    },
  }
}
</script>

<style scoped>

</style>