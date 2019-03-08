<script>
import Aplayer from "vue-aplayer";
const axios = require("axios");

export default {
  props: {
    initCount: Number,
    initActive: Boolean,
    iconImg: String
  },
  components: {
    Aplayer
  },
  computed: {
    uploadAction() {
      if (this.currentMode === "vocal") {
        return "http://101.201.28.186:8889/api/karaoke/upload/remove_voice";
      } else {
        return `http://101.201.28.186:8889/api/karaoke/upload/transpose_voice?semitone=${
          this.semitone
        }`;
      }
    }
  },
  data() {
    return {
      previewUrls: [],
      previewWork: {},
      trackOrderAppear: false,
      form: {},
      isLoading: false,
      currentMode: "vocal",
      file_id: "",
      semitone: "-1",
      options: [
        {
          value: "+1",
          label: "+1"
        },
        {
          value: "+2",
          label: "+2"
        },
        {
          value: "-1",
          label: "-1"
        },
        {
          value: "-2",
          label: "-2"
        }
      ]
    };
  },
  watch: {},
  created() {
    console.log(this.$route.params.orderId);
    if (this.$route.params.orderId) {
      this.trackOrder(this.$route.params.orderId);
    }
  },
  mounted() {},
  updated() {},
  beforeRouteEnter(to, from, next) {
    // i think need check here, e.g. cookie address
    next();
  },
  methods: {
    confirmOrder() {
      //to Ali
      axios
        .post(
          `http://101.201.28.186:8889/api/karaoke/order/remove_voice?file_id=${
            this.file_id
          }`
        )
        .then(function(response) {
          // handle success
          console.log(response);
        })
        .catch(function(error) {
          // handle error
          this.$notify({
            title: "Warning",
            message: "This is a warning message",
            type: "warning"
          });

          console.log(error);
        })
        .then(function() {
          // always executed
        });
    },
    trackOrder(orderId) {
      const order_id = orderId || this.form.orderId;
      console.log(order_id);
      this.isLoading = true;
      axios
        .get(
          `http://101.201.28.186:8889/api/karaoke/query?order_id=${order_id}&alipay_order_id=`
        )
        .then(res => {
          console.log(res);
          location.href = `http://${res.data.data}`;
        })
        .catch(() => {
          this.$notify({
            title: "Warning",
            message: "This is a warning message",
            type: "warning"
          });
        })
        .then(() => {
          this.isLoading = false;
          this.trackOrderAppear = false;
        });
    },
    handleSemitoneChange(v) {
      console.log(v);
    },
    handleRemove() {},
    handleProgress(env, file, fileList) {},
    handleChange(file, fileList) {},
    handleError(err) {
      this.$notify({
        title: "Warning",
        message: "This is a warning message",
        type: "warning"
      });
    },
    handleBeforeUpload() {
      console.log("zouaddd");
      this.isLoading = true;
      return true;
    },
    handleSuccess(res) {
      // this.previewUrls = [res.data.preview_url].map(item => {
      //   return {
      //     title: "预览",
      //     artist: "",
      //     src: item,
      //     pic: ""
      //   };
      // });
      this.isLoading = false;
      console.log(res);
      this.previewWork = {
        title: `${this.currentMode === "vocal" ? "去人声" : "转调"}预览`,
        artist: "va",
        src: `http://${res.data.preview_url}`,
        pic: "//2ndsenseaudio.com/wp-content/uploads/2015/10/logo@2x.png"
      };
      this.$refs.upload.clearFiles();
      console.log(this.previewWork);
      this.file_id = res.data.id;
    },
    handlePreview() {},
    previewLinks() {},
    updateMode(des) {
      this.currentMode = des;
    }
  }
};
</script>

<template>
  <div id="app">
    <el-container>
      <el-header>
        <!-- <router-link to="/">Home</router-link>|
        <router-link to="/about">About</router-link>-->
        <div
          style="position:relative;width:100%;background-color:orange;line-height:60px;"
        >Karaoke Gen</div>
        <span
          style="position:absolute;font-size:14px; color:white;width:10%;line-height:60px;top:0;right:0;cursor:pointer;"
          @click="trackOrderAppear = true"
        >找回订单</span>
      </el-header>
      <el-main v-loading="isLoading" element-loading-text="正在为您生成文件">
        <div class="function-section">
          <el-row :gutter="24">
            <el-col :span="12">
              <div
                class="productBtn"
                v-bind:class="{ active: currentMode == 'vocal' }"
                @click="updateMode('vocal')"
              >
                <h3>自动去人声</h3>
                <p>点击鼠标，一键生成伴奏</p>
              </div>
            </el-col>
            <el-col :span="12">
              <div
                class="productBtn"
                v-bind:class="{ active: currentMode == 'transpose' }"
                @click="updateMode('transpose')"
              >
                <h3>升调&降调</h3>
                <p>唱不上去的朋友有福了</p>
                <span style="position:absolute;bottom:5px;right:5%;">
                  <el-select v-model="semitone" placeholder="半音个数" :change="handleSemitoneChange">
                    <el-option
                      v-for="item in options"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value"
                    ></el-option>
                  </el-select>
                </span>
              </div>
            </el-col>
          </el-row>
        </div>
        <div class="upload-section">
          <el-upload
            ref="upload"
            class="upload-demo"
            drag
            :action="uploadAction"
            :on-preview="handlePreview"
            :limit="1"
            :on-change="handleChange"
            :on-progress="handleProgress"
            :on-remove="handleRemove"
            :on-success="handleSuccess"
            :on-error="handleError"
            :before-upload="handleBeforeUpload"
          >
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">
              拖拽音频文件或
              <em>点击上传</em>
            </div>
            <div class="el-upload__tip" slot="tip">上传待处理音频</div>
          </el-upload>
        </div>
        <div class="preview-section" v-if="previewWork.src">
          <aplayer autoplay :music="previewWork"/>
        </div>
        <div class="pay-section">
          <el-button type="primary" plain v-show="previewWork.src" @click="confirmOrder">确认购买</el-button>
        </div>

        <el-dialog
          element-loading-text="正在为您找回文件"
          title="找回订单"
          v-loading="isLoading"
          :visible.sync="trackOrderAppear"
        >
          <el-form :model="form">
            <el-form-item label="订单号" label-width="120px">
              <el-input v-model="form.orderId" autocomplete="off"></el-input>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="trackOrderAppear = false">取消</el-button>
            <el-button type="primary" @click="trackOrder(form.orderId)">确定</el-button>
          </span>
        </el-dialog>
      </el-main>
      <!-- <el-footer>Footer</el-footer> -->
    </el-container>
    <!-- <router-view /> -->
  </div>
</template>

<style lang="scss">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.active {
  background-color: orange !important;
}
.productBtn {
  // display: flex;
  // flex-direction: column;
  // align-items: center;
  // justify-content: center;
  cursor: pointer;
  position: relative;
  height: 80px;
  background-color: #2c3e50;
  color: antiquewhite;
  h3 {
    line-height: 40px;
    padding-top: 6px;
  }
}
.upload-section {
  margin-top: 20px;
  padding: 20px 0;
  position: relative;
  width: 100%;
  background-color: gray;
}
.preview-section {
  margin-top: 20px;
  padding: 20px 0;
  width: 100%;
  background-color: gray;
}
#nav {
  padding: 30px;
  a {
    font-weight: bold;
    color: #2c3e50;
    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
