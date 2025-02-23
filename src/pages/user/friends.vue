<template>
  <bd-page class="flex-col">
    <!-- 布局 -->
    <div class="flex-1 el-card border-none flex-col box-border overflow-hidden">
      <div class="h-50px pl-12px pr-12px box-border flex items-center justify-between bd-title">
        <div class="bd-title-left">
          <p class="m-0 font-600">
            <el-text type="primary">{{ $route.query.name }}</el-text>
            的好友
          </p>
        </div>
        <div class="flex items-center h-50px">
          <el-form inline>
            <el-form-item class="mb-0 !mr-16px">
              <el-input v-model="queryFrom.keyword" placeholder="uid/手机号/用户名" clearable />
            </el-form-item>
            <el-form-item class="mb-0 !mr-16px">
              <el-select v-model="queryFrom.sort_type" placeholder="请选择">
                <el-option label="添加时间排序" :value="0" />
                <el-option label="按消息顺序" :value="1" />
              </el-select>
            </el-form-item>
            <el-form-item class="mb-0 !mr-0">
              <el-button type="primary" @click="getUserList">查询</el-button>
            </el-form-item>
          </el-form>
        </div>
      </div>
      <div class="flex-1 overflow-hidden p-12px">
        <el-table v-loading="loadTable" :data="tableData" :border="true" style="width: 100%; height: 100%">
          <el-table-column type="index" :width="42" :align="'center'" :fixed="'left'">
            <template #header>
              <i-bd-setting class="cursor-pointer" size="16" />
            </template>
          </el-table-column>
          <el-table-column v-for="item in column" v-bind="item" :key="item.prop">
            <template #default="scope">
              <template v-if="item.render">
                <component :is="item.render" :row="scope.row"> </component>
              </template>
              <template v-else-if="item.formatter">
                <slot :name="item.prop" :row="scope.row">{{ item.formatter(scope.row) }}</slot>
              </template>
              <template v-else-if="item.prop">
                <slot :name="item.prop" :row="scope.row">{{ scope.row[item.prop!] }}</slot>
              </template>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div class="bd-card-footer pl-12px pr-12px mb-12px flex items-center justify-between">
        <div></div>
        <el-pagination
          v-model:current-page="queryFrom.page_index"
          v-model:page-size="queryFrom.page_size"
          :page-sizes="[15, 20, 30, 50, 100]"
          :background="true"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
          @size-change="onSizeChange"
          @current-change="onCurrentChange"
        />
      </div>
    </div>
  </bd-page>
</template>

<route lang="yaml">
meta:
  title: 好友列表
  isAffix: false
  isKeepAlive: true
</route>

<script lang="tsx" name="UserFriends" setup>
import { useRoute, useRouter } from 'vue-router';
import { ElButton, ElSpace, ElAvatar } from 'element-plus';
import { BU_DOU_CONFIG } from '@/config';
// API 接口
import { userFriendsGet } from '@/api/user';

const route = useRoute();
const router = useRouter();
/**
 * 表格
 */
const column = reactive<Column.ColumnOptions[]>([
  {
    prop: 'name',
    label: '好友昵称',
    width: 140
  },
  {
    prop: 'avatar',
    label: '好友头像',
    align: 'center',
    width: 90,
    render: (scope: any) => {
      let img_url = '';
      if (scope.row['uid']) {
        img_url = `${BU_DOU_CONFIG.APP_URL}users/${scope.row['uid']}/avatar`;
      }
      return (
        <ElAvatar src={img_url} size={54}>
          {scope.row['name']}
        </ElAvatar>
      );
    }
  },
  {
    prop: 'uid',
    label: '好友ID'
  },
  {
    prop: 'relationship_time',
    label: '成为好友时间'
  },
  {
    prop: 'remark',
    label: '备注'
  },
  {
    prop: 'operation',
    label: '操作',
    align: 'center',
    fixed: 'right',
    width: 124,
    render: (scope: any) => {
      return (
        <ElSpace>
          <ElButton type="primary" onClick={() => onRecordpersonal(scope.row)}>
            聊天记录
          </ElButton>
        </ElSpace>
      );
    }
  }
]);
const tableData = ref<any[]>([]);
const loadTable = ref<boolean>(false);
// 分页
const total = ref(0);

// 查询
const queryFrom = reactive({
  keyword: '',
  sort_type: 0,
  uid: route.query.uid,
  page_size: 15,
  page_index: 1
});

const getUserList = () => {
  loadTable.value = true;
  userFriendsGet(queryFrom).then((res: any) => {
    loadTable.value = false;
    tableData.value = res;
    total.value = res.length;
  });
};

// 分页page-size
const onSizeChange = (size: number) => {
  queryFrom.page_size = size;
  getUserList();
};

// 分页page-size
const onCurrentChange = (current: number) => {
  queryFrom.page_index = current;
  getUserList();
};
// 聊天记录
const onRecordpersonal = (item: any) => {
  router.push({
    path: '/message/recordpersonal',
    query: {
      name: route.query.name,
      uid: route.query.uid,
      touid: item.uid,
      toname: item.name
    }
  });
};

// 初始化
onMounted(() => {
  getUserList();
});
</script>

<style lang="scss" scoped>
// 样式
.bd-title {
  border-bottom: 1px solid var(--el-card-border-color);
}
</style>
