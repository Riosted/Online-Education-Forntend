<template>
  <div class="history">
    <div class="history-course" v-for="(item,index) in this.$store.state.StudentHistory.list" v-bind:key="index">
      <div class="history-time">
        <p>观看时间：{{ item.watchTime }}</p>
        <div class="history-icon">
          <el-button icon="el-icon-delete" circle @click="DeleteHistory(item.id)"></el-button>
        </div>
        <div class="history-teacher">
          <img v-if="item.course.teacher.teacherPicUrl" :src="item.course.teacher.teacherPicUrl" alt="图片缺失">
          <img v-if="!item.course.teacher.teacherPicUrl" src="../assets/SHUlogo.jpg" alt="图片缺失">
          <p>{{ item.course.teacher.name }} | {{ item.course.teacher.intro }}</p>
        </div>
      </div>
      <div class="history-content">
        <div class="history-img"
             @click="videoHistory(item.course.courseId)">
          <img v-if="item.course.coursePic" :src="item.course.coursePic" alt="图片缺失">
          <img v-if="!item.course.coursePic" src="../assets/SHUlogo.jpg" alt="图片缺失">
        </div>
        <div class="history-text">
          <div class="history-text-title"
               @click="videoHistory(item.course.courseId)">
            <h3>{{ item.course.name }}</h3>
          </div>
          <h4>{{ item.course.intro }}</h4>
          <h5>看到：{{ item.courseChapterVideo.videoName }}</h5>
        </div>
      </div>
    </div>
    <div class="block">
      <el-pagination
          background
          @current-change="handleCurrentChange"
          :current-page.sync="currentPage"
          layout="prev, pager, next"
          :total="this.$store.state.StudentHistory.total_element">
      </el-pagination>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "History",
  data() {
    return {
      currentPage: 1,
    };
  },
  created() {
    this.history();
  },
  methods: {
    handleCurrentChange(currentPage) {
      console.log("当前页", currentPage);
      let a = new URLSearchParams;
      let that = this;
      a.append("page", currentPage);
      a.append("user_id", this.$store.state.userData.userId);
      axios.post("http://" + this.Api + "/api/Student/findAllWatchRecords?" + a, null, {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': this.$store.state.JWT,
        }
      }).then(function (response) {
        that.currentPage = currentPage;
        console.log("当前学生观看历史", response);
        that.$store.commit('saveStudentHistory', response.data.data);
      }, function (err) {
        console.log(err);
      });
    },
    videoHistory(courseId) {
      this.$router.push({path: '/course', query: {courseId: courseId}});
    },
    history() {
      let a = new URLSearchParams;
      let that = this;
      a.append("page", 1);
      a.append("user_id", this.$store.state.userData.userId);
      axios.post("http://" + this.Api + "/api/Student/findAllWatchRecords?" + a, null, {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': this.$store.state.JWT,
        }
      }).then(function (response) {
        console.log("当前学生观看历史", response);
        that.$store.commit('saveStudentHistory', response.data.data);
      }, function (err) {
        console.log(err);
      });
    },
    DeleteHistory(watchRecordId) {
      let that = this;
      let a = new URLSearchParams;
      a.append("watchRecordId", watchRecordId);
      axios.post("http://" + this.Api + "/api/Student/deleteWatchRecords?" + a, null, {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': this.$store.state.JWT,
        }
      }).then(function (response) {
        console.log("成功删除观看历史", response);
        that.history();
        that.currentPage = 1;
      }, function (err) {
        console.log(err);
      });
    }
  },
};
</script>

<style scoped>
.history {
  margin: 50px auto;
  width: 1100px;
}

.history-course {
  text-align: left;
  margin: 20px auto;
  border-radius: 10px;
  border: #E4E7ED 1px solid;
  box-shadow: 0 0 3px rgba(95, 101, 105, 0.15);
  background: #F2F6FC;
}

.history-course:hover {
  background-color: #E4E7ED;
}

.history-time {
  text-align: right;
  float: right;
  font-size: 13px;
  margin: 20px 20px 0 0;
}

.history-icon {
  margin: 10px;
}

.history-teacher {
  display: flex;
}

.history-teacher img {
  border-radius: 50px;
  margin: 5px;
  height: 30px;
  width: 30px;
}

.history-content {
  display: flex;
  text-align: left;
}

.history-img {
  cursor: pointer;
  height: 130px;
  width: 230px;
  margin: 20px;
}

.history-img img {
  width: 100%;
  height: 100%;
}

.history-text {
  text-align: left;
}

.history-text-title {
  cursor: pointer;
}

.history-text h5 {
  color: #909399;
}

a {
  text-decoration: none;
  color: #1c1f21;
}

.block {
  margin-top: 20px;
  margin-bottom: 10px;
}

.clear {
  clear: both;
}
</style>
