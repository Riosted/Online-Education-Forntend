<template>
  <div>
    <el-container v-if="isLogin">
      <el-aside>
        <img :src="information.studentPicUrl" alt="图片缺失">
        <h3>{{ information.nickName }}</h3>
      </el-aside>
      <el-main class="Information">
        <h2>个人信息</h2>
        <el-button type="primary" @click="ChangeInformation">修改完成</el-button>
        <div class="clear"></div>
        <el-divider></el-divider>
        <el-form ref="form" :model="information" label-width="100px">
          <el-form-item label="头像：">
            <el-upload
                class="avatar-uploader"
                action=""
                :show-file-list="false"
                :http-request="uploadHttp">
              <img v-if="imageUrl" :src="imageUrl" class="avatar">
              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
          </el-form-item>
          <el-form-item label="昵称：">
            <el-input v-model="information.nickName"></el-input>
          </el-form-item>
          <el-form-item label="性别：">
            <el-select v-model="information.sex" clearable placeholder="请选择你的性别" @change="changeSex">
              <el-option
                  v-for="item in sex"
                  :key="item"
                  :label="item"
                  :value="item">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="学校：">
            <el-input v-model="information.school"></el-input>
          </el-form-item>
          <el-form-item label="专业：">
            <el-select v-model="information.majorContent" clearable placeholder="请选择你的专业" @change="changeMajor">
              <el-option
                  v-for="item in this.$store.state.MajorPrefer"
                  :key="item.majorId"
                  :label="item.majorContent"
                  :value="item.majorId">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="年级：">
            <el-select v-model="information.grade" clearable placeholder="请选择你的年级" @change="changeGrade">
              <el-option
                  v-for="item in grade"
                  :key="item"
                  :label="item"
                  :value="item">
              </el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </el-main>
    </el-container>
    <el-container v-if="isLoginTeacher">
      <el-aside>
        <img :src="informationTeacher.teacherPicUrl" alt="图片缺失">
        <h3>{{ informationTeacher.name }}</h3>
      </el-aside>
      <el-main class="Information">
        <h2>个人信息</h2>
        <el-button type="primary" @click="ChangeInformationTeacher">修改完成</el-button>
        <div class="clear"></div>
        <el-divider></el-divider>
        <el-form ref="form" :model="informationTeacher" label-width="100px">
          <el-form-item label="头像：">
            <el-upload
                class="avatar-uploader"
                action=""
                :show-file-list="false"
                :http-request="uploadHttp">
              <img v-if="imageUrl" :src="imageUrl" class="avatar">
              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
          </el-form-item>
          <el-form-item label="姓名：">
            <el-input v-model="informationTeacher.name"></el-input>
          </el-form-item>
          <el-form-item label="性别：">
            <el-select v-model="information.sex" clearable placeholder="请选择你的性别" @change="changeSex">
              <el-option
                  v-for="item in sexTeacher"
                  :key="item"
                  :label="item"
                  :value="item">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="学校：">
            <el-input v-model="informationTeacher.school"></el-input>
          </el-form-item>
          <el-form-item label="专业：">
            <el-select v-model="informationTeacher.majorContent" clearable placeholder="请选择你的专业" @change="changeMajor">
              <el-option
                  v-for="item in this.$store.state.MajorPrefer"
                  :key="item.majorId"
                  :label="item.majorContent"
                  :value="item.majorId">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="简介：">
            <el-input type="textarea" v-model="informationTeacher.intro"
                      :autosize="{ minRows: 2, maxRows: 4}"></el-input>
          </el-form-item>
        </el-form>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import axios from "axios";
import ossClient from "../aliyun.oss.client";

