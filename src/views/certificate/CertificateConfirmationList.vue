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
      <a-button type="primary" icon="download" @click="handleExportXls('证书确认')">导出</a-button>
<!--      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">-->
<!--        <a-button type="primary" icon="import">导入</a-button>-->
<!--      </a-upload>-->
<!--      &lt;!&ndash; 高级查询区域 &ndash;&gt;-->
<!--      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>-->
<!--      <a-dropdown v-if="selectedRowKeys.length > 0">-->
<!--        <a-menu slot="overlay">-->
<!--          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>-->
<!--        </a-menu>-->
<!--        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>-->
<!--      </a-dropdown>-->
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
          <a @click="handleEdit(record)">确认证书</a>

<!--          <a-divider type="vertical" />-->
<!--          <a-dropdown>-->
<!--            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>-->
<!--            <a-menu slot="overlay">-->
<!--              <a-menu-item>-->
<!--                <a @click="handleDetail(record)">详情</a>-->
<!--              </a-menu-item>-->
<!--              <a-menu-item>-->
<!--                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">-->
<!--                  <a>删除</a>-->
<!--                </a-popconfirm>-->
<!--              </a-menu-item>-->
<!--            </a-menu>-->
<!--          </a-dropdown>-->
        </span>

      </a-table>
    </div>

    <certificate-confirmation-modal ref="modalForm" @ok="modalFormOk"></certificate-confirmation-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import CertificateConfirmationModal from './modules/CertificateConfirmationModal'
  import { initDictOptions } from '../../components/dict/JDictSelectUtil'

  export default {
    name: 'CertificateConfirmationList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      CertificateConfirmationModal
    },
    data () {
      return {
        description: '证书确认管理页面',
        verifyResults:[],
        // 表头
        columns: [
          {
            title: '器具名称',
            align: "center",
            dataIndex: 'appliance.name',
          },
          {
            title: '器具型号',
            align: "center",
            dataIndex: 'appliance.model',
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
            dataIndex: 'detectionCycle',
            type:Number,
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
          list: "/traceability/traceabilityInformation/list2",
          delete: "/certificate/certificateConfirmation/delete",
          deleteBatch: "/certificate/certificateConfirmation/deleteBatch",
          exportXlsUrl: "/certificate/certificateConfirmation/exportXls",
        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
    this.getSuperFieldList();
    },
    computed: {
    },
    methods: {
      initDictConfig(){
        //初始化确认状态
        initDictOptions('verifyResults').then((res) => {
          if (res.success) {
            this.verifyResults = res.result;
          }
        });
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