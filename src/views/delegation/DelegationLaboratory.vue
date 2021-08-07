<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
<!--      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>-->
<!--      <a-button type="primary" icon="download" @click="handleExportXls('订单表')">导出</a-button>-->
<!--      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">-->
<!--        <a-button type="primary" icon="import">导入</a-button>-->
<!--      </a-upload>-->
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        :scroll="{x:true}"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        class="j-table-force-nowrap"
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
            v-else
            :ghost="true"
            type="primary"
            icon="download"
            size="small"
            @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
                    <a @click="showLogs(record)">详情</a>
          <a-divider type="vertical"/>
          <a @click="addLog(record)">评论</a>
            <a-divider v-if="record.status==='0'" type="vertical"/>
          <a v-if="record.status==='0'" @click="confirmMoney(record)">确认价格</a>
          <a-divider v-if="record.status==='3'" type="vertical"/>
          <a v-if="record.status==='3'" @click="changeStatus(record,4)">完成委托</a>

        </span>
      </a-table>
    </div>


    <a-modal
      title="添加评论"
      :visible="visible2"
      :confirm-loading="confirmLoading"
      @ok="handleOk"
      @cancel="handleCancel"
    >
      <a-spin :spinning="confirmLoading">
        <j-form-container :disabled="formDisabled">
          <a-form-model ref="orderLogForm" :model="order" :rules="validatorRules" slot="detail">
            <a-row>
              <a-col :span="24">
                <a-form-model-item label="评论" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="commentString">
                  <a-textarea v-model="order.commentString" rows="4" placeholder="请输入评论"/>
                </a-form-model-item>
              </a-col>
            </a-row>
          </a-form-model>
        </j-form-container>
      </a-spin>
    </a-modal>

    <a-modal
      title="确认价格"
      :visible="visible3"
      :confirm-loading="confirmLoading"
      @ok="handleOk2"
      @cancel="handleCancel2"
    >
      <a-spin :spinning="confirmLoading">
        <j-form-container :disabled="formDisabled">
          <a-form-model ref="orderMoneyForm" :model="order2" :rules="validatorRules2" slot="detail">
            <a-row>
              <a-col :span="50">
                <a-form-model-item label="单价" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="money">
                  <a-input-number v-model="order2.money" placeholder="请输入金额"/>
                </a-form-model-item>
              </a-col>
            </a-row>
            <a-row>
              <a-col>
                <a-form-item>
                  <a-form-item label="受检类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="testingType">
                    <a-radio-group v-model="order2.testingType" default-value="现场" button-style="solid">
                      <a-radio-button value="现场">
                        现场
                      </a-radio-button>
                      <a-radio-button value="带回">
                        带回
                      </a-radio-button>
                    </a-radio-group>
                  </a-form-item>
                </a-form-item>
              </a-col>
            </a-row>
          </a-form-model>
        </j-form-container>
      </a-spin>
    </a-modal>

    <a-drawer
      :title="title"
      :visible="visible"
      :body-style="{ paddingBottom: '80px' }"
      :maskClosable="true"
      :width="drawerWidth"
      placement="right"
      :closable="true"
      style="height: 100%;overflow: auto;padding-bottom: 53px;"
      @close="onClose">
      <a-card :bordered="false">
        <OrderDetail v-if="visible" v-bind:order="order"></OrderDetail>
      </a-card>
      <a-card :bordered="false">
        <OrderComment v-if="visible" v-bind:order="order"></OrderComment>
      </a-card>

    </a-drawer>
  </a-card>
</template>

<script>

import '@/assets/less/TableExpand.less'
import { mixinDevice } from '@/utils/mixin'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import {httpAction} from "../../api/manage";
import OrderDetail from "./OrderDetail";
import OrderComment from "./OrderComment";

