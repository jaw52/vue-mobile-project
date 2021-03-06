<template>
  <div class="nav">
    <!--  使用router 中配置的meta字段，动态改变导航栏的标题 -->
    <van-nav-bar :title="$route.meta.title" left-text="返回" left-arrow @click-left="back">
      <van-cell-group slot="title" v-if="isSearch">
        <van-field v-model="searchVal" placeholder="搜索用户" />
      </van-cell-group>
      <div slot="right">
        <div class="btn search-btn" @touchstart="search">
          <i class="iconfont">&#xe611;</i>
        </div>
        <div :class="['btn',isPulldown?'touch-color':'']" @touchstart="isPulldown=!isPulldown">
          <i class="iconfont" v-if="!isPulldown">&#xe605;</i>
          <i class="iconfont" v-else>&#xe62c;</i>
        </div>
      </div>
    </van-nav-bar>
    <transition name="fade">
      <!-- 登录状态的下拉菜单 -->
      <div class="pull-down" v-if="isPulldown">
        <div v-if="$store.state.token">
          <div class="user-card" @touchstart="goToHome">
            <van-image fit="cover" class="avatar" :src="userInfo.headimg"></van-image>
            <span>{{pulldownNickname}}</span>
            <van-icon name="arrow"></van-icon>
          </div>
          <ul class="link-list">
            <li @touchstart="goTo(item.url)" v-for="item in linkData" :key="item.id">
              {{item.title}}
              <van-icon name="arrow"></van-icon>
            </li>
          </ul>
          <div class="bottom" @touchstart="logout">
            <div class="logout-btn">退出登陆</div>
          </div>
        </div>
        <!-- 退出登陆后的下拉菜单 -->
        <div v-else>
          <div class="btn-div">
            <router-link class="logout-btn" to="/registerone">注册</router-link>
            <router-link class="login-btn" to="/login">登陆</router-link>
          </div>
          <ul class="link-list">
            <li @touchstart="goTo(item.url)" v-for="item in LogoutLineData" :key="item.id">
              {{item.title}}
              <van-icon name="arrow"></van-icon>
            </li>
          </ul>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import "@/assets/animate.css";
export default {
  name: "Nav",
  data() {
    return {
      isPulldown: false,
      linkData: [
        {
          id: 0,
          title: "我的关注",
          url: "/follow"
        },
        {
          id: 1,
          title: "热门",
          url: "/browse"
        },
        {
          id: 2,
          title: "首页",
          url: "/"
        }
      ],
      /* 退出登陆后，导航栏下拉菜单内容 */
      LogoutLineData: [
        {
          id: 0,
          title: "热门",
          url: "/browse"
        }
      ],
      searchVal: "", //搜索值
      isSearch: false, //是否显示搜索框
      goSearch: false, //确认调转至搜索页面
      userInfo: {
        nickname: "",
        headimg: ""
      }
    };
  },
  computed: {
    pulldownNickname() {
      return this.userInfo.nickname.substring(0, 6) + "...";
    },
    isLogin() {
      return localStorage.getItem("Flag") ? true : false;
    }
  },
  methods: {
    //返回
    back() {
      this.$router.go(-1);
    },
    goTo(path) {
      if (this.$route.path == path) {
        this.$router.go(0);
        this.$router.push(path);
      } else {
        this.$router.push(path);
      }
    },
    /* 
				登出
				FIXME:使用token技术
			*/
    logout() {
      this.$store.dispatch("logout");
      this.$toast.success("退出成功");
      this.$router.go(0);
      this.$router.push("/login");
    },
    /* 调转至个人主页 */
    goToHome() {
      let nickname = this.userInfo.nickname;
      if (this.$route.path == "/user") {
        this.$router.go(0);
        this.$router.push({
          path: "/user",
          query: {
            nickname
          }
        });
      } else {
        this.$router.push({
          path: "/user",
          query: {
            nickname
          }
        });
      }
    },
    /* 搜索 */
    search() {
      if (this.goSearch) {
        this.$router.push({
          path: "/browse",
          query: {
            searchVal: this.searchVal
          }
        });
        if (this.$route.path == "/browse") {
          this.$router.go(0);
        }
      } else {
        this.isSearch = true;
        this.goSearch = true;
      }
    },
    getUserInfo() {
      if(!this.$store.state.token) return
      this.$store
        .dispatch("getUserInfo")
        .then(res => {
          this.userInfo.nickname = res.nickname;
          this.userInfo.headimg = res.headimg;
        })
        .catch(err => {
          console.log(err);
        });
    }
  },
  mounted() {
    this.getUserInfo()
  },
  watch: {
    /* 路由变化时执行该方法 */
    $route: {
      handler() {
        this.isPulldown = false;
        this.getUserInfo()
      }
    }

  }
};
</script>

