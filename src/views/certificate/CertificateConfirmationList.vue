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
      <a-button type="primary" icon="download" @click="handleExportXls('证书确认')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
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
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
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

    <certificate-confirmation-modal ref="modalForm" @ok="modalFormOk"></certificate-confirmation-modal>
  </a-card>
</template>

<script>

  import '@/assets/less/TableExpand.less'
  import { mixinDevice } from '@/utils/mixin'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import CertificateConfirmationModal from './modules/CertificateConfirmationModal'

  export default {
    name: 'CertificateConfirmationList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      CertificateConfirmationModal
    },
    data () {
      return {
        description: '证书确认管理页面',
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
            title:'检测依据',
            align:"center",
            dataIndex: 'testBasis'
          },
          {
            title:'应用标准名称及编号',
            align:"center",
            dataIndex: 'standardNameAndNumber'
          },
          {
            title:'技术要求',
            align:"center",
            dataIndex: 'skillsRequirement'
          },
          {
            title:'溯源结果',
            align:"center",
            dataIndex: 'traceabilityResults'
          },
          {
            title:'溯源结果备注',
            align:"center",
            dataIndex: 'traceabilityResultsRemark',
            scopedSlots: {customRender: 'imgSlot'}
          },
          {
            title:'确认结果',
            align:"center",
            dataIndex: 'verifyResults_dictText'
          },
          {
            title:'确认人',
            align:"center",
            dataIndex: 'confirmor'
          },
          {
            title:'确认日期',
            align:"center",
            dataIndex: 'confirmationDate',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'审核人',
            align:"center",
            dataIndex: 'reviewer'
          },
          {
            title:'审核日期',
            align:"center",
            dataIndex: 'reviewDate',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
          },
          {
            title:'批准人',
            align:"center",
            dataIndex: 'approver'
          },
          {
            title:'批准日期',
            align:"center",
            dataIndex: 'approverDate',
            customRender:function (text) {
              return !text?"":(text.length>10?text.substr(0,10):text)
            }
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
          list: "/certificate/certificateConfirmation/list",
          delete: "/certificate/certificateConfirmation/delete",
          deleteBatch: "/certificate/certificateConfirmation/deleteBatch",
          exportXlsUrl: "/certificate/certificateConfirmation/exportXls",
          importExcelUrl: "certificate/certificateConfirmation/importExcel",
          
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
    },
    methods: {
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'testBasis',text:'检测依据',dictCode:''})
        fieldList.push({type:'string',value:'standardNameAndNumber',text:'应用标准名称及编号',dictCode:''})
        fieldList.push({type:'string',value:'skillsRequirement',text:'技术要求',dictCode:''})
        fieldList.push({type:'string',value:'traceabilityResults',text:'溯源结果',dictCode:''})
        fieldList.push({type:'string',value:'traceabilityResultsRemark',text:'溯源结果备注',dictCode:''})
        fieldList.push({type:'string',value:'verifyResults',text:'确认结果',dictCode:''})
        fieldList.push({type:'string',value:'confirmor',text:'确认人',dictCode:''})
        fieldList.push({type:'date',value:'confirmationDate',text:'确认日期'})
        fieldList.push({type:'string',value:'reviewer',text:'审核人',dictCode:''})
        fieldList.push({type:'date',value:'reviewDate',text:'审核日期'})
        fieldList.push({type:'string',value:'approver',text:'批准人',dictCode:''})
        fieldList.push({type:'date',value:'approverDate',text:'批准日期'})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>