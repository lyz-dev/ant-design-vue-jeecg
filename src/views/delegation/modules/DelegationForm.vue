<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="委托计划标题" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="title">
              <a-input v-model="model.title" placeholder="请输入委托计划标题"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="器具id" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="applianceinformationid">
              <a-input v-model="model.applianceinformationid" placeholder="请输入器具id"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="实验室" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="laboratoryId">
              <j-multi-select-tag :disabled="disableSubmit"
                                  v-model="model.laboratoryId"
                                  :options="tenantsOptions"
                                  placeholder="请选择实验室">
                {{ tenantsOptions }}
              </j-multi-select-tag>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
              <a-input v-model="model.remark" placeholder="请输入备注"  ></a-input>
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import JMultiSelectTag from '@/components/dict/JMultiSelectTag'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: 'DelegationForm',
    components: {
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
    data () {
      return {
        disableSubmit:false,
        model:{
         },
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
           title: [
              { required: true, message: '请输入委托计划标题!'},
           ],
           applianceinformationid: [
              { required: true, message: '请输入器具id!'},
           ],
           laboratoryId: [
              { required: true, message: '请输入实验室id!'},
           ],
        },
        tenantsOptions: [],
        url: {
          add: "/delegation/delegation/add",
          edit: "/delegation/delegation/edit",
          queryById: "/delegation/delegation/queryById",
          queryTenantList: '/sys/tenant/queryList'
        }
      }
    },
    computed: {
      formDisabled(){
        return this.disabled
      },
    },
    created () {
       //备份model原始值
      this.modelDefault = JSON.parse(JSON.stringify(this.model));
      this.initTenantList();
    },
    methods: {
      add () {
        this.edit(this.modelDefault);
      },
      edit (record) {
        this.model = Object.assign({}, record);
        this.visible = true;
      },
      //初始化租户字典
      initTenantList(){
        getAction(this.url.queryTenantList).then(res=>{
          if(res.success){
            this.tenantsOptions = res.result.map((item,index,arr)=>{
              let c = {label:item.name, value: item.id+""}
              return c;
            })
            console.log('this.tenantsOptions: ',this.tenantsOptions)
          }
        })
      },
      submitForm () {
        const that = this;
        // 触发表单验证
        this.$refs.form.validate(valid => {
          if (valid) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            httpAction(httpurl,this.model,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }

        })
      },
    }
  }
</script>