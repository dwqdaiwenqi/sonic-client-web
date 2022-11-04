<script setup>
  import { onMounted, ref } from 'vue';
  import { useRoute, useRouter } from 'vue-router';
  import { ElMessage } from 'element-plus';
  import Pageable from '../components/Pageable.vue';
  import axios from '../http/axios';

  const route = useRoute();
  const router = useRouter();
  const pageData = ref({});
  const pageSize = ref(15);
  const currentPage = ref(0);
  const getResultList = (pageNum, pSize) => {
    axios
      .get('/controller/results/list', {
        params: {
          projectId: route.params.projectId,
          page: pageNum || 1,
          pageSize: pSize || pageSize.value,
        },
      })
      .then((resp) => {
        pageData.value = resp.data;
      });
  };
  const delResult = (id) => {
    axios
      .delete('/controller/results', {
        params: {
          id,
        },
      })
      .then((resp) => {
        if (resp.code === 2000) {
          ElMessage.success({
            message: resp.message,
          });
          getResultList();
        }
      });
  };
  const forceStopSuite = (id) => {
    axios
      .get('/controller/testSuites/forceStopSuite', {
        params: {
          resultId: id,
        },
      })
      .then((resp) => {
        if (resp.code === 2000) {
          ElMessage.success({
            message: resp.message,
          });
          getResultList();
        }
      });
  };
  onMounted(() => {
    getResultList();
  });
</script>

<template>
  <el-table :data="pageData['content']" border>
    <el-table-column
      width="80"
      label="结果Id"
      prop="id"
      align="center"
      show-overflow-tooltip
    />
    <el-table-column
      label="测试套件名称"
      min-width="280"
      prop="suiteName"
      header-align="center"
      show-overflow-tooltip
    >
    </el-table-column>
    <el-table-column label="执行用户" width="120" align="center">
      <template #default="scope">
        <el-tag v-if="scope.row.strike === 'SYSTEM'" size="small"
          >定时任务
        </el-tag>
        <span v-else>{{ scope.row.strike }}</span>
      </template>
    </el-table-column>
    <el-table-column label="运行状态" width="120" align="center">
      <template #default="scope">
        <el-tag v-if="scope.row.status === 1" type="success" size="small"
          >测试通过
        </el-tag>
        <el-tag v-if="scope.row.status === 0" type="info" size="small"
          ><i class="el-icon-loading"></i> 运行中
        </el-tag>
        <el-tag v-if="scope.row.status === 3" type="danger" size="small"
          >测试失败
        </el-tag>
        <el-tag v-if="scope.row.status === 2" type="warning" size="small"
          >测试告警
        </el-tag>
      </template>
    </el-table-column>
    <el-table-column
      label="创建时间"
      width="200"
      prop="createTime"
      align="center"
    >
    </el-table-column>
    <el-table-column fixed="right" label="测试报告" width="160" align="center">
      <template #default="scope">
        <div style="text-align: center">
          <el-button
            size="mini"
            icon="el-icon-tickets"
            @click="router.push('ResultDetail/' + scope.row.id)"
            >查看报告
          </el-button>
        </div>
      </template>
    </el-table-column>
    <el-table-column fixed="right" label="操作" width="220" align="center">
      <template #default="scope">
        <el-popconfirm
          style="margin-left: 10px"
          :confirm-button-text="$t('form.confirm')"
          :cancel-button-text="$t('form.cancel')"
          icon="el-icon-warning"
          icon-color="red"
          title="确定中断本次测试吗？"
          @confirm="forceStopSuite(scope.row.id)"
        >
          <template #reference>
            <el-button
              :disabled="scope.row.status !== 0"
              type="warning"
              size="mini"
              icon="el-icon-video-pause"
              >中断
            </el-button>
          </template>
        </el-popconfirm>
        <el-popconfirm
          style="margin-left: 10px"
          :confirm-button-text="$t('form.confirm')"
          :cancel-button-text="$t('form.cancel')"
          icon="el-icon-warning"
          icon-color="red"
          title="确定删除该测试报告吗？"
          @confirm="delResult(scope.row.id)"
        >
          <template #reference>
            <el-button type="danger" size="mini" icon="el-icon-delete"
              >删除
            </el-button>
          </template>
        </el-popconfirm>
      </template>
    </el-table-column>
  </el-table>
  <pageable
    :is-page-set="true"
    :total="pageData['totalElements']"
    :current-page="pageData['number'] + 1"
    :page-size="pageData['size']"
    @change="getResultList"
  ></pageable>
</template>
