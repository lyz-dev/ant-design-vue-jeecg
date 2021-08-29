<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="检测依据" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="testBasis">
              <a-input v-model="model.testBasis" placeholder="请输入检测依据"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="应用标准名称及编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="standardNameAndNumber">
              <a-input v-model="model.standardNameAndNumber" placeholder="请输入应用标准名称及编号"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="技术要求" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="skillsRequirement">
              <a-input v-model="model.skillsRequirement" placeholder="请输入技术要求"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="溯源结果" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="traceabilityResults">
              <a-input v-model="model.traceabilityResults" placeholder="请输入溯源结果"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="溯源结果备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="traceabilityResultsRemark">
              <j-image-upload isMultiple  v-model="model.traceabilityResultsRemark" ></j-image-upload>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="确认结果" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="verifyResults">
              <j-dict-select-tag type="list" v-model="model.verifyResults" dictCode="" placeholder="请选择确认结果" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="确认人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="confirmor">
              <a-input v-model="model.confirmor" placeholder="请输入确认人"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="确认日期" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="confirmationDate">
              <j-date placeholder="请选择确认日期" v-model="model.confirmationDate"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="审核人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="reviewer">
              <a-input v-model="model.reviewer" placeholder="请输入审核人"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="审核日期" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="reviewDate">
              <j-date placeholder="请选择审核日期" v-model="model.reviewDate"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="批准人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="approver">
              <a-input v-model="model.approver" placeholder="请输入批准人"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="批准日期" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="approverDate">
              <j-date placeholder="请选择批准日期" v-model="model.approverDate"  style="width: 100%" />
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: 'CertificateConfirmationForm',
    components: {
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
        },
        url: {
          add: "/certificate/certificateConfirmation/add",
          edit: "/certificate/certificateConfirmation/edit",
          queryById: "/certificate/certificateConfirmation/queryById"
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
    },
    methods: {
      add () {
        this.edit(this.modelDefault);
      },
      edit (record) {
        this.model = Object.assign({}, record);
        this.visible = true;
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