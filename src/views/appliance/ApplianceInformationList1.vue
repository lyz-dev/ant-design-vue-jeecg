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
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('器具信息')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl"
                @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal"
                     @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <!--          <a-menu-item key="1" @click="batchDel">-->
          <!--            <a-icon type="delete"/>-->
          <!--            删除-->
          <!--          </a-menu-item>-->
          <a-menu-item key="1" @click="batchDelegate">
            <a-icon type="form"/>
            发起委托
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
          <img v-else :src="getImgView(text)" height="25px" alt=""
               style="max-width:80px;font-size: 12px;font-style: italic;"/>
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
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                  <a @click="showDrawer(record)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </a-table>
    </div>


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
        <detail-list>
          <detail-list-item :term="column.title" v-if="column.key != 'rowIndex'" v-for="column in columns"
                            :key="column.dataIndex">{{ showValue(column.dataIndex) }}
          </detail-list-item>
        </detail-list>
        <a-divider style="margin-bottom: 32px"/>

        <div class="title">溯源信息</div>
        <TraceabilityInformationList v-if="visible" v-bind:applianceInformationId="record.id"
                                     v-bind:visibility="visible">
        </TraceabilityInformationList>
      </a-card>
      <a-button type="primary">
        <router-link target="_blank" :to="{path:'/Qr/'+record.id}">生成二维码</router-link>
      </a-button>

    </a-drawer>

    <appliance-information-modal ref="modalForm" @ok="modalFormOk"></appliance-information-modal>

    <a-modal
      title="增加委托消息"
      :visible="visible2"
      :confirm-loading="confirmLoading"
      @ok="handleOk"
      @cancel="handleCancel"
    >
      <a-spin :spinning="confirmLoading">
        <j-form-container :disabled="formDisabled">
          <a-form-model ref="delegationForm" :model="delegation" :rules="validatorRules" slot="detail">
            <a-row>
              <a-col :span="24">
                <a-form-model-item label="委托计划标题" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="title">
                  <a-input v-model="delegation.title" placeholder="请输入委托计划标题"></a-input>
                </a-form-model-item>
              </a-col>
              <a-col :span="24">
                <a-form-model-item label="器具id" :labelCol="labelCol" :wrapperCol="wrapperCol"
                                   prop="applianceinformationid">
                  <a-input v-model="delegation.applianceinformationid" placeholder="请输入器具id"></a-input>
                </a-form-model-item>
              </a-col>
              <a-col :span="24">
                <a-form-model-item label="实验室" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="laboratoryId">
                  <j-multi-select-tag v-model="delegation.laboratoryId"
                                      :options="tenantsOptions"
                                      placeholder="请选择实验室">
                    {{ tenantsOptions }}
                  </j-multi-select-tag>
                </a-form-model-item>
              </a-col>
              <a-col :span="24">
                <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
                  <a-input v-model="delegation.remark" placeholder="请输入备注"></a-input>
                </a-form-model-item>
              </a-col>
            </a-row>
          </a-form-model>
        </j-form-container>
      </a-spin>
    </a-modal>
  </a-card>
</template>

<script>

import '@/assets/less/TableExpand.less'
import {mixinDevice} from '@/utils/mixin'
import {JeecgListMixin} from '@/mixins/JeecgListMixin'
import ApplianceInformationModal from './modules/ApplianceInformationModal'
import {filterMultiDictText} from '@/components/dict/JDictSelectUtil'
import PageLayout from '@/components/page/PageLayout'
import STable from '@/components/table/'
import DetailList from '@/components/tools/DetailList'
import ABadge from "ant-design-vue/es/badge/Badge"
import TraceabilityInformationList from '../traceability/TraceabilityInformationList'
import Qr from '../qr/Qr'
import {httpAction, getAction} from '@/api/manage'
import JMultiSelectTag from '@/components/dict/JMultiSelectTag'
import {validateDuplicateValue} from '@/utils/util'

const DetailListItem = DetailList.Item


