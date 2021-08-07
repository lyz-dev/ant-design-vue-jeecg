<template>
  <div>
    <div class="no-print" style="text-align: left">
      <a-button v-print="'#printContent'" ghost type="primary">打印</a-button>
    </div>
    <section ref="print" id="printContent">
      <table class="signToolTable">
        <tr>
          <th colspan="4" style="text-align:center;font-size:14px;letter-spacing: 2px;padding-top:6px;">设备标识卡</th>
        </tr>
        <tr>
          <th>器具名称:</th>
          <td colspan="3" id="sdName" class="signToolTableTd">
            {{ applianceInformation.name }}
          </td>
        </tr>
        <tr>
          <th>器具型号:</th>
          <td colspan="3" id="sdModel" class="signToolTableTd">
            {{ applianceInformation.model }}
          </td>
        </tr>
        <tr>
          <th>出厂编号:</th>
          <td id="sdFacNum" colspan="2" style="word-break:break-all;" class="signToolTableTd">
            {{ applianceInformation.serialNumber }}
          </td>
          <td rowspan="4">
            <vue-qr :logoSrc="downloadData.icon" :text="downloadData.url" style="height:80px;width:80px;"></vue-qr>
          </td>
        </tr>
        <tr>
          <th>设备编号:</th>
          <td id="sdNum" class="signToolTableTd">
            {{ applianceInformation.equipmentNumber }}
          </td>
        </tr>
        <tr>
          <th>所属部门:</th>
          <td id="sdDept" class="signToolTableTd">{{ applianceInformation.department }}</td>
        </tr>
        <tr>
          <th style="letter-spacing:3px; ">责任人:</th>
          <td id="sdUser" class="signToolTableTd">{{ applianceInformation.responsible }}</td>
        </tr>
        <tr>
          <th>生产厂家:</th>
          <td colspan="3" id="sdFac" class="signToolTableTd">{{ applianceInformation.manufacturer }}</td>
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
  name: "AppliancePrint",
  components: {
    ATextarea,
    ARow,
    ACol,
    vueQr
  },
  props: {
    reBizCode: {
      type: String,
      default: ''
    },
    applianceInformation: {}
  },
  data() {
    return {
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
  created() {

  },
  methods: {}
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

.signToolTableTd {
  width: 65px;
}

.signToolTable {
  font-size: 12px;
  width: 265px;
  height: 181px;
  font-family: "SimHei";
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