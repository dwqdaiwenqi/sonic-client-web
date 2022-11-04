<script setup>
import { ref } from 'vue';
import { useRoute } from 'vue-router';
import TestCaseList from '../components/TestCaseList.vue';

const route = useRoute();
const caseList = ref(null);
const projectId = ref(parseInt(route.params.projectId));
const platform = 1;
</script>

<template>
  <el-button size="mini" round type="primary" @click="caseList.open()"
    >添加用例</el-button
  >
  <test-case-list
    ref="caseList"
    :project-id="projectId"
    :platform="platform"
    :is-read-only="false"
  />
</template>
