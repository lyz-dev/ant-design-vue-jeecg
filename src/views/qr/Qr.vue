<template>
  <a-card :bordered="false" :class="{'abcdefg':true}">
    <div class="no-print" style="text-align: left">
      <a-button v-print="'#printContent'" ghost type="primary">打印</a-button>
    </div>
    <section ref="print" id="printContent" class="abcdefg">
      <vue-qr :logoSrc="downloadData.icon" :text="downloadData.url" :size="200"></vue-qr>
    </section>
  </a-card>
  <!--</page-layout>-->
</template>
<script>
import ACol from "ant-design-vue/es/grid/Col";
import ARow from "ant-design-vue/es/grid/Row";
import ATextarea from 'ant-design-vue/es/input/TextArea'
import vueQr from'vue-qr'


export default {
  components: {
    ATextarea,
    ARow,
    ACol,
    vueQr
  },
  name: 'Qr',
  props:{
    reBizCode:{
      type: String,
      default: ''
    },
    paramId:""

  },
  data(){
    return {
      downloadData: {
        url: 'http://'+window.location.host+'/result/detail/'+this.paramId,
        icon: require("../../../public/logo.png")
      },
      labelCol: {
        xs: { span: 24 },
        sm: { span: 2 },
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 8 },
      },

    }
  },
  created() {

  },
  methods: {

  }
}
</script>
<style scoped>
/*update_begin author:scott date:20191203 for:打印机打印的字体模糊问题 */
* {
  color: #000000!important;
  -webkit-tap-highlight-color: #000000!important;
}
/*update_end author:scott date:20191203 for:打印机打印的字体模糊问题 */

.abcdefg .ant-card-body{
  margin-left: 0%;
  margin-right: 0%;
  margin-bottom: 1%;
  border:0px solid black;
  min-width: 800px;
  color:#000000!important;
}
.explain{
  text-align: left;
  margin-left: 50px;
  color:#000000!important;
}
.explain .ant-input,.sign .ant-input{
  font-weight:bolder;
  text-align:center;
  border-left-width:0px!important;
  border-top-width:0px!important;
  border-right-width:0px!important;
}
.explain div{
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
</style>