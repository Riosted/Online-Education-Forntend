<template>
  <div class="sidebar">
    <ul class="sidebar-list">
      <li class="sidebar-item">
        <span class="sidebar-text"><router-link to="/Feedback">意见反馈</router-link></span>
      </li>
      <li class="sidebar-item" @click="backToTop">
        <span class="sidebar-text">返回顶部</span>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: "Sidebar",
  methods: {
    backToTop() {
      document.documentElement.scrollTop = document.body.scrollTop = 0;
    }
  }
};
</script>

<style scoped>
.sidebar {
  z-index: 66;
  position: fixed;
  left: 0;
  padding: 0 16px;
  top: 50%;
  transform: translateY(-50%);
  background-color: #ffffff;
  border-radius: 5px;
}

.sidebar-item {
  padding: 16px 0;
  width: 26px;
  height: 60px;
  box-sizing: border-box;
  border-bottom: 1px solid #edf1f2;
  text-align: center;
  cursor: pointer;
}

.sidebar-text {
  font-size: 12px;
  line-height: 16px;
  color: #99a9bf;
}

.sidebar-list {
  list-style: none;
  padding-inline-start: 0;
  Padding-left: 0;
}

a:hover {
  text-decoration: none;
  color: #ffd04b;
}

a {
  text-decoration: none;
  color: #99a9bf;
}
</style>
