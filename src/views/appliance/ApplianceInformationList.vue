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
                  <a-drawer
                    :title="title"
                    :width="1440"
                    :visible="visible"
                    :body-style="{ paddingBottom: '80px' }"
                    @close="onClose">
                    <a-card :bordered="false">
                      <div class="title"><router-link target="_blank" :to="{path:'/Qr/'+record.id}">生成二维码</router-link></div>
                      <detail-list>
                        <detail-list-item :term="column.title" v-if="column.key != 'rowIndex'" v-for="column in columns" :key="column.dataIndex">{{showValue(column.dataIndex)}}</detail-list-item>
                      </detail-list>
                      <a-divider style="margin-bottom: 32px"/>

                      <div class="title">溯源信息</div>
                      <TraceabilityInformationList v-bind:applianceInformationId="record.id" v-bind:visibility="true" >
                      </TraceabilityInformationList>
                    </a-card>
                  </a-drawer>
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

    <appliance-information-modal ref="modalForm" @ok="modalFormOk"></appliance-information-modal>
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
    Qr
  },
  data() {
    return {
      description: '器具信息管理页面',
      visible: false,
      record: {},
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

      },
      dictOptions: {},
      superFieldList: [],
    }
  },
  created() {
    this.$set(this.dictOptions, 'ifmetering', [{text: '是', value: 'Y'}, {text: '否', value: 'N'}])
    this.getSuperFieldList();
  },
  computed: {
    importExcelUrl: function () {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
    },
    title() {
      return this.$route.meta.title
    }
  },
  methods: {
    initDictConfig() {
    },
    showDrawer(data) {
      this.visible = true;
      this.record = data;

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