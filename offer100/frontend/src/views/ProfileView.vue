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
      <h2>个人/公司信息</h2>
      <form class="job-form" @submit.prevent="saveProfile">
        <input v-model.trim="form.nickname" placeholder="昵称" />
        <label class="file-label">
          头像（本地选择）
          <input type="file" accept="image/*" @change="onAvatarFileChange" />
        </label>
        <img v-if="form.avatarUrl" :src="form.avatarUrl" alt="avatar" class="avatar-preview" />
        <textarea v-model.trim="form.commonPhrase" placeholder="常用语（用于投递/邀请自动发送）" />

        <template v-if="authStore.activeIdentity === 'jobseeker'">
          <input v-model.trim="form.fullName" placeholder="姓名" />
          <input v-model.number="form.age" placeholder="年龄" />
          <label>
            性别
            <div class="radio-row">
              <label><input v-model="form.gender" type="radio" value="男" /> 男</label>
              <label><input v-model="form.gender" type="radio" value="女" /> 女</label>
            </div>
          </label>
          <label>
            求职状态
            <select v-model="form.jobHuntingStatus">
              <option v-for="item in jobHuntingStatusOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <label>
            求职类型
            <select v-model="form.expectedJobType">
              <option v-for="item in jobTypeOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <label>
            期望薪资
            <select v-model="form.expectedSalary">
              <option v-for="item in salaryOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <input v-model.trim="form.school" placeholder="学校" />
          <input v-model.trim="form.major" placeholder="专业" />
          <label>
            学历
            <select v-model="form.degree">
              <option v-for="item in degreeOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <input v-model.trim="form.graduationCohort" placeholder="毕业届别（如：2026届）" />
          <label>
            工作经验
            <select v-model="form.workExperience">
              <option v-for="item in workExperienceOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <label>
            个人所在地
            <select v-model="form.location">
              <option v-for="item in personalLocationOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <input v-model.trim="form.contactPhone" placeholder="联系电话" />
          <input v-model.trim="form.contactEmail" placeholder="联系邮箱" />
          <input v-model.trim="form.strengths" placeholder="个人优势" />
          <input v-model.trim="form.expectedPosition" placeholder="期望岗位" />
          <textarea v-model.trim="form.internshipExperience" placeholder="实习经历" />
          <textarea v-model.trim="form.projectExperience" placeholder="项目经历" />
          <textarea v-model.trim="form.competitionExperience" placeholder="比赛经历" />
          <textarea v-model.trim="form.campusExperience" placeholder="在校经历" />
        </template>

        <template v-else>
          <input v-model.trim="form.companyName" placeholder="公司名称" />
          <label>
            公司位置
            <select v-model="form.companyAddress">
              <option v-for="item in cityOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <label>
            公司规模
            <select v-model="form.companySize">
              <option v-for="item in companySizeOptions" :key="item" :value="item">{{ item }}</option>
            </select>
          </label>
          <textarea v-model.trim="form.companyIntro" placeholder="公司介绍" />
        </template>

        <div class="actions">
          <button type="submit">保存资料</button>
          <button type="button" @click="exportWord">导出 Word</button>
        </div>
      </form>
      <p v-if="tip" class="tip">{{ tip }}</p>
    </section>
  </main>
</template>

<script setup>
import { onMounted, reactive, ref } from 'vue';
import { useRouter } from 'vue-router';
import http from '../api/http';
import TopBar from '../components/TopBar.vue';
import { useAuthStore } from '../stores/auth';

const router = useRouter();
const authStore = useAuthStore();
const tip = ref('');
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

const form = reactive({
  nickname: '',
  avatarUrl: '',
  commonPhrase: '',
  companyName: '',
  companyAddress: '上海',
  companySize: '不限',
  companyIntro: '',
  fullName: '',
  age: null,
  gender: '男',
  expectedSalary: '',
  school: '',
  major: '',
  degree: '',
  graduationCohort: '',
  workExperience: '',
  location: '其他',
  contactPhone: '',
  contactEmail: '',
  expectedJobType: '不限',
  strengths: '',
  jobHuntingStatus: '考虑机会',
  expectedPosition: '',
  internshipExperience: '',
  projectExperience: '',
  competitionExperience: '',
  campusExperience: ''
});

function fileToDataUrl(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result);
    reader.onerror = () => reject(new Error('头像读取失败'));
    reader.readAsDataURL(file);
  });
}

async function onAvatarFileChange(event) {
  const file = event.target.files?.[0];
  if (!file) {
    return;
  }
  try {
    form.avatarUrl = await fileToDataUrl(file);
    tip.value = '头像已选择，点击保存资料后生效';
  } catch (error) {
    tip.value = '头像读取失败，请重试';
  }
}

async function loadProfile() {
  const { data } = await http.get('/profile/me');
  form.nickname = data.user.nickname || '';
  Object.assign(form, {
    avatarUrl: data.profile?.avatar_url || '',
    commonPhrase: data.profile?.common_phrase || '',
    companyName: data.profile?.company_name || '',
    companyAddress: data.profile?.company_address || '',
    companySize: data.profile?.company_size || '',
    companyIntro: data.profile?.company_intro || '',
    fullName: data.profile?.full_name || '',
    age: data.profile?.age || null,
    gender: data.profile?.gender || '男',
    expectedSalary: data.profile?.expected_salary || '',
    school: data.profile?.school || '',
    major: data.profile?.major || '',
    degree: data.profile?.degree || '',
    graduationCohort: data.profile?.graduation_cohort || '',
    workExperience: data.profile?.work_experience || '',
    location: data.profile?.location || '其他',
    contactPhone: data.profile?.contact_phone || '',
    contactEmail: data.profile?.contact_email || '',
    expectedJobType: data.profile?.expected_job_type || '不限',
    strengths: data.profile?.strengths || '',
    jobHuntingStatus: data.profile?.job_hunting_status || '考虑机会',
    expectedPosition: data.profile?.expected_position || '',
    internshipExperience: data.profile?.internship_experience || '',
    projectExperience: data.profile?.project_experience || '',
    competitionExperience: data.profile?.competition_experience || '',
    campusExperience: data.profile?.campus_experience || ''
  });
}

async function saveProfile() {
  try {
    await http.put('/profile/me', form);
    tip.value = '资料已保存';
  } catch (error) {
    tip.value = error.response?.data?.message || '保存失败';
  }
}

async function exportWord() {
  try {
    const response = await http.get('/profile/export-word', { responseType: 'blob' });
    const blob = new Blob([response.data], { type: 'application/msword' });
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `offer100-profile-${authStore.activeIdentity}.doc`;
    a.click();
    window.URL.revokeObjectURL(url);
  } catch (error) {
    tip.value = error.response?.data?.message || '导出失败';
  }
}

function switchIdentity(identity) {
  authStore.setActiveIdentity(identity);
  loadProfile();
}

function logout() {
  authStore.logout();
  router.push('/login');
}

onMounted(loadProfile);
</script>

<style scoped>
.actions {
  display: flex;
  gap: 10px;
  margin-top: 8px;
}

.file-label {
  display: grid;
  gap: 6px;
  color: #243b63;
  font-size: 14px;
}

.avatar-preview {
  width: 72px;
  height: 72px;
  border-radius: 50%;
  object-fit: cover;
  border: 1px solid #cfe0ff;
}

.radio-row {
  display: flex;
  gap: 14px;
  margin-top: 6px;
}

.tip {
  margin-top: 12px;
  color: #1d4ed8;
}
</style>