export default {
  name: 'DeviceOrderList',
  mixins:[JeecgListMixin, mixinDevice],
  components: {
    OrderDetail,
    OrderComment
  },
  data () {
    return {
      description: '委托单管理页面',
      order:{},
      order2:{},
      visible: false,
      drawerWidth: 850,
      disableSubmit: false,
      labelCol: {
        xs: {span: 24},
        sm: {span: 5},
      },
      wrapperCol: {
        xs: {span: 24},
        sm: {span: 16},
      },
      validatorRules: {
        commentString: [
          {required: true, message: '请输入评论信息!'},
        ],
      },
      validatorRules2: {
        money: [
          {required: true, message: '请输入价格!'},
        ],
        testingType:[
          {required: true, message: '请选择受检类型!'},
        ]
      },
      visible2: false,
      visible3: false,
      confirmLoading2: false,
      confirmLoading: false,
      // 表头
      columns: [
        {
          title: '#',
          dataIndex: '',
          key:'rowIndex',
          width:60,
          align:"center",
          customRender:function (t,r,index) {
            return parseInt(index)+1;
          }
        },
        {
          title:'订单编号',
          align:"center",
          dataIndex: 'id'
        },
        {
          title:'器具名称',
          align:"center",
          dataIndex: 'deviceId_dictText'
        },
        {
          title:'下单企业名称',
          align:"center",
          dataIndex: 'tenantId_dictText'
        },
        {
          title:'订单状态',
          align:"center",
          dataIndex: 'status_dictText'
        },
        {
          title:'价格',
          align:"center",
          dataIndex: 'money'
        },
        {
          title:'备注',
          align:"center",
          dataIndex: 'reamrk'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align:"center",
          fixed:"right",
          width:147,
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: "/deviceorder/deviceOrder/list",
        delete: "/deviceorder/deviceOrder/delete",
        deleteBatch: "/deviceorder/deviceOrder/deleteBatch",
       // exportXlsUrl: "/deviceorder/deviceOrder/exportXls"
        orderAdd: "/delegation/orderLogs/add",
        orderMoney: "/deviceorder/deviceOrder/money",
        orderStatus: "/deviceorder/deviceOrder/status",
      },
      dictOptions:{},
      superFieldList:[],
    }
  },
  created() {
    this.getSuperFieldList();
  },
  computed: {
    importExcelUrl: function(){
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
    },
    title() {
      return "订单详情";
    },
    formDisabled() {
      return this.disabled
    },
  },
  methods: {
    initDictConfig(){
    },
    getSuperFieldList(){
      let fieldList=[];
      fieldList.push({type:'string',value:'delegationId',text:'委托id',dictCode:''})
      fieldList.push({type:'string',value:'deviceId',text:'器具id',dictCode:''})
      fieldList.push({type:'string',value:'tenantId',text:'下单企业id',dictCode:''})
      fieldList.push({type:'string',value:'laboratoryId',text:'商户id',dictCode:''})
      fieldList.push({type:'double',value:'money',text:'价格',dictCode:''})
      fieldList.push({type:'string',value:'reamrk',text:'备注',dictCode:''})
      this.superFieldList = fieldList
    },

    addLog(record) {
      // alert(JSON.stringify(record));
      this.order.orderId = record.id
      this.visible2 = true;
    },
    confirmMoney(data){
      this.order2 = data
      // alert(JSON.stringify(this.order2))
      this.visible3 = true;
    },
    showLogs(data) {
      this.visible = true;
      this.order = data;
      this.resetScreenSize();
    },
    resetScreenSize() {
      let screenWidth = document.body.clientWidth;
      if (screenWidth < 500) {
        this.drawerWidth = screenWidth;
      } else {
        this.drawerWidth = 850;
      }
    },
    onClose() {
      this.visible = false;
    },
    changeStatus(data, status) {
      const that = this;
      this.order2 = data;
      this.order2.status = status;
      httpAction(this.url.orderStatus, this.order2, "post").then((res) => {
        if (res.success) {
          that.$message.success(res.message);
          that.$emit('ok');
          this.order = {};
          this.loadData();
        } else {
          that.$message.warning(res.message);
          this.order = {};
        }
      });

    },
    handleOk(e) {
      const that = this;
      // 触发表单验证
      this.$refs.orderLogForm.validate(valid => {
        if (valid) {
          that.confirmLoading = true;
          httpAction(this.url.orderAdd, this.order, "post").then((res) => {
            if (res.success) {
              that.$message.success(res.message);
              that.$emit('ok');
              this.visible2 = false;
              this.confirmLoading = false;
              this.order = {};
              this.loadData();
            } else {
              that.$message.warning(res.message);
              this.visible2 = false;
              this.confirmLoading = false;
              this.order = {};
            }
          }).finally(() => {
            that.confirmLoading = false;
          });
        }
      });

    },
    handleCancel(e) {
      console.log('Clicked cancel button');
      this.visible2 = false;
    },
    handleOk2(e) {
      const that = this;
      // 触发表单验证
      this.$refs.orderMoneyForm.validate(valid => {
        if (valid) {
          that.confirmLoading = true;
          // alert(JSON.stringify(this.order2))
          httpAction(this.url.orderMoney, this.order2, "post").then((res) => {
            if (res.success) {
              that.$message.success(res.message);
              that.$emit('ok');
              this.visible3 = false;
              this.confirmLoading = false;
              this.order2 = {};
              this.loadData();
            } else {
              that.$message.warning(res.message);
              this.visible3 = false;
              this.confirmLoading = false;
              this.order2 = {};
            }
          }).finally(() => {
            that.confirmLoading = false;
          });
        }
      });
    },
    handleCancel2(e) {
      console.log('Clicked cancel button');
      this.visible3 = false;
    },
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>