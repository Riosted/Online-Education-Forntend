<template>
  <div>
    <el-container>
      <el-main>
        <el-tabs v-model="activeName" @tab-click="handleClick">
          <el-tab-pane label="课程介绍" name="first">
            <div class="collect">
              <el-button v-if="collected" type="primary" icon="el-icon-star-off" circle
                         @click="deleteCollect"></el-button>
              <el-button v-if="!collected" icon="el-icon-star-off" circle @click="Collect"></el-button>
            </div>
            <div class="content">
              <div class="content-Title">
                <i class="el-icon-menu"></i>
                <h3>课程概述</h3>
              </div>
              {{ this.$store.state.courseData.intro }}
            </div>
          </el-tab-pane>
          <el-tab-pane label="课程评价" name="second" style="position: relative">

            <el-card :body-style="{ padding: '20px'}" v-show="evaluateBox"
                     style="position: relative;margin: 20px;overflow: hidden;min-height: 100px">
              <classevaluate :evalabel="evaluateLabel" :colors="colors" @pass="fchange"></classevaluate>
              <div>
                <el-input type="textarea" class="Comment" v-model="textarea" placeholder="请输入内容"></el-input>
              </div>
              <el-button round style="float: right;margin: 20px" @click="commentCourse">评价课程</el-button>
            </el-card>

            <el-card :body-style="{ padding: '20px'}" v-show="evaluateBox2"
                     style="position: relative;margin: 20px;overflow: hidden;min-height: 100px">
              <classevaluate :evalabel="evaluateLabel" :colors="colors" @pass="fchange"></classevaluate>
              <div>
                <el-input type="textarea" class="Comment" v-model="textarea" placeholder="请输入内容"></el-input>
              </div>
              <div>
                <el-button round style="float: right;margin: 20px" @click="commentCourse">评价课程</el-button>
                <el-button round style="float: right;margin: 20px" @click="exitEditComment">取消</el-button>
              </div>
            </el-card>

            <el-card :body-style="{ padding: '5px'}" v-show="currentComment"
                     style="position: relative;margin: 5px;overflow: hidden;min-height: 80px">
              <div style="overflow: hidden">
                <div style="text-align:left;font-size: 12px;margin: 0 10px">
                  <div style="display: inline;margin-right: 20px"> {{ currentUserComment.time }}</div>
                  <div style="display: inline;margin-right: 20px"> 点赞数：{{ currentUserComment.likes }}</div>
                  <el-rate style="display: inline"
                           v-model="currentUserComment.commentMark"
                           disabled
                           show-score
                           text-color="#ff9900"
                           score-template="{value}">
                  </el-rate>
                </div>
              </div>
              <div style="overflow: hidden;text-align:left;margin: 10px 10px">
                <div style="display: inline;color: red;font-size: 16px"> 我的评价：</div>
                <div style="display:inline;font-size: 16px"> {{ currentUserComment.content }}</div>
                <div style="display:inline;width: 400px">
                  <el-button round style="float: right" @click="editComment">编辑</el-button>
                </div>
              </div>
            </el-card>


            <el-card :body-style="{ padding: '5px'}" v-for="(item,index) in comment" :key="index"
                     style="position: relative;margin: 5px;overflow: hidden;min-height: 80px">
              <div>
                <div style="text-align:left;font-size: 12px;margin: 0 10px">
                  <div style="display: inline;margin-right: 20px"> {{ item.time }}</div>

                  <el-rate style="display: inline"
                           v-model="item.commentMark"
                           disabled
                           show-score
                           text-color="#ff9900"
                           score-template="{value}">
                  </el-rate>
                  <div style="float: right;margin-top: 20px">
                    <el-button circle class="commentStar"
                               @click="commentStar(item,index)">
                      <img src="@/assets/damuzhi.png" width="20px" height="20px">
                    </el-button>
                    <div style="text-align: center"> {{ item.likes }}</div>
                  </div>
                </div>
              </div>
              <div style="overflow: hidden;margin: 10px 10px">
                <div style="float:left;text-align:left;font-size: 16px"> {{ item.content }}</div>
              </div>
            </el-card>
          </el-tab-pane>
        </el-tabs>
      </el-main>
      <el-aside>
        <div>
          <h3>授课老师</h3>
        </div>
        <div class="Teacher" v-if="this.$store.state.courseData">
          <img :src="this.$store.state.courseData.teacher.teacherPicUrl" alt="图片缺失"
               @click="Teacher($store.state.courseData.teacherId)">
          <div class="Teacher-Introduce">
            <div class="TeacherName" @click="Teacher($store.state.courseData.teacherId)">
              {{ this.$store.state.courseData.teacher.name }}
            </div>
            <div class="TeacherSchool">
              {{ this.$store.state.courseData.teacher.school }}
            </div>
          </div>
        </div>
      </el-aside>
    </el-container>
    <div class="RelatedCourses">
      <div class="RelatedCourses-Title">
        <h3>
          相关课程
        </h3>
      </div>
      <div class="Course-content">
        <Menu v-for="(item,index) in this.$store.state.RelatedCourses"
              v-bind:key="index" @click.native="Course(item.courseId)"
              :name="item.name" :teacherName="item.teacher.name" :coursePic="item.coursePic" :isFree="1"
              :VIP="item.needVip" :courseWatches="item.courseWatches" :courseAvgMark="item.courseAvgMark"
              style="width: 18%"></Menu>
      </div>
    </div>
  </div>
