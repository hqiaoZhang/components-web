/*
 * @Author: zhanghongqiao 
 * @Date: 2021-06-04 15:28:51 
 * @Email: 991034150@qq.com 
 * @Description: 入口页
 * @Last Modified by: zhanghongqiao
 * @Last Modified time: 2021-06-04 16:28:13
 */
 <template>
   <div>
     <div class="screen">
       <div class="logo">ChartFun</div>
       <div class="desc">一站式数据大屏制作平台</div>
       <transition name="slide-fade">
         <div class="login-box" v-if="show">
           <div class="radio-group">
             <div class="radio-btn" :class="{active: tab == 'login'}" @click="tab = 'login'">登录</div>
             <div class="radio-btn" :class="{active: tab == 'reg'}" @click="tab = 'reg'">注册</div>
           </div>
           <el-input  placeholder="请输入用户名" v-model="form.user"></el-input>

           <el-input  placeholder="请输入密码"
            :type="tab == 'login' ? 'password' : 'text'"
            v-model="form.password"
            style="margin-top: 10px;"></el-input>
            
         </div>
       </transition>
       <div class="btn-wrapper">
         <span class="btn" @click="handleClick">进入系统</span>
       </div>
     </div>
   </div>
 </template>
 

<script>
/* eslint-disable */
import md5 from 'js-md5';

export default {
  data() {
    return {
      show: false,
      tab: 'login',
      form: {
        user: '',
        password: '',
      },
    };
  },
  mounted() {
  },
  methods: {
    handleClick() {
      if (!this.show) {
        if (sessionStorage.getItem('uid')) {
          this.$router.push('home');
        } else {
          this.show = true;
        }
      } else {
        // 开始检查
        const md5pass = md5(this.form.password); // 对密码进行MD5加密
        this.$http
          .post(`/user/${this.tab}`, {
            user: this.form.user,
            pass: md5pass
          })
          .then((res) => {
            const { errno, errmsg, data } = res.data;
            if (errno === 0) {
              this.$message({
                type: 'success',
                message: '验证成功'
              });
              sessionStorage.setItem('uid', data.uid);
              sessionStorage.setItem('user', data.name);
              this.$router.push('home');
            } else {
              this.$message.error(errmsg);
            }
          })
          .catch(() => {});
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.screen {
  position: absolute;
  width: 100vw;
  height: 100vh;
  background: url('../../assets/images/bg.png');
  background-position: 50%;
  background-size: cover;
  background-repeat: no-repeat;
}
.logo {
  height: 72px;
  line-height: 72px;
  text-align: center;
  font-weight: bold;
  font-size: 36px;
  color: #515151;
  margin-top: 180px;

  &::after {
    content: ".";
    font-size: 42px;
    color: rgba(255, 0, 0, 0.6);
  }
}
.desc {
  text-align: center;
  font-size: 16px;
  font-weight: 300;
  font-family: 'Noto Sans SC', sans-serif;
  letter-spacing: 4px;
  color: #808080;
  margin: 0 0 40px;
}

.login-box {
  background: #ffffff;
  width: 240px;
  margin: 20px auto 40px;
  padding: 0 40px 36px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.06);
  .radio-group {
    text-align: center;
    .radio-btn {
      display: inline-block;
      padding: 10px 20px;
      margin: 10px 10px;
      color: #999999;
      cursor: pointer;
      &.active {
        color: #212121;
        border-bottom: 2px solid #212121;
      }
    }
  }
}

.btn-wrapper {
  margin-top: 40px;
  text-align: center;
  a {
    text-decoration: none;
    color: #ffffff;
  }
  .btn {
    padding: 12px 32px;
    background: rgba(0, 0, 0, 0.5);
    color: #ffffff;
    transition: background 0.3s ease;
    font-size: 14px;
    border-radius: 50px;
    &:hover {
      background: #409EFF;
      cursor: pointer;
    }
  }
}

.slide-fade-enter-active {
  transition: all .3s ease;
}
.slide-fade-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-fade-enter, .slide-fade-leave-to
/* .slide-fade-leave-active for below version 2.1.8 */ {
  transform: translateX(10px);
  opacity: 0;
}
</style>

 
