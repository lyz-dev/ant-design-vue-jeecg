<template>

  <div class="emqrcode">
<!--    <a-button type="primary" @click="showQRcode">生成二维码</a-button>-->
    <a-button v-print="'#qrcode'" ghost type="primary">打印</a-button>

    <div id="qrcode" ref="qrcode"></div>
  </div>
</template>

<script>
import QRCode from "qrcodejs2";

export default {
  name: "Qr",
  data() {
    return {
      link: 'http://'+window.location.host+'/result/detail/'+this.$route.params.id,
    }
  },
  mounted: function () {
    this.showQRcode();
  },
  methods: {
    /**
     * @description 生成二维码
     * @param  {number} qWidth  宽度
     * @param  {number} qHeight  高度
     * @param  {string} qText  二维码内容（跳转连接）
     * @param  {string} qRender 渲染方式（有两种方式 table和canvas，默认是canvas）
     */
    qrcode(qWidth, qHeight, qText, qRender) {
      let qrcode = new QRCode("qrcode", {
        width: qWidth,
        height: qHeight,
        text: qText,
        render: qRender
      });
    },

    /**
     * @description 点击显示二维码
     */
    showQRcode() {
      //二维码初始化 点击一次添加一个二维码
      this.$refs.qrcode.innerHTML = "";
      this.$nextTick(function () {
        this.qrcode(124, 124, this.link, "canvas");
      });
    },
  }
}
</script>

<style scoped>

</style>