export default {
  name: 'ApplianceInformationList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    ApplianceInformationModal,
    PageLayout,
    ABadge,
    DetailListItem,
    DetailList,
    STable,
    TraceabilityInformationList,
    Qr,
    JMultiSelectTag,
    validateDuplicateValue
  },
  props: {
    //表单禁用
    disabled: {
      type: Boolean,
      default: false,
      required: false
    }
  },
  data() {
    return {
      description: '器具信息管理页面',
      visible: false,
      visible2: false,
      drawerWidth: 850,
      confirmLoading: false,
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
        title: [
          {required: true, message: '请输入委托计划标题!'},
        ],
        laboratoryId: [
          {required: true, message: '请输入实验室!'},
        ],
      },
      record: {},
      delegation: {},
      tenantsOptions: [],
      // 表头
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
          title: '检测周期',
          align: "center",
          dataIndex: 'detectionCycle'
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
      url: {
        list: "/appliance/applianceInformation/list",
        delete: "/appliance/applianceInformation/delete",
        deleteBatch: "/appliance/applianceInformation/deleteBatch",
        exportXlsUrl: "/appliance/applianceInformation/exportXls",
        importExcelUrl: "appliance/applianceInformation/importExcel",
        queryTenantList: '/sys/tenant/byType?type=2',
        delegationAdd: "/delegation/delegation/add",
      },
      dictOptions: {},
      superFieldList: [],

    }
  },
  created() {
    this.$set(this.dictOptions, 'ifmetering', [{text: '是', value: 'Y'}, {text: '否', value: 'N'}])
    this.getSuperFieldList();
    this.initTenantList();
  },
  computed: {
    importExcelUrl: function () {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
    },
    title() {
      return this.$route.meta.title
    },
    formDisabled() {
      return this.disabled
    }
  },
  methods: {
    initDictConfig() {
    },
    showDrawer(data) {
      this.visible = true;
      this.record = data;
      this.resetScreenSize();
    },
    //初始化租户字典
    initTenantList() {
      getAction(this.url.queryTenantList).then(res => {
        if (res.success) {
          this.tenantsOptions = res.result.map((item, index, arr) => {
            let c = {label: item.name, value: item.id + ""}
            return c;
          })
          console.log('this.tenantsOptions: ', this.tenantsOptions)
        }
      })
    },
    // 根据屏幕变化,设置抽屉尺寸
    resetScreenSize() {
      let screenWidth = document.body.clientWidth;
      if (screenWidth < 500) {
        this.drawerWidth = screenWidth;
      } else {
        this.drawerWidth = 850;
      }
    },
    batchDelegate: function () {
      var ids = "";
      for (var a = 0; a < this.selectedRowKeys.length; a++) {
        ids += this.selectedRowKeys[a] + ",";
      }
      // alert(ids);
      this.delegation.applianceinformationid = ids;
      this.visible2 = true;
    },
    handleOk(e) {
      const that = this;
      // 触发表单验证
      this.$refs.delegationForm.validate(valid => {
        if (valid) {
          that.confirmLoading = true;
          let httpurl = this.url.delegationAdd;
          let method = 'post';

          httpAction(httpurl, this.delegation, method).then((res) => {
            if (res.success) {
              that.$message.success(res.message);
              that.$emit('ok');
              this.visible2 = false;
              this.delegation = {};
            } else {
              that.$message.warning(res.message);
            }
          }).finally(() => {
            that.confirmLoading = false;
          })
        }

      })
    },
    handleCancel(e) {
      console.log('Clicked cancel button');
      this.visible2 = false;
    },
    showValue: function (key) {
      const value = this.record[key];
      return !!value ? value : "--";
    },
    onClose() {
      this.visible = false;
    },
    getSuperFieldList() {
      let fieldList = [];
      fieldList.push({type: 'string', value: 'name', text: '器具名称', dictCode: ''})
      fieldList.push({type: 'string', value: 'model', text: '型号', dictCode: ''})
      fieldList.push({type: 'string', value: 'manufacturer', text: '生产厂家', dictCode: ''})
      fieldList.push({type: 'string', value: 'serialNumber', text: '出厂编号', dictCode: ''})
      fieldList.push({type: 'string', value: 'equipmentNumber', text: '设备编号', dictCode: ''})
      fieldList.push({type: 'string', value: 'measurementRange', text: '测量范围', dictCode: ''})
      fieldList.push({type: 'string', value: 'accuracy', text: '准确度	', dictCode: ''})
      fieldList.push({type: 'string', value: 'accessoriesSituation', text: '配件情况', dictCode: ''})
      fieldList.push({type: 'string', value: 'department', text: '所属部门', dictCode: ''})
      fieldList.push({type: 'string', value: 'responsible', text: '责任人', dictCode: ''})
      fieldList.push({type: 'string', value: 'storageLocation', text: '存放地点', dictCode: ''})
      fieldList.push({type: 'string', value: 'fileNumber', text: '档案编号', dictCode: ''})
      fieldList.push({type: 'string', value: 'deviceStatus', text: '设备状态', dictCode: ''})
      fieldList.push({type: 'string', value: 'usedNumber', text: '曾用编号', dictCode: ''})
      fieldList.push({type: 'switch', value: 'ifmetering', text: '是否为计量器具'})
      fieldList.push({type: 'string', value: 'remark', text: '备注	', dictCode: ''})
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>