</template>

<script>
import classevaluate from "./childcpn/classevaluate";
import axios from "axios";
import Menu from "@/components/childcpn/Menu";

export default {
  name: "CourseIntroduce",
  components: {
    Menu,
    classevaluate,
  },
  data() {
    return {
      activeName: 'first',
      value: [null],
      colors: ['#99A9BF', '#F7BA2A', '#FF9900'],
      textarea: '',
      evaluateLabel: ["课程打分"],
      comment: this.$store.state.commentData,
      evaluateBox: true,
      evaluateBox2: false,
      currentComment: false,
      currentUserComment: {},
      page: this.$store.state.commentData.total_element,
      collected: false,
    };
  },
  watch: {
    '$route'(to, from) {
      this.GetComment();
      this.displayComment();
      this.initCollect();
    }
  },
  created() {
    this.GetComment();
    this.displayComment();
    this.initCollect();
  },
  methods: {
    Teacher(teacherId) {
      this.$router.push({path: '/TeacherInfo', query: {teacherId: teacherId}});
    },
    initCollect() {
      if (this.$store.state.isLogin) {
        let a = new URLSearchParams;
        let that = this;
        a.append("courseId", this.$route.query.courseId);
        a.append("user_id", this.$store.state.userData.userId);
        axios.post("http://" + this.Api + "/api/Student/getLikeByStudentAndCourse?" + a, null, {
          headers: {
            'Content-Type': 'application/json',
            'Authorization': this.$store.state.JWT,
          }
        }).then(function (response) {
          console.log("获取学生是否收藏了该课程", response);
          if (response.data.data == null) {
            that.collected = false;
          } else {
            that.collected = true;
          }
        }, function (err) {
          console.log(err);
        });
      }
    },
    deleteCollect() {
      let that = this;
      let a = new URLSearchParams;
      a.append("courseId", this.$route.query.courseId);
      a.append("user_id", this.$store.state.userData.userId);
      axios.post("http://" + this.Api + "/api/Student/cancelLikeCourse?" + a, null, {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': this.$store.state.JWT,
        }
      }).then(function (response) {
        console.log("成功删除收藏记录", response);
        that.collected = false;
      }, function (err) {
        console.log(err);
      });
    },
    Collect() {
      if (this.$store.state.isLogin) {
        let a = new URLSearchParams;
        let that = this;
        a.append("courseId", this.$route.query.courseId);
        a.append("user_id", this.$store.state.userData.userId);
        axios.post("http://" + this.Api + "/api/Student/likeCourse?" + a, null, {
          headers: {
            'Content-Type': 'application/json',
            'Authorization': this.$store.state.JWT,
          }
        }).then(function (response) {
          console.log("收藏课程", response);
          that.collected = true;
        }, function (err) {
          console.log(err);
        });
      } else {
        this.$message.error('请登录后收藏课程');
      }
    },
    Course(courseId) {
      this.$router.push({path: '/course', query: {courseId: courseId}});
    },
    async commentStar(item, index) {
      let that = this;
      if (that.$store.state.isLogin !== false) {
        let JWT = that.$store.state.JWT;
        document.getElementsByClassName('commentStar')[index].classList.toggle("pink");
        if (document.getElementsByClassName('commentStar')[index].classList[4] !== "pink") {          //取消点赞
          let a = new URLSearchParams();
          a.append('commentId', item.commentId);
          a.append('user_id', that.$store.state.userData.userId);
          item.likes = item.likes - 1;
          await axios.post("http://" + that.Api + "/api/Student/dislikeComment", a, {
            headers: {
              'Authorization': JWT,
            }
          }).then(function (response) {
            // console.log(response);
            if (response.data.code === 1000) {
              console.log("取消点赞成功");
            }
          }, function (err) {
            console.log(err);
          });
        } else {  //点赞
          let a = new URLSearchParams();
          a.append('commentId', item.commentId);
          a.append('user_id', that.$store.state.userData.userId);
          item.likes++;
          await axios.post("http://" + that.Api + "/api/Student/likeComment", a, {
            headers: {
              'Authorization': JWT,
            }
          }).then(function (response) {
            // console.log(response);
            if (response.data.code === 1000) {
              console.log("点赞成功");
            }
          }, function (err) {
            console.log(err);
          });
        }
      } else {
        that.$message.error('请登陆后评论');
      }
    },
    editComment() {
      this.textarea = this.currentUserComment.content;
      this.evaluateBox2 = true;
      this.currentComment = false;
    },
    exitEditComment() {
      this.textarea = '';
      this.evaluateBox2 = false;
      this.currentComment = true;
    },
    handleClick(tab, event) {
      console.log(tab, event);
    },
    fchange(num, index) {               //接受子组件传递来的评价
      this.value = num;
      console.log(this.value, index);
    },
    commentCourse() {
      let that = this;
      if (that.value[0] === null || that.textarea === '') {
        that.$notify.info({
          title: '警告',
          message: '请输入评价评分',
          type: 'warning',
        });
      } else {
        that.$confirm('确定要提交评论吗？')
            .then(_ => {
              let that = this;
              let JWT = that.$store.state.JWT;
              let params = {
                comment: that.textarea,
                commentMark: that.value[0],
                courseId: that.$route.query.courseId,
                studentId: that.$store.state.userData.userId,
              };
              axios.post("http://" + that.Api + "/api/Student/commentCourseByCourseId", params, {
                headers: {
                  'Content-Type': 'application/json',
                  'Authorization': JWT,
                }
              }).then(function (response) {
                // console.log(response);
                if (response.data.code === 1000) {
                  console.log("提交成功");

                  let b = new URLSearchParams();
                  b.append('course_id', that.$route.query.courseId);
                  b.append('user_id', that.$store.state.userData.userId);
                  axios.post("http://" + that.Api + "/api/Student/getCourseCommentByStudentAndCourse", b, {
                    headers: {
                      'Authorization': JWT,
                    }
                  }).then(function (response) {
                    // console.log(response);
                    if (response.data.code === 1000) {
                      if (response.data.data == null) ;
                      else {
                        that.currentUserComment = response.data.data;
                        that.evaluateBox = false;
                        that.evaluateBox2 = false;
                        that.currentComment = true;
                        that.displayComment();
                      }
                      that.comment = that.$store.state.commentData;

                    }
                  }, function (err) {
                    console.log(err);
                  });
                }
              }, function (err) {
                console.log(err);
              });
            });
      }
    },
    //发送评论
    async displayComment() {
      let that = this;
      let JWT = that.$store.state.JWT;
      let a = new URLSearchParams();
      a.append('courseId', that.$route.query.courseId);
      a.append('page', 1);
      a.append('sort', 1);
      await axios.post("http://" + that.Api + "/api/Course/getCourseComments", a).then(function (response) {
        if (response.data.code === 1000) {
          that.$store.commit("saveCommentData", response.data.data.list);
          that.comment = that.$store.state.commentData;

          if (that.$store.state.isLogin !== false) {
            let clist = new Array();
            console.log("查看全部评论：", that.comment);
            let c = that.comment;
            for (let i = 0; i < c.length; i++) {
              clist[i] = c[i].commentId;
            }
            console.log("查看评论的ID:", clist);
            let pa = new URLSearchParams();
            let params = clist;
            // pa.append('commentIds', JSON.stringify(clist));
            let id = that.$store.state.userData.userId;
            pa.append('user_id', that.$store.state.userData.userId);
            axios.post("http://" + that.Api + "/api/Student/getStudentIsLikedComments?user_id=" + id, params, {
              headers: {
                'Content-Type': 'application/json',
                'Authorization': JWT,
              }
            }).then(function (response) {
              console.log("查看点赞成功", response);
              if (response.data.code === 1000) {
                for (let j = 0; j < c.length; j++) {
                  for (let k = 0; k < response.data.data.length; k++) {
                    if (c[j].commentId === response.data.data[k].commentId) {
                      document.getElementsByClassName('commentStar')[j].classList.add("pink");
                    }
                  }
                }
              }
            }, function (err) {
              console.log(err);
            });
          }

        }
      }, function (err) {
        console.log(err);
      });
    },
    GetComment() {
      let that = this;
      let JWT = that.$store.state.JWT;
      if (that.$store.state.isLogin !== false) {
        let b = new URLSearchParams();
        b.append('course_id', that.$route.query.courseId);
        b.append('user_id', that.$store.state.userData.userId);
        axios.post("http://" + that.Api + "/api/Student/getCourseCommentByStudentAndCourse", b, {
          headers: {
            'Authorization': JWT,
          }
        }).then(function (response) {
          console.log("获取评论成功");
          if (response.data.code === 1000) {
            if (response.data.data == null) ;
            else {
              that.currentUserComment = response.data.data;
              that.evaluateBox = false;
              that.currentComment = true;
            }
            that.comment = that.$store.state.commentData;
          }
        }, function (err) {
          console.log(err);
        });
      } else {
        that.evaluateBox = false;
        console.log("关闭评论")
      }
    }
  },
};
</script>