export default {
  name: "InformationChange",
  data() {
    return {
      information: {
        grade: this.$store.state.userData.grade,
        major: this.$store.state.userData.major,
        majorContent: this.$store.state.userData.major.majorContent,
        majorId: this.$store.state.userData.majorId,
        nickName: this.$store.state.userData.nickName,
        password: this.$store.state.userData.password,
        school: this.$store.state.userData.school,
        sex: this.$store.state.userData.sex,
        userId: this.$store.state.userData.userId,
        vip: this.$store.state.userData.vip,
        wechatId: this.$store.state.userData.wechatId,
        studentPicUrl: this.$store.state.userData.studentPicUrl,
      },
      informationTeacher: {
        intro: this.$store.state.userData.intro,
        major: this.$store.state.userData.major,
        majorContent: this.$store.state.userData.major.majorContent,
        majorId: this.$store.state.userData.majorId,
        name: this.$store.state.userData.name,
        password: this.$store.state.userData.password,
        school: this.$store.state.userData.school,
        sex: this.$store.state.userData.sex,
        userId: this.$store.state.userData.userId,
        wechatId: this.$store.state.userData.wechatId,
        teacherPicUrl: this.$store.state.userData.teacherPicUrl,
        teacherStatus: this.$store.state.userData.teacherStatus,
      },
      isLogin: this.$store.state.isLogin,
      isLoginTeacher: this.$store.state.isLoginTeacher,
      sex: ['男', '女', '保密'],
      sexTeacher: ['男', '女'],
      grade: ['大一', '大二', '大三', '大四'],
      imageUrl: '',
      uploadConf: {
        endpoint: "https://oss-accelerate.aliyuncs.com",
        region: 'oss-cn-shanghai',
        accessKeyId: 'LTAI4GGsTQ35tQcWWDVNKwqG',
        accessKeySecret: 'reWjqrK73PE0ZvJQH0Hwjr9eyuWbuc',
        bucket: 'shu-online-edu',
      },
    };
  },
  methods: {
    async uploadHttp({file}) {
      console.log(file);
      let that = this;
      let f = await this.$Api.compressImg(file);
      console.log(f);
      let fileName = `${this.$store.state.userData.userId}_Header/${file.name}`;  //定义唯一的文件名
      if (this.isLogin) {
        fileName = `pic/Student/` + fileName;
      } else {
        fileName = `pic/Teacher/` + fileName;
      }
      ossClient(this.uploadConf).put(fileName, f, {
        'ContentType': 'image/jpeg'
      }).then(({res, url, name}) => {
        if (res && res.status == 200) {
          that.imageUrl = url;
          that.information.studentPicUrl = url;
          that.informationTeacher.teacherPicUrl = url;
          console.log(`阿里云OSS上传图片成功回调`, res, url, name);
        }
      }).catch((err) => {
        console.log(`阿里云OSS上传图片失败回调`, err);
      });
    },
    changeMajor(value) {
      console.log(value);
      this.information.majorId = value;
      this.informationTeacher.majorId = value;
    },
    changeSex(value) {
      console.log(value);
      this.information.sex = value;
      this.informationTeacher.sex = value;
    },
    changeGrade(value) {
      console.log(value);
      this.information.grade = value;
    },
    ChangeInformation() {
      let that = this;
      let params = {
        grade: this.information.grade,
        majorId: this.information.majorId,
        nickname: this.information.nickName,
        picUrl: this.information.studentPicUrl,
        school: this.information.school,
        sex: this.information.sex,
      };
      let a = new URLSearchParams();
      a.append('user_id', this.information.userId);
      axios.post("http://" + this.Api + "/api/Student/completeStudentById?" + a, params, {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': this.$store.state.JWT,
        }
      }).then(function (response) {
        console.log(response);
        axios.post("http://" + that.Api + "/api/Student/getStudentById?" + a, null, {
          headers: {
            'Content-Type': 'application/json',
            'Authorization': that.$store.state.JWT,
          }
        }).then(function (res) {
          console.log(res);
          that.$router.push('/information');
          that.$store.commit('saveData', res.data.data);
        }, function (err) {
          console.log(err);
        });
      }, function (err) {
        console.log(err);
      });
    },
    ChangeInformationTeacher() {
      let that = this;
      let params = {
        intro: this.informationTeacher.intro,
        majorId: this.informationTeacher.majorId,
        name: this.informationTeacher.name,
        school: this.informationTeacher.school,
        sex: this.informationTeacher.sex,
        picUrl: this.informationTeacher.teacherPicUrl,
      };
      let a = new URLSearchParams();
      a.append('user_id', this.informationTeacher.userId);
      axios.post("http://" + this.Api + "/api/Teacher/completeTeacherById?" + a, params, {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': this.$store.state.JWT,
        }
      }).then(function (response) {
        console.log(response);
        axios.post("http://" + that.Api + "/api/Teacher/getTeacherById?" + a, null, {
          headers: {
            'Content-Type': 'application/json',
            'Authorization': this.$store.state.JWT,
          }
        }).then(function (res) {
          console.log(res);
          that.$router.push('/information');
          that.$store.commit('saveData', res.data.data);
        }, function (err) {
          console.log(err);
        })
      }, function (err) {
        console.log(err);
      });
    },
  },
};
</script>

<style scoped>
.el-container {
  margin: 50px auto;
  width: 1100px;
}

.el-aside {
  border-radius: 10px;
  border: #C0C4CC 1px solid;
  color: #333;
  text-align: center;
  line-height: 20px;
  width: 300px;
}

.el-aside img {
  margin-top: 30px;
  width: 150px;
  height: 150px;
}

.el-aside h3 {
  margin-bottom: 10px;
}

.el-main {
  border-radius: 10px;
  border: #C0C4CC 1px solid;
  color: #333;
  text-align: left;
  line-height: 30px;
}

body > .el-container {
  margin-bottom: 40px;
}

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

.el-input {
  width: 50%;
}

.el-textarea {
  width: 50%;
}

.el-select {
  width: 50%;
}

.Information h2 {
  float: left;
}

.Information button {
  float: right;
  margin: 20px;
}

.clear {
  clear: both;
}
</style>
