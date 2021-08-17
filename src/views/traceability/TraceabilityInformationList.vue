<template>
  <a-card :bordered="false">
    <div class="table-operator">
      <a-button v-if="visibility" @click="handleAdd" type="primary" icon="plus">新增一周期</a-button>
    </div>

    <!-- table区域-begin -->
    <div>
      <div v-if="visibility" class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
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
          <a @click="handleEdit(record)" v-if="visibility">编辑</a>

          <a-divider type="vertical"/>
          <a-dropdown v-if="visibility">
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)">详情</a>
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

    <traceability-information-modal v-bind:applianceInformationId="this.applianceInformationId"
                                    v-bind:cycle="this.cycle"
                                    ref="modalForm" @ok="modalFormOk"></traceability-information-modal>
  </a-card>
</template>

<script>

import '@/assets/less/TableExpand.less'
import {mixinDevice} from '@/utils/mixin'
import {JeecgListMixin} from '@/mixins/JeecgListMixin'
import TraceabilityInformationModal from './modules/TraceabilityInformationModal'

export default {
  name: 'TraceabilityInformationList',
  mixins: [JeecgListMixin, mixinDevice],
  props: ['applianceInformationId','visibility','cycle'],
  components: {
    TraceabilityInformationModal
  },
  data() {
    return {
      description: '溯源信息管理页面',
      // 表头
      columns: [
        {
          title: '是否为主周期',
          align: "center",
          dataIndex: 'flag',
          customRender: function (text) {
            return !text ? "否" : (text===true ?'是':'否')
          }
        },
        {
          title: '检测机构',
          align: "center",
          dataIndex: 'testingFacility'
        },
        {
          title: '检测类型',
          align: "center",
          dataIndex: 'testingType'
        },
        {
          title: '检测地点',
          align: "center",
          dataIndex: 'detectionLocation'
        },
        {
          title: '检测日期',
          align: "center",
          dataIndex: 'detectionDate',
          customRender: function (text) {
            return !text ? "" : (text.length > 10 ? text.substr(0, 10) : text)
          }
        },
        {
          title: '证书编号',
          align: "center",
          dataIndex: 'certificateNumber'
        },
        {
          title: '检测周期',
          align: "center",
          dataIndex: 'detectionCycle'
        },
        {
          title: '到期日期',
          align: "center",
          dataIndex: 'dateDue',
          customRender: function (text) {
            return !text ? "" : (text.length > 10 ? text.substr(0, 10) : text)
          }
        },
        {
          title: '检测项目',
          align: "center",
          dataIndex: 'testItem'
        },
        {
          title: '检测费',
          align: "center",
          dataIndex: 'testFee'
        },
        {
          title: '电子证书',
          align: "center",
          dataIndex: 'certificateBackup',
          scopedSlots: {customRender: 'fileSlot'}
        },
        {
          title: '备注',
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
        list:"/traceability/traceabilityInformation/byApplianceInformationId?applianceInformationId="+this.applianceInformationId,
        delete: "/traceability/traceabilityInformation/delete",
        deleteBatch: "/traceability/traceabilityInformation/deleteBatch",
        exportXlsUrl: "/traceability/traceabilityInformation/exportXls",
        importExcelUrl: "traceability/traceabilityInformation/importExcel",
      },
      dictOptions: {},
      superFieldList: [],
    }
  },
  created() {
    this.getSuperFieldList();
  },
  computed: {
    importExcelUrl: function () {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
    },
  },
  methods: {
    initDictConfig() {
    },
    getSuperFieldList() {
      let fieldList = [];
      fieldList.push({type: 'string', value: 'testingFacility', text: '检测机构', dictCode: ''})
      fieldList.push({type: 'string', value: 'testingType', text: '检测类型', dictCode: ''})
      fieldList.push({type: 'string', value: 'detectionLocation', text: '检测地点', dictCode: ''})
      fieldList.push({type: 'date', value: 'detectionDate', text: '检测日期'})
      fieldList.push({type: 'string', value: 'certificateNumber', text: '证书编号', dictCode: ''})
      fieldList.push({type: 'date', value: 'dateDue', text: '到期日期'})
      fieldList.push({type: 'string', value: 'testItem', text: '检测项目', dictCode: ''})
      fieldList.push({type: 'string', value: 'testFee', text: '检测费', dictCode: ''})
      fieldList.push({type: 'string', value: 'certificateBackup', text: '电子证书', dictCode: ''})
      fieldList.push({type: 'string', value: 'remark', text: '备注', dictCode: ''})
      fieldList.push({type: 'string', value: 'applianceid', text: '器具id', dictCode: ''})
      this.superFieldList = fieldList
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>