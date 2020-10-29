# 记录一些在实际工作中碰到的问题以及解决方案

## 前端使用 `vue-pdf` 进行 pdf 预览和打印

+ 文档链接地址：https://www.npmjs.com/package/vue-pdf

+ 已经将修改过源码的vue-pdf包放在同级目录，已经解决了预览和打印乱码的问题（版本：4.2.0）

+ 字体问题导致的乱码解决方案的地址：https://github.com/FranckFreiburger/vue-pdf/issues/229

  ```js
  // IE 139.5 其他 145
  let userAgent = navigator.userAgent; //判断浏览器内核
  if (userAgent.indexOf('Trident') != -1) {
      // IE 内核
      this.$refs.myPdfComponent[0].print(139.5);
  }else{
      // 非 IE 内核
   	this.$refs.myPdfComponent[0].print(145);
  }
  // 以上代码解决不同浏览器打印出现的不完整的调整（比如：每页下面的页码没显示出来）
  ```

  ```vue
  <template>
  	<pdf
          ref="myPdfComponent"
          v-for="i in numPages"
          :key="i"
          :src="src"
          :page="i"
          style="display: inline-block;width: 100%;border: 1px solid #ccc;"
  ></pdf>
  </template>
  <script>
  import pdf from 'vue-pdf'
  export default {
  	components: {pdf},
      data(){
          return{
              src:'',
              numPages: undefined
          }
      },
      mounted(){
          this.$post('/rcfz/expertDeclarationFileService/eleDeclarForm').then(res => {
              this.fileUpload.getFileUrl(res.alikey).then(key => {
                  this.src = pdf.createLoadingTask( key )
              }).then(_=>{
                  this.src.promise.then(pdf => {
                      this.numPages = pdf.numPages;
              });
          }) 
      },
      methods:{
  		handlePrint() {
              if (!this.canClick) return;
              // IE 139.5 其他 145
              let userAgent = navigator.userAgent; //判断浏览器内核
              if (userAgent.indexOf('Trident') != -1) {
                  // IE 内核
                  this.$refs.myPdfComponent[0].print(139.5);
              }else{
                  // 非 IE 内核
                  this.$refs.myPdfComponent[0].print(145);
              }
              
          }
      }
  }
  </script>
  ```

  