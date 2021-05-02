<template>
  <div>
    <IndexImage></IndexImage>
    <IndexDetail></IndexDetail>
  </div>
</template>

<script>
  import IndexImage from './IndexImage'
  import IndexDetail from "./IndexDetail"

  export default {
    name: "Analysis",
    components: {
      IndexImage,
      IndexDetail
    },
    data() {
      return {}
    },
    created() {

    },
    methods: {

    }
  }
</script>