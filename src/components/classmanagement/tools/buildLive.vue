<template>
  <div>
    <el-main>
      <div style="width: 550px">
        <el-form label-width="160px" :key="key">
          <el-form-item label="直播名称" style="width: 300px">
            <el-input v-model="formBuild.name"
                      placeholder="请输入直播名称"
                      maxlength="10"
                      show-word-limit></el-input>
          </el-form-item>
          <el-form-item>
            <span slot="label" class="emphasize">上传直播介绍：</span>
            <el-input
                type="textarea"
                :rows="3"
                placeholder="请输入内容"
                v-model="formBuild.intro">
            </el-input>
          </el-form-item>
          <el-form-item label="选择直播日期">
            <el-date-picker style="float: left;margin: 10px 24px"
                            v-model="formBuild.liveDate"
                            align="right"
                            placeholder="选择日期"
                            @change="checkArrange"
                            :picker-options="pickerOptions"
                            value-format="yyyy-MM-dd">
            </el-date-picker>
          </el-form-item>
          <el-form-item label="选择直播时间" v-show="timeView">
            <el-select v-model="formBuild.arrange" @change="checkAddress" placeholder="请选择时间"
                       style="float: left;margin: 10px 24px">
              <el-option
                  v-for="item in options1"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                  :disabled="item.disabled"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="选择直播线路" v-show="arrangeView">
            <el-select v-model="formBuild.addressId" placeholder="请选择时间" style="float: left;margin: 10px 24px">
              <el-option
                  v-for="item in options2"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                  :disabled="item.disabled"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="上传直播封面" style="width: 550px">
            <el-col class="normal" :span="24"> 课程封面只能上传一张图片，且上传图片的大小最大为2MB</el-col>
          </el-form-item>
          <el-upload
              class="avatar-uploader"
              action=""
              :show-file-list="false"
              :http-request="uploadHttp">
            <img v-if="imageUrl" :src="imageUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
          <el-form-item style="position: relative;margin: 0;padding: 0">
            <el-button type="primary" class="headerbutton" @click="closed" :loading="loading"
                       style="position: absolute;left: 60px">
              {{ loading ? '提交中 ...' : '确认上传' }}
            </el-button>
          </el-form-item>
        </el-form>
      </div>
    </el-main>
  </div>
</template>

<script>

import ossClient from "@/aliyun.oss.client";
import axios from "axios";

