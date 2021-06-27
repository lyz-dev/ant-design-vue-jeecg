<template>
  <a-card :bordered="false">
    <detail-list>
      <detail-list-item :term="column.title" v-if="column.key != 'rowIndex'" v-for="column in columns"
                        :key="column.dataIndex">{{ showValue(column.dataIndex) }}
      </detail-list-item>
    </detail-list>
    <a-divider style="margin-bottom: 32px"/>

    <div class="title">溯源信息</div>
    <TraceabilityInformationList v-bind:applianceInformationId="$route.params.id">
    </TraceabilityInformationList>

    <div class="emqrcode">
      <button @click="showQRcode">点击分享二维码</button>
      <div id="qrcode" ref="qrcode"></div>
    </div>
    <h3>{{ $route.params.id }}</h3>
  </a-card>
</template>

<script>

import TraceabilityInformationList from '../traceability/TraceabilityInformationList'
import QRCode from "qrcodejs2";
import DetailList from '@/components/tools/DetailList'


const DetailListItem = DetailList.Item
export default {
  name: "Detail",
  components: {
    TraceabilityInformationList,
    DetailListItem,
    DetailList
  },
  data() {
    return {
      myTempObj: {},
      link: window.location.href,
      columns: [
        {
          title: '#',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: "center",
          customRender: function (t, r, index) {
            return parseInt(index) + 1;
          }
        },
        {
          title: '器具名称',
          align: "center",
          dataIndex: 'name'
        },
        {
          title: '型号',
          align: "center",
          dataIndex: 'model'
        },
        {
          title: '生产厂家',
          align: "center",
          dataIndex: 'manufacturer'
        },
        {
          title: '出厂编号',
          align: "center",
          dataIndex: 'serialNumber'
        },
        {
          title: '设备编号',
          align: "center",
          dataIndex: 'equipmentNumber'
        },
        {
          title: '测量范围',
          align: "center",
          dataIndex: 'measurementRange'
        },
        {
          title: '准确度	',
          align: "center",
          dataIndex: 'accuracy'
        },
        {
          title: '配件情况',
          align: "center",
          dataIndex: 'accessoriesSituation'
        },
        {
          title: '所属部门',
          align: "center",
          dataIndex: 'department'
        },
        {
          title: '责任人',
          align: "center",
          dataIndex: 'responsible'
        },
        {
          title: '存放地点',
          align: "center",
          dataIndex: 'storageLocation'
        },
        {
          title: '档案编号',
          align: "center",
          dataIndex: 'fileNumber'
        },
        {
          title: '设备状态',
          align: "center",
          dataIndex: 'deviceStatus'
        },
        {
          title: '曾用编号',
          align: "center",
          dataIndex: 'usedNumber'
        },
        {
          title: '是否为计量器具',
          align: "center",
          dataIndex: 'ifmetering',
          customRender: (text) => (text ? filterMultiDictText(this.dictOptions['ifmetering'], text) : ''),
        },
        {
          title: '备注	',
          align: "center",
          dataIndex: 'remark'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: "center",
          fixed: "right",
          width: 147,
          scopedSlots: {customRender: 'action'}
        }
      ],
    }
  },
  computed: {
    detail: {
      get: function () {
        return this.myTempObj; // 在这里把临时对象的值通过计算属性赋值给页面中用到的对象
      }
    }
  },
  created() {
    this.getInfo(this.$route.params.id);
  },
  methods: {
    getInfo(id) { // 发起get请求
      // let result = {}
      // this.$http.get('/appliance/applianceInformation/byId?id=' + id).then(function (result) {
      //   // 通过 result.body 拿到服务器返回的成功的数据
      //   console.log(result.result)
      //   this.myTempObj = result.result
      //   // // Vue.set(this.detail,result)
      //   // Vue.set(this.detail, 0, result.result)
      // })

      this.$http({
        method: "get",
        url:"/appliance/applianceInformation/byId?id="+id,
      }).then(res => {
          this.myTempObj=res.result ; // 在这里用临时变量接受服务端返回值
        })
    },
    showValue: function (key) {
      const value = this.detail[key];
      return !!value ? value : "--";
    },
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