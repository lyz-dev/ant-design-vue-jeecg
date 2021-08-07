<template>
  <div>
    <div class="no-print" style="text-align: left">
      <a-button v-print="'#printContent'" ghost type="primary">打印</a-button>
    </div>
    <section ref="print" id="printContent">
      <table class="signTraceTable">
        <tr>
          <th colspan="4" style="text-align:center;letter-spacing: 14px;font-size:14px;padding-top: 5px;">合格</th>
        </tr>
        <tr>
          <th style="height:20px">器具名称:</th>
          <td style="height:20px" id="stName" class="traceTableTd" colspan="3">
            {{ applianceInformation.name }}
          </td>
        </tr>
        <tr>
          <th>出厂编号:</th>
          <td id="stFacNum" class="traceTableTd ">
            {{ applianceInformation.serialNumber }}
          </td>
        </tr>
        <tr>
          <th>设备编号:</th>
          <td id="stDevNum" class="traceTableTd ">
            {{ applianceInformation.equipmentNumber }}
          </td>
        </tr>
        <tr>
          <th>检测类型:</th>
          <td id="stType" class="traceTableTd">
            {{ track.testingType }}
          </td>
          <td rowspan="4">
            <vue-qr :logoSrc="downloadData.icon" :text="downloadData.url" style="height:65px;width:65px;"></vue-qr>
          </td>
        </tr>
        <tr>
          <th>检测日期:</th>
          <td id="stDate" class="traceTableTd">
            {{ track.detectionDate }}
          </td>
        </tr>
        <tr>
          <th>到期日期:</th>
          <td id="stExpire" class="traceTableTd">
            {{ track.dateDue }}
          </td>
        </tr>
      </table>
    </section>
  </div>
</template>

<script>

import ACol from "ant-design-vue/es/grid/Col";
import ARow from "ant-design-vue/es/grid/Row";
import ATextarea from 'ant-design-vue/es/input/TextArea'
import vueQr from 'vue-qr'

export default {
  name: "TracePrint",
  components: {
    ATextarea,
    ARow,
    ACol,
    vueQr
  },
  props: {
    applianceInformation: {}
  },
  data() {
    return {
      description: '溯源信息管理页面',
      myTempObj: {},
      dictOptions: {},
      superFieldList: [],

      downloadData: {
        url: 'http://' + window.location.host + '/result/detail/' + this.applianceInformation.id,
        icon: require("../../../public/logo.png")
      },
      labelCol: {
        xs: {span: 24},
        sm: {span: 2},
      },
      wrapperCol: {
        xs: {span: 24},
        sm: {span: 8},
      },
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
    this.getInfo(this.applianceInformation.id);
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
/*update_begin author:scott date:20191203 for:打印机打印的字体模糊问题 */
* {
  color: #000000 !important;
  -webkit-tap-highlight-color: #000000 !important;
}

/*update_end author:scott date:20191203 for:打印机打印的字体模糊问题 */

.abcdefg .ant-card-body {
  margin-left: 0%;
  margin-right: 0%;
  margin-bottom: 1%;
  border: 0px solid black;
  min-width: 800px;
  color: #000000 !important;
}

.explain {
  text-align: left;
  margin-left: 50px;
  color: #000000 !important;
}

.explain .ant-input, .sign .ant-input {
  font-weight: bolder;
  text-align: center;
  border-left-width: 0px !important;
  border-top-width: 0px !important;
  border-right-width: 0px !important;
}

.explain div {
  margin-bottom: 10px;
}

/* you can make up upload button and sample style by using stylesheets */
.ant-upload-select-picture-card i {
  font-size: 32px;
  color: #999;
}

.ant-upload-select-picture-card .ant-upload-text {
  margin-top: 8px;
  color: #666;
}

.signTraceTable {
  font-size: 12px;
  width: 225px;
  height: 160px;
  font-family: "SimHei";
}

.signTraceTable th {
  text-align: right;
  width: 60px;

}

table {
  display: table;
  border-collapse: separate;
  box-sizing: border-box;
  text-indent: initial;
  border-spacing: 2px;
  border-color: grey;
}
</style>