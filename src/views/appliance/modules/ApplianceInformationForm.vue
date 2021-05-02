<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="器具名称" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="name">
              <a-input v-model="model.name" placeholder="请输入器具名称"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="型号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="model">
              <a-input v-model="model.model" placeholder="请输入型号"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="生产厂家" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="manufacturer">
              <a-input v-model="model.manufacturer" placeholder="请输入生产厂家"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="出厂编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="serialNumber">
              <a-input v-model="model.serialNumber" placeholder="请输入出厂编号"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="设备编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="equipmentNumber">
              <a-input v-model="model.equipmentNumber" placeholder="请输入设备编号"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="测量范围" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="measurementRange">
              <a-input v-model="model.measurementRange" placeholder="请输入测量范围"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="准确度	" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="accuracy">
              <a-input v-model="model.accuracy" placeholder="请输入准确度	"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="配件情况" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="accessoriesSituation">
              <a-input v-model="model.accessoriesSituation" placeholder="请输入配件情况"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="所属部门" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="department">
              <a-input v-model="model.department" placeholder="请输入所属部门"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="责任人" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="responsible">
              <a-input v-model="model.responsible" placeholder="请输入责任人"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="存放地点" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="storageLocation">
              <a-input v-model="model.storageLocation" placeholder="请输入存放地点"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="档案编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="fileNumber">
              <a-input v-model="model.fileNumber" placeholder="请输入档案编号"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="设备状态" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="deviceStatus">
              <a-input v-model="model.deviceStatus" placeholder="请输入设备状态"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="曾用编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="usedNumber">
              <a-input v-model="model.usedNumber" placeholder="请输入曾用编号"  ></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="是否为计量器具" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="ifmetering">
              <j-switch v-model="model.ifmetering"  ></j-switch>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="备注	" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
              <a-textarea v-model="model.remark" rows="4" placeholder="请输入备注	" />
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
    name: 'ApplianceInformationForm',
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
          add: "/appliance/applianceInformation/add",
          edit: "/appliance/applianceInformation/edit",
          queryById: "/appliance/applianceInformation/queryById"
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