export default {

  name: "buildClass",
  props: {
    liveId: {
      type: Number,
    },
    isEdit: {
      type: Boolean,
      default: false,
    },
    liveInfo: {},
  },
  data() {
    return {
      timeView: false,
      arrangeView: false,
      pickerOptions: {                   //可选直播的时间
        disabledDate(time) {
          let n = time.getTime() < Date.now() - 3600 * 1000 * 24;
          let m = time.getTime() > Date.now() + 3600 * 1000 * 24 * 7;
          let c = n | m;
          return c;
        },
      },
      dayLive: [],
      imageUrl: '',
      loading: false,
      formBuild: {              //课程基本属性
        addressId: null,
        name: '',
        intro: '',
        liveDate: '',
        arrange: null,
        livePicUrl: '',
      },
      options1: [{
        value: 1,
        label: ' 8:00',
        disabled: false
      }, {
        value: 2,
        label: '10:00',
        disabled: false
      }, {
        value: 3,
        label: '14:00',
        disabled: false
      }, {
        value: 4,
        label: '16:00',
        disabled: false
      }, {
        value: 5,
        label: '18:00',
        disabled: false
      }],
      options2: [{
        value: 1,
        label: '线路一',
        disabled: false
      }, {
        value: 2,
        label: '线路二',
        disabled: false
      }],
      uploadConf: {
        endpoint: "https://oss-accelerate.aliyuncs.com",
        region: 'oss-cn-shanghai',
        accessKeyId: 'LTAI4GGsTQ35tQcWWDVNKwqG',
        accessKeySecret: 'reWjqrK73PE0ZvJQH0Hwjr9eyuWbuc',
        bucket: 'shu-online-edu',
      },
      key: 1,
    };
  },
  methods: {
    async checkArrange(val) {
      let that = this;
      console.log("选择日期", val);
      let JWT = that.$store.state.JWT;
      that.options1.map((val, index) => {
        that.options1[index].disabled = false;
      });
      let b = new URLSearchParams();
      b.append("liveDate", val);
      await axios.post("http://" + that.Api + "/api/Live/findArrangeIsValidInDay", b, {
        headers: {
          'Authorization': JWT,
        }
      }).then(function (response) {
        console.log("查看当天直播安排成功", response);
        let n = [{arrange: 0, address: 0}, {arrange: 0, address: 0}, {arrange: 0, address: 0}, {
          arrange: 0,
          address: 0
        }, {arrange: 0, address: 0}];
        console.log(n);
        response.data.data.map((item) => {
          if (item.liveCount >= 1) {
            n[item.liveArrange - 1].address = item.liveAddressId;
            n[item.liveArrange - 1].arrange++;
          }
        });
        that.dayLive = n;
        n.map((item, index) => {
          if (item.arrange > 1) {
            that.options1[index].disabled = true;
          }
        });
        that.key++;
        that.timeView = true;
        console.log(that.options1);
      }, function (err) {
        console.log(err);
      });
    },
    checkAddress(val) {
      console.log("111");
      let that = this;
      that.options2.map((val, index) => {
        console.log(index);
        that.options2[index].disabled = false;
      });
      if (that.dayLive[val - 1].address !== 0) {
        that.options2[that.dayLive[val - 1].address - 1].disabled = true;
      }
      that.arrangeView = true;
      console.log(that.options2);
      that.key++;
    },
    async uploadHttp({file}) {
      let that = this;
      let f = await this.$Api.compressImg(file);
      console.log(f);
      let fileName = `${this.$store.state.userData.userId}_Header/${file.name}`;  //定义唯一的文件名
      fileName = `pic/Live/` + fileName;
      ossClient(this.uploadConf).put(fileName, f, {
        'ContentType': 'image/jpeg'
      }).then(({res, url, name}) => {
        if (res && res.status == 200) {
          that.imageUrl = url;
          that.formBuild.livePicUrl = url;
          console.log(`阿里云OSS上传图片成功回调`, res, url, name);
        }
      }).catch((err) => {
        console.log(`阿里云OSS上传图片失败回调`, err);
      });
    },
    async closed() {
      let that = this;
      let params = {
        addressId: that.formBuild.addressId,
        liveArrange: that.formBuild.arrange,
        liveDate: that.formBuild.liveDate,
        liveIntro: that.formBuild.intro,
        liveName: that.formBuild.name,
        livePicUrl: that.formBuild.livePicUrl,
        teacherId: that.$store.state.userData.userId,
      };
      console.log(params);
      if (that.isEdit === true) {
        let JWT = that.$store.state.JWT;
        await axios.post("http://" + that.Api + "/api/Live/modifyLive?liveId=" + that.liveId, params, {
          headers: {
            'Content-Type': 'application/json',
            'Authorization': JWT,
          }
        }).then(function (response) {
          if (response.data.code === 3005) {
            this.$message.error('此时间段已被预约满,请换个时间段');
          } else if (response.data.code === 1000) {
            console.log("修改成功", response);
            that.arrangeView = false;
            that.timeView = false;
            that.$emit('close', false);
            that.$alert('修改成功', '提示', {
              confirmButtonText: '确定',
            });
          }
        }, function (err) {
          console.log(err);
        });
      } else {
        let JWT = that.$store.state.JWT;
        await axios.post("http://" + that.Api + "/api/Live/addLive", params, {
          headers: {
            'Content-Type': 'application/json',
            'Authorization': JWT,
          }
        }).then(function (response) {
          if (response.data.code === 3005) {
            this.$message.error('此时间段已被预约满,请换个时间段');
          } else if (response.data.code === 1000) {
            console.log("创建成功", response);
            that.$emit('close', false);
            that.$alert('创建成功', '提示', {
              confirmButtonText: '确定',
            });
          }
          that.$emit('close', false);
        }, function (err) {
          console.log(err);
        });
      }
    }
  },
  mounted() {
    let that = this;
    if (that.isEdit) {
      that.formBuild.name = that.liveInfo.liveName;
      that.formBuild.intro = that.liveInfo.intro;
      that.formBuild.liveDate = that.liveInfo.liveDate;
      that.formBuild.addressId = that.liveInfo.addressId;
      that.formBuild.arrange = that.liveInfo.liveArrange;
      that.formBuild.livePicUrl = that.liveInfo.livePicUrl;
      that.imageUrl = that.liveInfo.livePicUrl;
      that.timeView = true;
      that.arrangeView = true;
    }
  }
}
</script>

<style scoped>
.avatar-uploader-icon {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}

.avatar-uploader-icon:hover {
  border-color: #409EFF;
}

.avatar {
  width: 178px;
  height: 178px;
  display: block;
}

.headerbutton {
  cursor: pointer;
  position: relative;
  left: -140px;
  margin-top: 12px;
}


.emphasize {
  font-family: "Microsoft YaHei", "微软雅黑", "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB", Arial, sans-serif;
  font-size: 13px;
}

.el-tag + .el-tag {
  margin-left: 10px;
}

</style>