<style scoped="scoped">
.nav {
  position: fixed;
  width: 100%;
  z-index: 99;
  top: 0;
}

.nav >>> .van-nav-bar__right div {
  display: flex;
  align-items: center;
}

.nav >>> .van-nav-bar__right {
  right: 0;
}

.nav >>> .van-nav-bar__left {
  left: 0.24rem;
}

.iconfont {
  font-size: 0.58rem;
}

.btn {
  width: 1.17rem;
  height: 100%;
  display: flex;
  justify-content: center;
}

/* 搜索框 */
.search-btn:active {
  color: #cd1111;
}

.nav .van-cell-group {
  height: 1.22667rem;
  display: flex;
  align-items: center;
  width: 5.33rem;
}

.nav >>> .van-cell {
  padding: 0;
}

.van-nav-bar >>> input {
  background-color: #f3f3f3;
  padding: 0 0.21rem;
  height: 0.74rem;
  border: 1px solid #ddd;
}

/* 点击按钮后的背景色 */
.touch-color {
  background-color: #f2f2f2;
}

/* 下拉菜单 */
.pull-down {
  width: 100%;
  background-color: #444444;
  color: #ddd;
  font-size: 0.42rem;
  position: absolute;
  top: 1.17rem;
}

.user-card {
  display: flex;
  align-items: center;
  border-bottom: 1px solid #3b3b3b;
  padding: 0.32rem 0 0.24rem;
  box-sizing: border-box;
}

.avatar {
  width: 1.33rem;
  height: 1.33rem;
  margin: 0 0.53rem 0.16rem;
  border: 2px solid #555;
}

.user-card span {
  font-size: 0.48rem;
  position: relative;
}

.user-card .van-icon {
  font-size: 0.42rem;
  position: absolute;
  right: 0.32rem;
}

.link-list li {
  border-bottom: 1px solid #3b3b3b;
  height: 1.17rem;
  line-height: 1.17rem;
  padding-left: 20px;
  position: relative;
}

.link-list .van-icon {
  position: absolute;
  right: 0.32rem;
  line-height: 1.17rem;
}

/* 下拉菜单底部 */
.bottom {
  padding: 0.32rem 0.53rem;
}

.logout-btn {
  width: 100%;
  background: linear-gradient(#e53e49, #d43636);
  line-height: 1.06rem;
  text-align: center;
  border-radius: 0.1rem;
  color: #ddd;
}

/* 退出登陆后的下拉菜单 */
.btn-div {
  display: flex;
  justify-content: space-between;
  padding: 0.53rem;
  border-bottom: 0.02rem solid #3b3b3b;
}

.login-btn {
  width: 3.14rem;
  text-align: center;
  line-height: 1.06rem;
  background: linear-gradient(#fafafa, #f2f2f2);
  color: #444;
  margin-left: 0.42rem;
  border-radius: 0.1rem;
}

/* 动画效果 */
.fade-enter-active {
  transition: all 0.5s ease;
}

.fade-leave-active {
  transition: all 0.5s ease;
}

.fade-enter,
.fade-leave-active {
  top: -6.26rem;
  opacity: 0.8;
}
</style>