<style scoped>
.el-aside {
  border-radius: 10px;
  border: #C0C4CC 1px solid;
  color: #333;
  width: 300px;
}

.el-main {
  text-align: center;
  border-radius: 10px;
  border: 1px #C0C4CC solid;
  color: #333;
  line-height: 30px;
}

.collect {
  float: right;
  margin: 5px;
}

.pink {
  background-color: #00CCFF;
}

.el-container {
  margin: 50px auto;
  width: 1100px;
}

.Teacher {
  text-align: left;
  display: flex;
}

.Teacher img {
  cursor: pointer;
  height: 100px;
  width: 100px;
  margin: 10px 10px;
}

.TeacherName {
  cursor: pointer;
  font-size: 20px;
  margin: 10px;
}

.TeacherSchool {
  margin: 10px;
  font-size: 14px;
}

.content {
  text-align: left;
}

.content-Title {
  display: flex;
}

.content-Title i {
  margin: 35px 0 10px 10px;
}

.content-Title h3 {
  margin: 30px 10px 10px 10px;
}

.RelatedCourses {
  margin: 50px auto;
  width: 1100px;
}

.RelatedCourses-Title {
  margin: 50px 20px 30px 20px;
  text-align: left;
}

.Course-content {
  display: flex;
  flex-wrap: wrap;
}

.Comment textarea.el-textarea__inner {
  padding: 10px 10px 90px 10px;
}
</style>
