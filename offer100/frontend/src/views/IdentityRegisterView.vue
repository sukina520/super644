<template>
  <main class="page">
    <TopBar
      :username="authStore.user?.nickname || authStore.user?.username"
      :active-identity="authStore.activeIdentity"
      :identities="authStore.identities"
      @switch-identity="switchIdentity"
      @logout="logout"
    />

    <section class="panel">
      <h2>注册新身份</h2>
      <p>当前已注册：{{ authStore.identities.join(' / ') || '无' }}</p>
      <label>
        选择身份
        <select v-model="identity">
          <option value="jobseeker" :disabled="authStore.identities.includes('jobseeker')">求职者</option>
          <option value="recruiter" :disabled="authStore.identities.includes('recruiter')">招聘者</option>
        </select>
      </label>

      <form class="job-form" @submit.prevent="submit">
        <template v-if="identity === 'jobseeker'">
          <input v-model.trim="jobseekerProfile.fullName" placeholder="姓名" required />
          <input v-model.number="jobseekerProfile.age" placeholder="年龄" required />
          <label>
            性别
            <div class="radio-row">
              <label><input v-model="jobseekerProfile.gender" type="radio" value="男" required /> 男</label>
              <label><input v-model="jobseekerProfile.gender" type="radio" value="女" required /> 女</label>
            </div>
          </label>
          <label>
            求职状态
            <select v-model="jobseekerProfile.jobHuntingStatus" required>
              <option v-for="item in jobHuntingStatusOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <input v-model.trim="jobseekerProfile.strengths" placeholder="个人优势" required />
          <textarea v-model.trim="jobseekerProfile.projectExperience" placeholder="项目经历" />
        </template>
        <template v-else>
          <input v-model.trim="recruiterProfile.companyName" placeholder="公司名称" required />
          <input v-model.trim="recruiterProfile.companyAddress" placeholder="公司地址" required />
          <input v-model.trim="recruiterProfile.companySize" placeholder="公司规模" required />
          <textarea v-model.trim="recruiterProfile.companyIntro" placeholder="公司介绍" />
        </template>
        <button type="submit" :disabled="isCurrentIdentityRegistered">注册该身份</button>
      </form>

      <p v-if="tip" class="tip">{{ tip }}</p>
    </section>
  </main>
</template>

<script setup>
import { computed, reactive, ref } from 'vue';
import { useRouter } from 'vue-router';
import http from '../api/http';
import TopBar from '../components/TopBar.vue';
import { useAuthStore } from '../stores/auth';

const router = useRouter();
const authStore = useAuthStore();
const jobHuntingStatusOptions = ['随时到岗', '月内到岗', '考虑机会', '暂不考虑'];

const identity = ref('jobseeker');
const tip = ref('');
const isCurrentIdentityRegistered = computed(() => authStore.identities.includes(identity.value));

const recruiterProfile = reactive({
  companyName: '',
  companyAddress: '',
  companySize: '',
  companyIntro: ''
});

const jobseekerProfile = reactive({
  fullName: '',
  age: 22,
  gender: '男',
  jobHuntingStatus: '考虑机会',
  strengths: '',
  projectExperience: ''
});

async function submit() {
  if (isCurrentIdentityRegistered.value) {
    tip.value = '该身份已注册，无需重复提交';
    return;
  }
  try {
    const { data } = await http.post('/profile/register-identity', {
      identity: identity.value,
      recruiterProfile,
      jobseekerProfile
    });
    authStore.setUserIdentities(data.identities);
    tip.value = '身份注册成功，可以在顶部切换';
  } catch (error) {
    tip.value = error.response?.data?.message || '身份注册失败';
  }
}

function switchIdentity(nextIdentity) {
  authStore.setActiveIdentity(nextIdentity);
}

function logout() {
  authStore.logout();
  router.push('/login');
}
</script>

<style scoped>
.tip {
  margin-top: 12px;
  color: #1d4ed8;
}

.radio-row {
  display: flex;
  gap: 14px;
  margin-top: 6px;
}
</style>
