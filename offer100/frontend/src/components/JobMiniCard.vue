<template>
  <el-card shadow="hover" class="job-card">
    <router-link :to="`/jobs/${job.id}`" class="link">
      <h3>{{ job.title }}</h3>
      <p class="meta">公司：{{ job.company }} | 地点：{{ job.city }}</p>
      <p class="meta">求职类型：{{ job.employmentType || '不限' }} | 公司规模：{{ job.companySize || '不限' }}</p>
      <p class="meta">需求经验：{{ job.experienceRequirement || '不限' }} | 学历：{{ job.educationRequirement || '不限' }}</p>
      <div class="tags" v-if="job.categoryL1 || job.categoryL2">
        <el-tag v-if="job.categoryL1" size="small" effect="plain">{{ job.categoryL1 }}</el-tag>
        <el-tag v-if="job.categoryL2" size="small" effect="plain">{{ job.categoryL2 }}</el-tag>
      </div>
      <p class="desc">简介：{{ shortDescription }}</p>
      <p class="salary" v-if="showDescription">薪资：{{ job.salaryRange || '-' }}</p>
    </router-link>
  </el-card>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  job: {
    type: Object,
    required: true
  },
  showDescription: {
    type: Boolean,
    default: false
  }
});

const shortDescription = computed(() => {
  const text = String(props.job?.description || '').trim();
  if (!text) {
    return '暂无岗位简介';
  }
  return text.length > 20 ? `${text.slice(0, 20)}...` : text;
});
</script>

<style scoped>
.job-card {
  border-radius: 12px;
}

.link {
  display: block;
  text-decoration: none;
  color: #1f3f75;
  padding: 14px;
}

h3 {
  margin: 0 0 8px;
}

.meta,
.salary,
.desc {
  margin: 4px 0;
  font-size: 14px;
}

.tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: 8px;
}


</style>
