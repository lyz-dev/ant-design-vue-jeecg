<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :md="6" :sm="24">
            <a-form-item label="到期天数">
              <j-input type="number" v-model="queryParam.numOfDays" placeholder="请输入到期天数"></j-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="24">
            <a-form-item label="到期起止日期">
              <a-range-picker
                format="YYYY-MM-DD"
                @change="onChangeTimeProject"
                v-model="timeValue"/>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <!--      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>-->
      <a-button type="primary" icon="download" @click="handleExportXls('预超期器具信息')">导出</a-button>
      <!--      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl"-->
      <!--                @change="handleImportExcel">-->
      <!--        <a-button type="primary" icon="import">导入</a-button>-->
      <!--      </a-upload>-->
      <!-- 高级查询区域 -->
      <!--      <j-super-query :fieldList="superFieldList" ref="superQueryModal"-->
      <!--                     @handleSuperQuery="handleSuperQuery"></j-super-query>-->
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
            <a @click="showDrawer(record)">详情</a>
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


      <a-space>
        <a-button type="primary" @click="qjbs">
          导出器具标识
        </a-button>

        <a-button type="primary" @click="jlbs">
          导出计量标识
        </a-button>

        <a-button type="primary" @click="qrcodebs">
          <!--          <router-link target="_blank" :to="{path:'/Qr/'+record.id}">生成二维码</router-link>-->
          生成二维码
        </a-button>

      </a-space>

      <a-divider style="margin-bottom: 32px"/>
      <a-descriptions title="器具信息" bordered>
        <a-descriptions-item :label="column.title" v-if="column.key !== 'rowIndex' && column.title !=='操作'"
                             v-for="column in columns" :key="column.key">
          {{ showValue(column.dataIndex) }}
        </a-descriptions-item>
      </a-descriptions>
      <a-divider style="margin-bottom: 32px"/>

      <trace-detail v-bind:applianceInformationId="record.id" v-if="visible"></trace-detail>

    </a-drawer>


    <a-modal v-model="qjVisible" title="器具标识">
      <template slot="footer">
        <a-button key="back" @click="qjCancel">
          关闭
        </a-button>
      </template>
      <appliance-print v-bind:applianceInformation="record" v-if="qjVisible"></appliance-print>
    </a-modal>

    <a-modal v-model="jlVisible" title="计量标识">
      <template slot="footer">
        <a-button key="back" @click="jlCancel">
          关闭
        </a-button>
      </template>
      <trace-print v-bind:applianceInformation="record" v-if="jlVisible"></trace-print>
    </a-modal>

    <a-modal v-model="qrcodeVisible" title="二维码">
      <template slot="footer">
        <a-button key="back" @click="qrCancel">
          关闭
        </a-button>
      </template>
      <Qr v-bind:paramId="record.id" v-if="qrcodeVisible"></Qr>
    </a-modal>

    <appliance-information-modal ref="modalForm" @ok="modalFormOk"></appliance-information-modal>

    <a-modal
      title="发起委托议价"
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
import JeecgDemoModal from "../jeecg/modules/JeecgDemoModal";
import AppliancePrint from "./AppliancePrint";
import TracePrint from "../traceability/TracePrint";
import JDate from '@/components/jeecg/JDate';
import moment from 'moment';
import TraceDetail from "../traceability/TraceDetail";



const DetailListItem = DetailList.Item


export default {
  name: 'ApplianceInformationList2',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    JeecgDemoModal,
    JDate,
    ApplianceInformationModal,
    PageLayout,
    ABadge,
    DetailListItem,
    DetailList,
    STable,
    TraceabilityInformationList,
    Qr,
    JMultiSelectTag,
    validateDuplicateValue,
    AppliancePrint,
    TracePrint,
    TraceDetail
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
      //dateFormat: 'YYYY/MM/DD',
      visible: false,
      visible2: false,
      qjVisible: false,
      jlVisible: false,
      qrcodeVisible: false,
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
      moment,
      formItemLayout: {
        labelCol: { span: 6 },
        wrapperCol: { span: 14 }
      },
      //  因为一般情况下期望得到的都是对象，所以我们要自己定义一个时间名字(随便定义)
      //  这里定义的都是默认当天的时间段，格式是年月日
      timeValue: [],
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
          dataIndex: 'name',
          fixed: 'left'
        },
        {
          title: '型号',
          align: "center",
          dataIndex: 'model',
          fixed: 'left'
        },
        {
          title: '生产厂家',
          align: "center",
          dataIndex: 'manufacturer',
          fixed: 'left'
        },
        {
          title: '设备编号',
          align: "center",
          dataIndex: 'equipmentNumber',
          fixed: 'left'
        },
        {
          title: '出厂编号',
          align: "center",
          dataIndex: 'serialNumber',
        },
        {
          title: '测量范围',
          align: "center",
          dataIndex: 'measurementRange',
        },
        {
          title: '准确度	',
          align: "center",
          dataIndex: 'accuracy',
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
          title: '检测周期（月）',
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
        list: "/appliance/applianceInformation/preStage",
        delete: "/appliance/applianceInformation/delete",
        deleteBatch: "/appliance/applianceInformation/deleteBatch",
        exportXlsUrl: "/appliance/applianceInformation/exportXls",
        importExcelUrl: "appliance/applianceInformation/preStage/importExcel",
        queryTenantList: '/sys/tenant/byType?type=2',
        delegationAdd: "/delegation/delegation/add",
      },
      dictOptions: {},
      superFieldList: [],
      queryParam: {
        numOfDays:'30'
      },
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
    onChangeTimeProject(date, dateString) {
      // 让我们传的值等于我们所选的值
      this.timeValue = date;
      // 需求小技巧：此处为判断我们所选择的时间是否超过三个月，超过了三个月需要友好提醒
      if (this.timeValue[0] && this.timeValue[1]) {
        if (Math.abs(moment(this.timeValue[0]).diff(moment(this.timeValue[1]), 'days')) > 90) {
          this.$message.info('时间跨度超过3个月，查询时间可能较长', 5);
        }
      }
      this.queryParam.begin = this.timeValue.length > 0 ? this.timeValue[0].format('YYYY-MM-DD') : '';
      this.queryParam.end = this.timeValue.length > 0 ? this.timeValue[1].format('YYYY-MM-DD') : '';
      console.log(this.queryParam.begin);
      console.log(this.queryParam.end);
    },
    qjbs() {
      this.qjVisible = true;
    },
    jlbs() {
      this.jlVisible = true;
    },
    qrcodebs() {
      this.qrcodeVisible = true;
    },
    qjCancel(e) {
      this.qjVisible = false;
    },
    jlCancel(e) {
      this.jlVisible = false;
    },
    qrCancel(e) {
      this.qrcodeVisible = false;
    },
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