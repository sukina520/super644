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
          <label>
            求职类型
            <select v-model="jobseekerProfile.expectedJobType">
              <option v-for="item in jobTypeOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <label>
            期望薪资
            <select v-model="jobseekerProfile.expectedSalary">
              <option v-for="item in salaryOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <label>
            学历
            <select v-model="jobseekerProfile.degree">
              <option v-for="item in degreeOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <label>
            工作经验
            <select v-model="jobseekerProfile.workExperience">
              <option v-for="item in workExperienceOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <label>
            个人所在地
            <select v-model="jobseekerProfile.location">
              <option v-for="item in personalLocationOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <input v-model.trim="jobseekerProfile.expectedPosition" placeholder="期望岗位" />
          <input v-model.trim="jobseekerProfile.strengths" placeholder="个人优势" required />
          <textarea v-model.trim="jobseekerProfile.projectExperience" placeholder="项目经历" />
        </template>
        <template v-else>
          <input v-model.trim="recruiterProfile.companyName" placeholder="公司名称" required />
          <label>
            公司位置
            <select v-model="recruiterProfile.companyAddress" required>
              <option v-for="item in cityOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <label>
            公司规模
            <select v-model="recruiterProfile.companySize" required>
              <option v-for="item in companySizeOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <textarea v-model.trim="recruiterProfile.companyIntro" placeholder="公司介绍" />
        </template>
        <button type="submit" :disabled="isCurrentIdentityRegistered">注册该身份</button>
      </form>

      <p v-if="tip" class="tip">{{ tip }}</p>
    </section>
  </main>
</template>

<script setup>
import { computed, reactive, ref, watch } from 'vue';
import { useRouter } from 'vue-router';
import http from '../api/http';
import TopBar from '../components/TopBar.vue';
import { useAuthStore } from '../stores/auth';

const router = useRouter();
const authStore = useAuthStore();
const jobHuntingStatusOptions = ['随时到岗', '月内到岗', '考虑机会', '暂不考虑'];
const jobTypeOptions = ['不限', '全职', '实习', '兼职'];
const salaryOptions = ['不限', '3k以下', '3-5k', '5-10k', '10-20k', '20k以上'];
const degreeOptions = ['不限', '高中', '大专', '本科', '硕士', '博士'];
const workExperienceOptions = ['不限', '在校生', '应届生', '1年以内', '1-3年', '3-5年', '5-10年', '10年以上'];
const personalLocationOptions = [
  '其他', '北京市', '天津市', '上海市', '重庆市', '河北省', '山西省', '辽宁省', '吉林省', '黑龙江省',
  '江苏省', '浙江省', '安徽省', '福建省', '江西省', '山东省', '河南省', '湖北省', '湖南省', '广东省',
  '海南省', '四川省', '贵州省', '云南省', '陕西省', '甘肃省', '青海省', '内蒙古自治区', '广西壮族自治区',
  '西藏自治区', '宁夏回族自治区', '新疆维吾尔自治区', '香港特别行政区', '澳门特别行政区',
  '广州市', '深圳市', '杭州市', '南京市', '苏州市', '成都市', '武汉市', '西安市'
];
const cityOptions = ['其他', '北京', '上海', '广州', '深圳', '杭州', '南京', '苏州', '成都', '重庆', '武汉', '西安'];
const companySizeOptions = ['不限', '0-20人', '20-100人', '100-200人', '200-500人', '500-1000人', '1000人以上'];

function getOppositeIdentity(currentIdentity) {
  return currentIdentity === 'jobseeker' ? 'recruiter' : 'jobseeker';
}

const identity = ref(getOppositeIdentity(authStore.activeIdentity));
const tip = ref('');
const isCurrentIdentityRegistered = computed(() => authStore.identities.includes(identity.value));

const recruiterProfile = reactive({
  companyName: '',
  companyAddress: '上海',
  companySize: '不限',
  companyIntro: ''
});

const jobseekerProfile = reactive({
  fullName: '',
  age: 22,
  gender: '男',
  jobHuntingStatus: '考虑机会',
  expectedJobType: '不限',
  expectedSalary: '不限',
  degree: '不限',
  workExperience: '不限',
  location: '其他',
  expectedPosition: '',
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

watch(
  () => authStore.activeIdentity,
  (nextIdentity) => {
    identity.value = getOppositeIdentity(nextIdentity);
  }
);
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
