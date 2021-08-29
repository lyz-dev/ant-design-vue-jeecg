<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    switchFullscreen
    @ok="handleOk"
    :okButtonProps="{ class:{'jee-hidden': disableSubmit} }"
    @cancel="handleCancel"
    cancelText="关闭">
    <traceability-information-form v-bind:applianceInformationId="this.applianceInformationId"
                                   v-bind:cycle="this.cycle+0"
                                   ref="realForm" @ok="submitCallback"
                                   :disabled="disableSubmit"></traceability-information-form>
  </j-modal>
</template>

<script>

import TraceabilityInformationForm from './TraceabilityInformationForm'

export default {
  name: 'TraceabilityInformationModal',
  props: {
    applianceInformationId: {
      type: String,
      required: true
    },
    //周期
    cycle: {
      type: Number,
      default: 1,
      required: false
    }
  },
  components: {
    TraceabilityInformationForm
  },
  data() {
    return {
      title: '',
      width: 800,
      visible: false,
      disableSubmit: false
    }
  },
  methods: {
    add() {
      this.visible = true
      this.$nextTick(() => {
        this.$refs.realForm.add();
      })
    },
    edit(record) {
      this.visible = true
      this.$nextTick(() => {
        this.$refs.realForm.edit(record);
      })
    },
    close() {
      this.$emit('close');
      this.visible = false;
    },
    handleOk() {
      this.$refs.realForm.submitForm();
    },
    submitCallback() {
      this.$emit('ok');
      this.visible = false;
    },
    handleCancel() {
      this.close()
    }
  }
}
</script>