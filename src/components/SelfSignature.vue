<template>
  <div>


    <!-- 签名组件 -->
    <VueSignaturePad
      width="80%"
      height="150px"
      ref="signaturePad"
      :customStyle="signatureStyle"
      :options="{ onBegin, onEnd }"
    />

    <!-- 清空 保存 删除 -->
    <div class="tab">
      <van-button class="tab-item" color="#1989fa" size="small" @click="clearImg()">清空</van-button>
      <div style="width:10px;"></div>
      <van-button class="tab-item" color="#1989fa" size="small" @click="undoImg()">撤销</van-button>
      <div style="width:10px;"></div>
      <van-button class="tab-item" color="#1989fa" size="small" @click="saveImg()">保存</van-button>
      <div style="width:10px;"></div>
      <van-button class="tab-item" color="#1989fa" size="small" @click="uploadImg()">上传</van-button>
    </div>
    

  </div>
</template>

<script>

import { selfRequest } from "@/utils/SelfRequest.js";

export default {
  name: 'SelfSignature',
  props: {
    msg: String
  },
  /** 
   * 数据
  */
  data() {
    return {
      message: 'SelfSignature',
      signatureStyle : {
        border: 'gray 1px solid',
         margin : '20px auto'
      },
    }
  },
  /**
   *  计算属性
   *  如果属性没有发生变化，直接拿缓存的值，不走过程（第一次走过程）
  */
  computed: {
    isTitle(){
      return "加入我们";
    }
  },
  /** 
   * 侦听属性
   * 如果属性发生变化就触发对应的方法
  */
  watch: {
    isTitle : function(newValue, oldValue){
        console.log(newValue+ "-" + oldValue )
    }
  },  
  /** 
   * 方法
  */
  methods: {
    /**
     * 开始涂画
     */
    onBegin() {
      console.log('=== Begin 开始涂画===');
    },
    /**
     * 结束涂画
     */
    onEnd() {
      console.log('=== End 结束涂画===');

      //得到signaturePad画板数据
      const { isEmpty, data } = this.$refs.signaturePad.saveSignature();

      //子组件通过this.$emit()的方式将值传递给父组件
      //注意：这里emit第一个参数是父组件中绑定的函数名
      this.$emit('getSignaturePadData', data)
    },
    /**
     *  清空图片
     */
    clearImg(){
      this.$refs.signaturePad.clearSignature()
    },
    /**
     *  撤销图片
     */
    undoImg(){
      this.$refs.signaturePad.undoSignature();
    },
    /**
     *  保存图片
     */
    saveImg(){
      //得到signaturePad画板数据
      const { isEmpty, data } = this.$refs.signaturePad.saveSignature();
      //判断是否涂画
      if(!isEmpty){
        //把画板 涂画的内容保存到 本地
        this.downloadFile(this.convertBase64UrlToBlob(data) , 'pic');
      }else{
        //提示 请在画板涂画
        this.$toast({
          message : "请在画板涂画",
          icon : 'fail'
        })
      }           
    },
    /**
     *  上传图片
     */
    uploadImg(){
      //得到signaturePad画板数据
      const { isEmpty, data } = this.$refs.signaturePad.saveSignature();
      //判断是否涂画
      if(!isEmpty){
        
        // 使用formdata上传图片
        var formData = new FormData();
        //设置文件名
        let filename = new Date().getTime() + ".png";
        //添加图片Blob对象
        formData.append("file", this.convertBase64UrlToBlob(data) , filename);

        selfRequest({method: 'post' , url : 'http://127.0.0.1:8080/Supervision/file/upload' , data : formData})

      }else{
        //提示 请在画板涂画
        this.$toast({
          message : "请在画板涂画",
          icon : 'fail'
        })
      }       
    },
    /**
     * 将以base64的图片url数据转换为Blob
     */
    convertBase64UrlToBlob(urlData){

       //去掉url的头，并转换为byte
      let bytes = window.atob(urlData.split(',')[1]);       
        
      //处理异常,将ascii码小于0的转换为大于0
      let ab = new ArrayBuffer(bytes.length);
      let ia = new Uint8Array(ab);
      for (let i = 0; i < bytes.length; i++) {
          ia[i] = bytes.charCodeAt(i);
      }

      return new Blob( [ab] , {type : 'image/png'});
    },
   
    /**
     * 下载文件
     * 参数 :
     *  Blob 对象
     *  fileName 文件名
     */
    downloadFile(blob , fileName){

      //Blob 对象的URL
      let url = window.URL.createObjectURL(blob)

      //创建a标签
      let link = document.createElement('a');
      //设置样式
      link.style.display = 'none';
      //设置url
      link.href = url;
      //设置属性
      link.setAttribute('download', fileName + '.png');
      //添加到body标签
      document.body.appendChild(link);
      //a标签点击
      link.click();
    },

  },
}
</script>

<style scoped>

   .tab{
        display: flex;
        width: 80%;
        margin: 0 auto;
    }
    .tab-item{
        flex: 1;
        text-align: center;
        width: 150px;
    }

</style>