<template>
  <el-card class="panel" shadow="never">
    <h2>我的发布岗位</h2>
    <el-alert title="仅显示当前招聘者发布的岗位，可直接删除" type="info" :closable="false" show-icon />

    <el-table :data="jobs" style="width: 100%; margin-top: 12px">
      <el-table-column prop="title" label="岗位名称" min-width="180" />
      <el-table-column prop="city" label="城市" width="100" />
      <el-table-column prop="salaryRange" label="薪资" width="120" />
      <el-table-column prop="employmentType" label="类型" width="100" />
      <el-table-column prop="publishAt" label="发布时间" width="120" />
      <el-table-column label="操作" width="200">
        <template #default="scope">
          <el-button link type="primary" @click="goDetail(scope.row.id)">查看</el-button>
          <el-button link type="danger" @click="removeJob(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-empty v-if="!jobs.length" description="暂无已发布岗位" :image-size="80" />
    <el-alert v-if="tip" :title="tip" type="success" :closable="false" show-icon class="tip" />
  </el-card>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import { useRouter } from 'vue-router';
import http from '../../api/http';

const router = useRouter();
const jobs = ref([]);
const tip = ref('');

async function loadJobs() {
  try {
    const { data } = await http.get('/jobs/mine');
    jobs.value = Array.isArray(data) ? data : [];
  } catch (error) {
    tip.value = error.response?.data?.message || '加载岗位失败';
  }
}

function goDetail(jobId) {
  router.push(`/jobs/${jobId}`);
}

async function removeJob(job) {
  const ok = window.confirm(`确认删除岗位：${job.title}？`);
  if (!ok) {
    return;
  }

  try {
    await http.delete(`/jobs/${job.id}`);
    tip.value = '岗位删除成功';
    await loadJobs();
  } catch (error) {
    tip.value = error.response?.data?.message || '岗位删除失败';
  }
}

onMounted(loadJobs);
</script>

<style scoped>
.tip {
  margin-top: 12px;
}
</style>
