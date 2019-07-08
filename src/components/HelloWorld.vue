<template>
  <div class="hello">
    <img :src="imgUrl" class="img">
    <button @click="showUpload()">点击上传</button>
    <VueCrop :visible="isShow" @on-close="close()" @on-success="corpSucc"></VueCrop>
  </div>
</template>

<script>
import VueCrop from '@/components/VueCrop'
export default {
  name: 'HelloWorld',
  data () {
    return {
      imgUrl: '',
      url: '',
      isShow: false
    }
  },
  components: {
    VueCrop
  },
  methods: {
    showUpload () {
      this.isShow = true
    },
    close () {
      this.isShow = false
    },
    corpSucc (v) {
      this.imgUrl = v
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello {
  margin: 50px auto;
}
.img {
  width: 300px;
  height: 300px;
}
</style>
