<template>
  <v-container>
    <canvas
      ref="canvasContext"
      :width="styleObject.width"
      :height="styleObject.height"
      @click="pathPointXY()"
      @dblclick="endPathXY()"
    ></canvas>
    <v-card class="cardClass">
      <v-form>
        <v-row>
          <v-col cols="4">
            <v-btn color="warning" dark @click="resetImage()">重置图片</v-btn>
          </v-col>
          <v-col cols="4">
            <v-btn color="primary" dark @click="savePaths()">下载标识</v-btn>
          </v-col>
        </v-row>
        <v-alert
          :value="alert"
          color="green"
          dark
          border="top"
          icon="mdi-home"
          transition="scale-transition"
        >
        标注信息保存成功，请到根目录查看。
        </v-alert>
        <v-container>
          <v-row>
            <v-col v-for="(item, index) in paths" :key="index" cols="2">
              <v-text-field
                v-model="item.title"
                label="名称"
                outlined
              ></v-text-field>
            </v-col>
          </v-row>
          <v-textarea
            outlined
            name="input-7-4"
            label="标识结果"
            :value="textare"
           ></v-textarea>
        </v-container>
      </v-form>
    </v-card>
  </v-container>
</template>

<script>
const { ipcRenderer } = window.require("electron");

export default {
  name: "HelloWorld",
  props: ["msg"],
  data: () => ({
    alert: false,
    canvasObject: Object,
    styleObject: {
      width: document.documentElement.clientWidth,
      height: document.documentElement.clientHeight,
      backgroundImage: "test",
    },
    mousePath: [],
    count: 0,
    paths: [],
    scaleNum: 0,
    textare:''
  }),

  mounted() {
    // this.$data.styleObject.backgroundImage = this.msg.path
    // 将文件本地存储信息传给electron主进程,主进程将文件读取后传回渲染进程
    ipcRenderer.send("filepath", this.msg.path);
    // ipcRenderer.send('filepath', 'ping')
    // 读取electron主进程读取的文件内容，并传存
    ipcRenderer.on("showpic", (e, { data }) => {
      var that = this;
      var file = new File([data], "AnyName.jpg", { type: "image/jpg" });
      var reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onload = function (e) {
        var newUrl = this.result;
        // console.log(newUrl)
        that.$data.styleObject.backgroundImage = newUrl;
        that.initCanvas();
      };
    });
  },

  methods: {
    // 添加canvas
    initCanvas() {
      var c = this.$refs.canvasContext;
      var ctx = c.getContext("2d");
      this.$data.canvasObject = ctx;
      var img = new Image();
      img.src = this.$data.styleObject.backgroundImage;
      const that = this;
      img.onload = function () {
        if (img.complete) {
          //  根据图像重新设定了canvas的长宽
          that.$data.scaleNum = document.documentElement.clientWidth / img.width;
          ctx.scale(that.$data.scaleNum, that.$data.scaleNum);
          console.log(that.$data.scaleNum, img.width);
          // c.setAttribute('width', img.width)
          // c.setAttribute('height', img.height)
          //  绘制图片
          ctx.drawImage(img, 0, 0, img.width, img.height);
        }
      };
    },
    // 选择区域框坐标
    pathPointXY(event) {
      const e = event || window.event;
      const tmp1 = {
        x: e.offsetX / this.$data.scaleNum,
        y: e.offsetY / this.$data.scaleNum,
      };
      if (this.$data.mousePath.length !== 0) {
        this.$data.canvasObject.moveTo(
          this.$data.mousePath[this.$data.mousePath.length - 1].x,
          this.$data.mousePath[this.$data.mousePath.length - 1].y
        );
        this.$data.canvasObject.lineTo(tmp1.x, tmp1.y);
        this.$data.canvasObject.strokeStyle = "red";
        this.$data.canvasObject.lineWidth = 3;
        this.$data.canvasObject.stroke();
      }
      // 将坐标点存入队列
      this.$data.mousePath.push(tmp1);
      console.log(this.$data.mousePath);
    },
    // 绘制区域框
    endPathXY(event) {
      const e = event || window.event;
      const tmp2 = {
        x: e.offsetX / this.$data.scaleNum,
        y: e.offsetY / this.$data.scaleNum,
      };
      if (this.$data.mousePath.length > 1) {
        this.$data.canvasObject.moveTo(tmp2.x, tmp2.y);
        this.$data.canvasObject.lineTo(
          this.$data.mousePath[0].x,
          this.$data.mousePath[0].y
        );
        this.$data.canvasObject.strokeStyle = "red";
        this.$data.canvasObject.lineWidth = 3;
        this.$data.canvasObject.stroke();
        // 将坐标点存入队列
        this.$data.mousePath.push(tmp2);
        // 绘制区域编号
        this.$data.canvasObject.font = "80px Arial";
        this.$data.canvasObject.fillStyle = "yellow";
        this.$data.canvasObject.fillText(this.$data.count, tmp2.x, tmp2.y);
        // 完成一组路径坐标点存储并清空原有存储队列
        // 双击会产生两个坐标，清除最后一个
        this.$data.mousePath.pop();
        this.$data.paths.push({
          title: this.$data.count,
          path: this.$data.mousePath,
        });
        this.$data.count++;
        this.$data.textare = JSON.stringify(this.$data.paths)
      }
      this.$data.mousePath = [];
    },
    // 重置图片
    resetImage (){
      var d = this.$refs.canvasContext;
      d.setAttribute('width',document.documentElement.clientWidth),
      d.setAttribute('height', document.documentElement.clientHeight)
      console.log("重置画布")
      this.$data.paths = []
      this.$data.textare = ''
      this.initCanvas() 
    },
    // 下载路径信息到文件
    savePaths (){
      this.$data.textare = JSON.stringify(this.$data.paths)
      console.log(this.$data.paths)
      ipcRenderer.send("pathsInfo", [this.msg.name, this.$data.textare]);
      // 主进程写入文件成功的日志
      ipcRenderer.on("pathsInfoBack", (e, data) => {
        console.log("BackInfo:",data)
        this.$data.alert = true
        
        setTimeout(() => {
          this.$data.alert = false
        }, 2000 )
      })
    }
  },  
};
</script>

<style scoped>
.cardClass{
  padding: 8px;
}
</style>