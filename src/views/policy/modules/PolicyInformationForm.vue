<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="政策链接" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="policyLink">
              <j-editor v-model="model.policyLink" />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="是否置顶" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="istop">
              <a-radio-group v-model="model.istop">
                <a-radio :value="0">
                  不置顶
                </a-radio>
                <a-radio :value="1">
                  置顶
                </a-radio>
              </a-radio-group>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="是否上架" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="active">
                <a-radio-group v-model="model.active">
                  <a-radio :value="0">
                    下架
                  </a-radio>
                  <a-radio :value="1">
                    上架
                  </a-radio>
                </a-radio-group>
              </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>

  import { httpAction, getAction } from '@api/manage'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: 'PolicyInformationForm',
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
           policyLink: [
              { required: true, message: '请输入政策链接!'},
           ],
           istop: [
              { required: true, message: '请输入是否置顶!'},
           ],
           active: [
              { required: true, message: '请输入是否上架!'},
           ],
        },
        url: {
          add: "/policy/policyInformation/add",
          edit: "/policy/policyInformation/edit",
          queryById: "/policy/policyInformation/queryById"
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