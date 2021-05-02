<template>
  <a-carousel autoplay>
    <!--  <a slot="customPaging" slot-scope="props">-->
    <!--  <img :src="getImgUrl(props.i)"/>-->
    <!--  </a>-->
    <div v-for="item in imageUrls">
      <img :src='item' />
    </div>
  </a-carousel>
</template>
<script>

// const baseUrl =
//   'https://raw.githubusercontent.com/vueComponent/ant-design-vue/master/components/vc-slick/assets/img/react-slick/'
// console.log(process.env.VUE_APP_API_BASE_URL);

export default {
  name: 'IndexImage',
  data() {
    return {
      imageUrls:[],
    }
  },
  mounted() {
    if (!this.loaded) {
      this.list();
    }
  },
  methods: {
    list:function(){
      this.$http.get("/enterprise/enterpriseImage/list?active=1").then((resp)=> {
        if (resp.code === 200) {
          for (let i = 0; i < resp.result.records.length; i++) {
            this.imageUrls.push(process.env.VUE_APP_API_BASE_URL+'/sys/common/static/'+resp.result.records[i].filePath)
          }
        }
      })
    }
  }
}
</script>
<style scoped>
/* For demo */
.ant-carousel >>> .slick-dots {
  height: 280px;
  overflow: hidden;

}

.ant-carousel >>> .slick-slide img {
  /*border:5px solid #fff;*/
  display: block;
  margin: auto;
  max-width: 100%;
  max-height: 100%;
  height: 280px;
  width: 100%;
}

.ant-carousel >>> .slick-thumb {
  bottom: -45px;
}

.ant-carousel >>> .slick-thumb li {
  width: 60px;
  height: 45px;
}

.ant-carousel >>> .slick-thumb li img {
  width: 100%;
  height: 100%;
  filter: grayscale(100%);
}

.ant-carousel >>> .slick-thumb li.slick-active img {
  filter: grayscale(0%);
}
</style>