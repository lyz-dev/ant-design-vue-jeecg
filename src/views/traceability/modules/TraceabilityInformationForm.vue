<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="检测机构" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="testingFacility">
              <a-input v-model="model.testingFacility" placeholder="请输入检测机构"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="检测类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="testingType">
              <a-input v-model="model.testingType" placeholder="请输入检测类型"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="检测地点" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="detectionLocation">
              <a-input v-model="model.detectionLocation" placeholder="请输入检测地点"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="检测日期" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="detectionDate">
              <j-date placeholder="请选择检测日期" v-model="model.detectionDate" style="width: 100%"/>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="证书编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="certificateNumber">
              <a-input v-model="model.certificateNumber" placeholder="请输入证书编号"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="检测周期" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="detectionCycle">
              <a-input v-model="model.detectionCycle" placeholder="请输入检测周期"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="到期日期" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="dateDue">
              <j-date placeholder="请选择到期日期" v-model="model.dateDue" style="width: 100%"/>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="检测项目" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="testItem">
              <a-input v-model="model.testItem" placeholder="请输入检测项目"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="检测费" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="testFee">
              <a-input v-model="model.testFee" placeholder="请输入检测费"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="证书备份" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="certificateBackup">
              <j-upload v-model="model.certificateBackup"></j-upload>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="备注" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="remark">
              <a-input v-model="model.remark" placeholder="请输入备注"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item hidden label="器具id" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="applianceid">
              <a-input v-model="model.applianceid"   placeholder="请输入器具id"></a-input>
            </a-form-model-item>
          </a-col>
        </a-row>
      </a-form-model>
    </j-form-container>
  </a-spin>
</template>

<script>

import {httpAction, getAction} from '@/api/manage'
import {validateDuplicateValue} from '@/utils/util'

export default {
  name: 'TraceabilityInformationForm',
  components: {},
  props: {
    //表单禁用
    disabled: {
      type: Boolean,
      default: false,
      required: false
    },
    applianceInformationId: {
      type: String,
      required: true
    },
  },
  data() {
    return {
      model: {},
      labelCol: {
        xs: {span: 24},
        sm: {span: 5},
      },
      wrapperCol: {
        xs: {span: 24},
        sm: {span: 16},
      },
      confirmLoading: false,
      validatorRules: {
        testingFacility: [
          {required: true, message: '请输入检测机构!'},
        ],
        testingType: [
          {required: true, message: '请输入检测类型!'},
        ],
        detectionLocation: [
          {required: true, message: '请输入检测地点!'},
        ],
        detectionDate: [
          {required: true, message: '请输入检测日期!'},
        ],
        certificateNumber: [
          {required: true, message: '请输入证书编号!'},
        ],
        detectionCycle: [
          {required: true, message: '请输入检测周期!'},
        ],
        dateDue: [
          {required: true, message: '请输入到期日期!'},
        ],
        testItem: [
          {required: true, message: '请输入检测项目!'},
        ],
        testFee: [
          {required: true, message: '请输入检测费!'},
        ],
      },
      url: {
        add: "/traceability/traceabilityInformation/add",
        edit: "/traceability/traceabilityInformation/edit",
        queryById: "/traceability/traceabilityInformation/queryById"
      }
    }
  },
  computed: {
    formDisabled() {
      return this.disabled
    },
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model));
  },
  methods: {
    add() {
      this.edit(this.modelDefault);
    },
    edit(record) {
      record.applianceid = this.applianceInformationId
      this.model = Object.assign({}, record);
      this.visible = true;
    },
    submitForm() {
      const that = this;
      // 触发表单验证
      this.$refs.form.validate(valid => {
        if (valid) {
          that.confirmLoading = true;
          let httpurl = '';
          let method = '';
          if (!this.model.id) {
            httpurl += this.url.add;
            method = 'post';
          } else {
            httpurl += this.url.edit;
            method = 'put';
          }
          httpAction(httpurl, this.model, method).then((res) => {
            if (res.success) {
              that.$message.success(res.message);
              that.$emit('ok');
            } else {
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