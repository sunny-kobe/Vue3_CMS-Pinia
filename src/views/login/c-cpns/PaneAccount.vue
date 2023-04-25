<template>
  <div class="PaneAccount">
    <el-form label-width="60px" style="max-width: 460px" size="large" :model="account" :rules="rules" status-icon
      ref="formRef">
      <el-form-item label="账号" prop="name">
        <el-input v-model="account.name" />
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input v-model="account.password" show-password />
      </el-form-item>
    </el-form>
  </div>
</template>

<script lang="ts" setup>
import { reactive, ref } from 'vue'
import type { FormInstance, FormRules } from 'element-plus'
import axios from 'axios';
import { ElMessage } from 'element-plus'
import { localCache } from '@/utils/cache'
import type { IAccount } from '@/types';

const formRef = ref<FormInstance>()

//定义常量
const CACHE_NAME = 'name'
const CACHE_PASSWORD = 'password'

//1. 定义account数据
const account = reactive<IAccount>({
  name: localCache.getCache(CACHE_NAME) ?? '',
  password: localCache.getCache(CACHE_PASSWORD) ?? ''
})

//2. 表单校验
const rules = reactive<FormRules>({
  name: [{ required: true, message: '请输入账号信息~', trigger: 'blur' },
  {
    pattern: /^[a-z0-9]{6,20}$/,
    message: '必须是6~20数字或字母组成~',
    trigger: 'blur'
  }],
  password: [{ required: true, message: '请输入账号密码~', trigger: 'blur' },
  // 密码要求6位，且必须是纯数字组成
  {
    pattern: /^[0-9]{6}$/,
    message: '密码要求6位，且必须是纯数字组成~',
    trigger: 'blur'
  }
  ]
})

//3. 执行账号的登录逻辑
const loginAction = (isRemPwd: boolean) => {
  //1. 表单登录成功
  formRef.value?.validate((valid) => {
    if (valid) {
      const { name, password } = account;
      async function loginAccountAction({ name, password }: { name: string, password: string }) {
        const res = await axios.post('http://123.207.32.32:5000/login', {
          name,
          password
        })
        console.log(res);
      }
      //2. 向服务器发送网络请求（携带账号和密码）
      loginAccountAction({ name, password }).then(() => {
        //3. 判断用户是否需要记住密码
        if (isRemPwd) {
          localCache.setCache(CACHE_NAME, name)
          localCache.setCache(CACHE_PASSWORD, password)
        } else {
          localCache.removeCache(CACHE_NAME)
          localCache.removeCache(CACHE_PASSWORD)
        }
      })
    } else {
      ElMessage.error('账号或者密码输入的规则错误~')
    }
  })
}

defineExpose({
  loginAction,
})
</script>

<style lang="less" scoped></style>
