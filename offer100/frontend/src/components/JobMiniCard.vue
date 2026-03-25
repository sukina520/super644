<template>
  <el-card shadow="hover" class="job-card">
    <router-link :to="`/jobs/${job.id}`" class="link">
      <h3>{{ job.title }}</h3>
      <p class="meta">{{ job.company }} | {{ job.city }}</p>
      <p class="salary">{{ job.salaryRange }}</p>
      <p class="meta">学历：{{ job.educationRequirement || '无限制' }}</p>
      <div class="tags" v-if="job.categoryL1 || job.categoryL2">
        <el-tag v-if="job.categoryL1" size="small" effect="plain">{{ job.categoryL1 }}</el-tag>
        <el-tag v-if="job.categoryL2" size="small" effect="plain">{{ job.categoryL2 }}</el-tag>
      </div>
      <p class="desc" v-if="showDescription">{{ job.description || '暂无岗位描述' }}</p>
    </router-link>
  </el-card>
</template>

<script setup>
defineProps({
  job: {
    type: Object,
    required: true
  },
  showDescription: {
    type: Boolean,
    default: false
  }
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
