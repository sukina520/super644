<template>
  <el-card shadow="hover" class="seeker-card">
    <router-link :to="`/seekers/${seeker.userId}`" class="link">
      <div class="head-row">
        <div class="head">
          <el-avatar :size="44" :src="seeker.avatarUrl || ''">{{ (seeker.fullName || seeker.username || 'U').slice(0, 1) }}</el-avatar>
          <div>
            <h3>{{ seeker.fullName || seeker.username }}</h3>
            <p class="meta">{{ seeker.gender || '-' }} | {{ seeker.age || '-' }}岁 | {{ seeker.graduationCohort || '届别未填' }}</p>
          </div>
        </div>
        <el-tag type="success" size="small">{{ seeker.jobHuntingStatus || '考虑机会' }}</el-tag>
      </div>

      <div class="info-grid">
        <p class="line"><span class="label">期望薪资</span>{{ seeker.expectedSalary || '未填写' }}</p>
        <p class="line"><span class="label">学历</span>{{ seeker.degree || seeker.education || '未填写' }}</p>
        <p class="line"><span class="label">学校</span>{{ seeker.school || '未填写' }}</p>
        <p class="line"><span class="label">专业</span>{{ seeker.major || '未填写' }}</p>
        <p class="line"><span class="label">所在地</span>{{ seeker.location || '其他' }}</p>
        <p class="line"><span class="label">工作经验</span>{{ seeker.workExperience || seeker.experience || '未填写' }}</p>
        <p class="line"><span class="label">求职类型</span>{{ seeker.expectedJobType || '不限' }}</p>
        <p class="line"><span class="label">期望岗位</span>{{ seeker.expectedPosition || '未填写' }}</p>
      </div>

      <p class="line full"><span class="label">个人优势</span>{{ shortStrengths }}</p>
      <p class="line full" v-if="showExperience"><span class="label">经历摘要</span>{{ seeker.internshipExperience || seeker.projectExperience || seeker.experience || '未填写' }}</p>
      <slot />
    </router-link>
  </el-card>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  seeker: {
    type: Object,
    required: true
  },
  showExperience: {
    type: Boolean,
    default: false
  }
});

const shortStrengths = computed(() => {
  const text = String(props.seeker?.strengths || '').trim();
  if (!text) {
    return '未填写';
  }
  return text.length > 20 ? `${text.slice(0, 20)}...` : text;
});
</script>

<style scoped>
.seeker-card {
  border-radius: 14px;
  border: 1px solid #dbe8ff;
}

.link {
  display: block;
  text-decoration: none;
  color: #1f3f75;
  padding: 10px;
}

.head-row {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 10px;
}

.head {
  display: flex;
  gap: 10px;
  align-items: center;
}

h3 {
  margin: 0;
  color: #123568;
}

.meta {
  margin: 6px 0 0;
  font-size: 13px;
  color: #5474a3;
}

.info-grid {
  margin-top: 10px;
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 6px 12px;
}

.line {
  margin: 0;
  font-size: 13px;
  color: #304f7f;
}

.line.full {
  margin-top: 8px;
}

.label {
  color: #6b86b1;
  margin-right: 6px;
}

@media (max-width: 768px) {
  .info-grid {
    grid-template-columns: 1fr;
  }
}
</style>
