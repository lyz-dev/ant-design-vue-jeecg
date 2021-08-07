<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form-model ref="form" :model="model" :rules="validatorRules" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-model-item label="检测机构" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="testingFacility">
<!--              <a-input v-model="model.testingFacility" placeholder="请输入检测机构"></a-input>-->
              <a-auto-complete
                v-model="model.testingFacility"
                :data-source="testingFacility_dataSource"
                placeholder="请输入检测机构"
                :filter-option="filterOption"
              />
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="检测类型" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="testingType">
<!--              <a-input v-model="model.testingType" placeholder="请输入检测类型"></a-input>-->
              <a-radio-group  v-model="model.testingType" default-value="检定" button-style="solid">
                <a-radio-button value="检定">
                  检定
                </a-radio-button>
                <a-radio-button value="校准">
                  校准
                </a-radio-button>
                <a-radio-button value="检测报告">
                  检测报告
                </a-radio-button>
                <a-radio-button value="其他">
                  其他
                </a-radio-button>
              </a-radio-group>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="检测地点" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="detectionLocation">
<!--              <a-input v-model="model.detectionLocation" placeholder="请输入检测地点"></a-input>-->
              <a-radio-group  v-model="model.detectionLocation" default-value="本地" button-style="solid">
                <a-radio-button value="本地">
                  本地
                </a-radio-button>
                <a-radio-button value="送检">
                  送检
                </a-radio-button>
                <a-radio-button value="其他">
                  其他
                </a-radio-button>
              </a-radio-group>
              <a-input v-if="model.detectionLocation=='其他'" v-model="model.locationOther" placeholder="请输入检测地点"></a-input>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="检测日期" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="detectionDate">
              <j-date placeholder="请选择检测日期" v-model="model.detectionDate" style="width: 100%" @change="autoDateDue"/>
            </a-form-model-item>
          </a-col>
          <a-col :span="24">
            <a-form-model-item label="证书编号" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="certificateNumber">
              <a-input v-model="model.certificateNumber" placeholder="请输入证书编号"></a-input>
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
            <a-form-model-item label="电子证书" :labelCol="labelCol" :wrapperCol="wrapperCol" prop="certificateBackup">
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
import moment from 'moment'

export default {
  name: 'TraceabilityInformationForm',
  components: {moment},
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
    //周期
    cycle: {
      type: Number,
      default: 1,
      required:false
    }
  },
  data() {
    return {
      myTempObj:[],
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
    testingFacility_dataSource: {
      get: function () {
        return this.myTempObj; // 在这里把临时对象的值通过计算属性赋值给页面中用到的对象
      }
    }
  },
  created() {
    //备份model原始值
    this.modelDefault = JSON.parse(JSON.stringify(this.model));
    this.getAllTestingFacility();
  },
  methods: {
    autoDateDue(){
      // alert(this.model.detectionDate);
      this.model.dateDue = moment(this.model.detectionDate).add(this.cycle, 'month').subtract(1, 'days').format('YYYY-MM-DD');
    },
    filterOption(input, option) {
      return (
        option.componentOptions.children[0].text.toUpperCase().indexOf(input.toUpperCase()) >= 0
      );
    },
    getAllTestingFacility() { // 发起get请求
      this.$http({
        method: "get",
        url: "/traceability/traceabilityInformation/getAllTestingFacility",
      }).then(res => {
        this.myTempObj = res.result; // 在这里用临时变量接受服务端返回值
      })
    },
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