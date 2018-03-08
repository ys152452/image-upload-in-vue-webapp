<template>
  <div id="imaige-upload">
    <img v-for="(item, index) in uploadList" src="item" :key="index">
    <button @click="imgUpload">upload</button>
  </div>
</template>
<script>
// 按需引入mintUI中的组件
import {  } from "mint-ui";
export default {
  computed: {},
  data() {
    return {
      // 图片的数据
      uploadList: [
        '111.png',
        '222.png',
      ]
    };
  },
  components: {},
  activated() {},
  mounted() {
    const _this = this;
    // 配置底部唤出的操作项文案以及回调，对应配置api参考mintUI文档
    this.uploadActions = [
      {
        name: "Camera",
        method: function() {
          // 获取手机摄像头
          let cmr = plus.camera.getCamera();
          // 拍照回调
          cmr.captureImage(
            function(capturedFile) {
              // 通过HTML5+PLUS中的io进行拍照文件的处理
              plus.io.resolveLocalFileSystemURL(
                capturedFile,
                function(entry) {
                  // 解析为可在页面显示的src并push到uploadList中
                  _this.uploadList.push(entry.toLocalURL());
                },
                function(e) {
                  console.log(e);
                }
              );
            },
            function(error) {
              console.log(error);
            },
            {}
          );
        }
      },
      {
        name: "Album",
        method: function() {
          // 拉起系统相册并选择图片
          plus.gallery.pick(
            function(capturedFile) {
              _this.uploadList.push(capturedFile);
            },
            function(error) {
              console.log(error);
            },
            {
              filter: "image"
            }
          );
        }
      }
    ];
  },
  methods: {
    imgUpload() {
      const _this = this;
      // 未选择图片时的提示
      if (this.uploadList.length <= 0) {
        Toast("No image to upload");
        return;
      }
      // mintUI中的过渡效果
      Indicator.open("Loading...");
      // 遍历并转化为base64，单个上传至服务器
      this.uploadList.forEach((item, index) => {
        let num = index + 1;
        let imgObj = new Image()
        imgObj.src = item
        _this.$http.post("URL", {
            basecode: _this.getBase64Image(imgObj)
          })
          .then(response => {
            if (response.data.errorCode === "0") {
              // 全部完成后的操作
              if (num === _this.uploadList.length) {
                Indicator.close();
                _this.uploadList = [];
                _this.$router.push("prePage");
              }
            } else {
              // 抛出错误提示
              Toast("Image " + num + " upload failed");
            }
          })
          .catch(e => {
            console.log(e);
          });
      });
    },
    getBase64Image(img) {
      // 利用canvas将图片转为base64，可配置压缩
      let canvas = document.createElement("canvas");
      // 获取图片原始的尺寸
      const width = img.naturalWidth;
      const height = img.naturalHeight;
      canvas.width = width;
      canvas.height = height;
      let ctx = canvas.getContext("2d");
      ctx.drawImage(img, 0, 0, width, height);
      let dataURL = canvas.toDataURL("image/png", 0.8);
      return dataURL.replace("data:image/png;base64,", "");
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
