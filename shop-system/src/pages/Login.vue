<template>
  <el-card class="box-card">
    <el-card class="box-form">
      <template #header>
        <div class="card-header">
          <h3>“微商城”后台管理系统</h3>
        </div>
      </template>
      <el-form ref="ruleFormRef" status-icon :model="form" :rules="rules" label-width="120px">
        <el-form-item prop="username" label="用户名：">
          <el-input v-model="form.username" placeholder="请输入用户名" />
        </el-form-item>
        <el-form-item prop="password" label="密　码：">
          <el-input v-model="form.password" type="password" show-password placeholder="请输入密码" />
        </el-form-item>
        <el-form-item prop="captcha" label="验证码：">
          <el-input v-model="form.captcha" placeholder="请输入验证码" style="width: 50%;"/>
          <canvas id="captcha" width="160" height="40"  @click="refreshCaptcha"></canvas>
          <el-button @click="refreshCaptcha" style="margin-right: 30px;">刷新</el-button>
        </el-form-item>
        <el-form-item>
          <el-button class="button" @click="submitForm(ruleFormRef)" type="primary" size="large">登录</el-button>
          <el-button class="button" @click="resetForm" type="info" size="large">重置</el-button>
          <a href="./Register.vue" target="_blank" style="margin-left: 30px;">注册</a> 
        </el-form-item>
      </el-form>
    </el-card>
  </el-card>
</template>

<script setup>
import { ref, reactive,onMounted } from 'vue'
import { login } from '../api'
import { useRouter } from 'vue-router'
import useToken from '../stores/token'
import notification from '../utils/notification'

const router = useRouter()
const { updateToken } = useToken()

const form = reactive({
  username: '',
  password: ''
})
// 生成验证码
function generateCaptcha() {
  const canvas = document.getElementById('captcha');
  const ctx = canvas.getContext('2d');
  const captcha = Math.random().toString(36).substring(2, 8).toUpperCase();
  localStorage.setItem('captcha', captcha);
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.font = '30px Arial';
  ctx.fillStyle = '#333';
  ctx.fillText(captcha, 20, 40);
}

// 刷新验证码
function refreshCaptcha() {
  generateCaptcha();
}
//在挂载时加载验证码
onMounted(() => {
  generateCaptcha();
})
const ruleFormRef = ref()
//获得当前时间
function getFormattedCurrentTime() {
  const now = new Date();
  return `${now.getFullYear()}-${(now.getMonth() + 1).toString().padStart(2, '0')}-${now.getDate().toString().padStart(2, '0')} 
   ${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}:${now.getSeconds().toString().padStart(2, '0')}`;
}
//验证码不区分大小写
const captcha = localStorage.getItem('captcha').toUpperCase();



// 表单提交
const submitForm = formEl => {
  formEl.validate(async valid => {
    if (valid ) {
      const data = await login(form)
      if (data) {
        updateToken(data.token)
        router.push({ name: 'index' })
        // 记录登录时间保存到本地存储
        const currentTime = getFormattedCurrentTime();
        console.log(`用户 ${data.username} 登录成功，登录时间为 ${currentTime}`);
        localStorage.setItem('loginTime', currentTime);
        // 显示登录成功通知
        notification({
          message: '登录成功',
          type: 'success'
        })
      }
    } else {
      notification({
        message: '表单填写有误',
        message: '验证码错误',
        type: 'error'
      })
    }
  })
}

// 表单重置
const resetForm = () => {
  ruleFormRef.value.resetFields()
}

const rules = reactive({
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 3, max: 12, message: '用户名长度为3~12个字符', trigger: 'blur' },
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, max: 24, message: '密码长度为6~24个字符', trigger: 'blur' },
   ],
    captcha: [
      { required: true, message: '请输入验证码', trigger: 'blur' },
      { min: 6, max: 6, message: '验证码长度为6个字符', trigger: 'blur' },
    ]
  
})
</script>


<style lang="scss" scoped>
.box-card {
  height: 100%;
  background: rgba(38, 72, 176) url('/images/loginBg.jpg') center / cover no-repeat;

  .box-form {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 70%;
    max-width: 750px;
    transform: translate(-50%, -50%);

    .card-header {
      display: flex;
      justify-content: center;
      align-items: center;
    }

    .el-form {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;

      .el-form-item {
        width: 70%;
        display: flex;
        align-items: center;
        justify-content: center;
        --el-form-label-font-size: 16px;
        margin-top: 15px;
        ;
        margin-bottom: 15px;

        .button {
          width: 90px;
        }

        &.center {
          display: flex;
          justify-content: center;
        }
      }
    }
  }
}

#captcha {
  font-size: 1px;
  background-color: aqua;
  text-align: center;
  border: 1px solid #ccc; 
  margin-left: 10px;
}
</style>
