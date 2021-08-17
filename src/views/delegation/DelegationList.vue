<template>
  <a-card :bordered="false">

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>

      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>
            删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a
        style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :expandedRowKeys="expandedRowKeys"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange"
        @expand="handleExpand"
      >

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical"/>
          <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
            <a>删除</a>
          </a-popconfirm>
        </span>
        <a-table
          slot="expandedRowRender"
          slot-scope="text"
          :columns="innerColumns"
          :dataSource="innerData"
          size="middle"
          bordered
          rowKey="id"
          :pagination="false"
          :loading="loading"
        >
            <span slot="action2" slot-scope="text,record">
              <a @click="showLogs(record)">详情</a>
              <a-divider type="vertical"/>
              <a @click="addLog(record)">评论</a>
              <a-divider v-if="record.status==='1'" type="vertical"/>
              <a v-if="record.status==='1'" @click="changeStatus(record,2)">确认议价</a>
              <a-divider v-if="record.status==='2'" type="vertical"/>
              <a v-if="record.status==='2'" @click="changeStatus(record,3)">锁定议价</a>
              <a-divider v-if="record.status==='3' || record.status==='4'" type="vertical"/>
              <a v-if="record.status==='3' || record.status==='4'" @click="termina(record)">确认完成</a>
            </span>
        </a-table>
      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <DelegationModal ref="modalForm" @ok="modalFormOk"></DelegationModal>

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
      title="人工终止"
      :visible="visible3"
      :confirm-loading="confirmLoading"
      @ok="handleOk2"
      @cancel="handleCancel2"
    >
      <a-spin :spinning="confirmLoading">
        <j-form-container :disabled="formDisabled">
          <a-form-model ref="orderMoneyForm" :model="order2" :rules="validatorRules2" slot="detail">
            <a-row>
              <a-col :span="24">
                <a-form-model-item label="证书编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="certificateNumber">
                  <a-input v-model="order2.certificateNumber" placeholder="请输入证书编号"></a-input>
                </a-form-model-item>
              </a-col>
              <a-col :span="24">
                <a-form-model-item label="电子证书" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="certificateBackup">
                  <j-upload v-model="order2.certificateBackup"></j-upload>
                </a-form-model-item>
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
import {getAction} from '@/api/manage'
import DelegationModal from "./modules/DelegationModal";
import {JeecgListMixin} from '@/mixins/JeecgListMixin'
import OrderLogsList from "../orderlogs/OrderLogsList";
import OrderLogsModal from "../orderlogs/modules/OrderLogsModal__Style#Drawer";
import {httpAction} from "../../api/manage";
import OrderDetail from "./OrderDetail";
import OrderComment from "./OrderComment";


export default {
  name: "TableDemo",
  mixins: [JeecgListMixin],
  components: {
    OrderLogsModal,
    DelegationModal,
    OrderLogsList,
    OrderDetail,
    OrderComment
  },
  data() {
    return {
      visible: false,
      drawerWidth: 850,
      disableSubmit: false,
      order: {},
      order2: {},
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
      visible2: false,
      visible3: false,
      validatorRules2: {
        certificateNumber: [
          {required: true, message: '请输入证书编号!'},
        ],
      },
      confirmLoading: false,
      // 子表表头
      innerColumns: [
        {
          title: '实验室名称',
          align: "center",
          width: 100,
          dataIndex: 'laboratoryId_dictText',
          key: 'name',
        },
        {
          title: '器具名称',
          align: "center",
          dataIndex: 'deviceId_dictText'
        },
        {
          title: '单价',
          align: "center",
          dataIndex: 'money',
        },
        {
          title: '状态',
          align: "center",
          dataIndex: 'status_dictText',
        },
        {
          title: '备注',
          align: "center",
          dataIndex: 'reamrk',
        },
        {
          title: '操作',
          dataIndex: 'action2',
          align: "center",
          scopedSlots: {customRender: 'action2'},
        }
      ],
      innerData: [],
      expandedRowKeys: [],
      id: ' ',
      description: '委托订单',
      // 列表表头
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
          title: '委托标题',
          align: "center",
          dataIndex: 'title'
        },
        {
          title: '委托日期',
          align: "center",
          dataIndex: 'createTime'
        },
        {
          title: '发起人',
          align: "center",
          dataIndex: 'createBy'
        },
        {
          title: '委托备注',
          align: "center",
          dataIndex: 'content'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: "center",
          scopedSlots: {customRender: 'action'},
        }],
      // 分页参数
      url: {
        list: "/delegation/delegation/list2",
        delete: "/delegation/delegation/delete",
        deleteBatch: "/delegation/delegation/deleteBatch",
        listOrderByDelegationId: "/delegation/delegation/listOrderByDelegationId",
        orderAdd: "/delegation/orderLogs/add",
        orderStatus: "/deviceorder/deviceOrder/status",
        orderLogs: "/delegation/orderLogs/list",
        termina: "/deviceorder/deviceOrder/termina",
      },
    }
  },
  computed: {
    currentId() {
      return this.id;
    },
    title() {
      return "订单详情";
    },
    formDisabled() {
      return this.disabled
    },
  },
  methods: {
    termina(data) {
      this.order2 = data
      // alert(JSON.stringify(this.order2))
      this.visible3 = true;
    },
    addLog(record) {
      // alert(JSON.stringify(record));
      this.order = record
      this.visible2 = true;
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
      this.order = data;
      this.order.status = status;
      httpAction(this.url.orderStatus, this.order, "post").then((res) => {
        if (res.success) {
          that.$message.success(res.message);
          that.$emit('ok');
          this.order = {};
        } else {
          that.$message.warning(res.message);
          this.order = {};
        }
      });
      this.handleExpand(true, this.order);
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
      this.handleExpand(true, this.order);
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
          httpAction(this.url.termina, this.order2, "post").then((res) => {
            if (res.success) {
              that.$message.success(res.message);
              that.$emit('ok');
              this.visible3 = false;
              this.confirmLoading = false;
              this.order2 = {};
            } else {
              that.$message.warning(res.message);
              this.visible3 = false;
              this.confirmLoading = false;
              this.order2 = {};
            }
          }).finally(() => {
            that.confirmLoading = false;
            this.handleExpand(true, this.order2);
          });
        }
      });
    },
    handleCancel2(e) {
      console.log('Clicked cancel button');
      this.visible3 = false;
    },
    handleExpand(expanded, record) {
      this.expandedRowKeys = [];
      this.innerData = [];
      if (expanded === true) {
        this.loading = true;
        this.expandedRowKeys.push(record.id);
        getAction(this.url.listOrderByDelegationId, {
          delegationId: record.id,
          active: true,
          status: "gt 0"
        }).then((res) => {
          if (res.success) {
            this.loading = false;
            this.innerData = res.result.records;
          }
        });
      }
    },
  }
}
</script>
<style scoped>
.ant-card-body .table-operator {
  margin-bottom: 18px;
}

.ant-table-tbody .ant-table-row td {
  padding-top: 15px;
  padding-bottom: 15px;
}

.anty-row-operator button {
  margin: 0 5px
}

.ant-btn-danger {
  background-color: #ffffff
}

.ant-modal-cust-warp {
  height: 100%
}

.ant-modal-cust-warp .ant-modal-body {
  height: calc(100% - 110px) !important;
  overflow-y: auto
}

.ant-modal-cust-warp .ant-modal-content {
  height: 90% !important;
  overflow-y: hidden
}
</style>