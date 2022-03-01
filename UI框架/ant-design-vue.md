## 后台管理表格基础模板

```vue
<template>
  <div>
    <a-space>
      <a-button type="primary" @click="modalShow({ formObj: createRoleModal })">
        <a-icon type="plus"></a-icon>新增权限角色
      </a-button>
      <a-input placeholder="请输入手机号或姓名进行搜索" class="search" allowClear v-model="searchRoleKeywords">
        <a-icon slot="suffix" type="search" />
      </a-input>
    </a-space>
    <a-divider></a-divider>
    <a-table
      :loading="tableLoading"
      :columns="roleTableColumns"
      :data-source="roleTableData"
      :pagination="pagination"
      rowKey="no"
      @change="handleTableChange"
      @showSizeChange="tableSizeChange"
    >
      <span slot="user_list" slot-scope="text, record">{{ record.user_list.length }}</span>
      <span slot="permission_list" slot-scope="text, record">{{ record.permission_list.length }}</span>
      <span slot="action" slot-scope="text, record">
        <a-tooltip placement="top">
          <template slot="title">
            <span>查看权限角色详情</span>
          </template>
          <a-icon
            type="eye"
            theme="twoTone"
            two-tone-color="#52c41a"
            style="margin-left: 10px;"
            @click="modalShow({ formObj: viewRoleModal, paramObj: record })"
          ></a-icon>
        </a-tooltip>
        <a-tooltip placement="top">
          <template slot="title">
            <span>编辑权限角色</span>
          </template>
          <a-icon
            type="edit"
            theme="twoTone"
            style="margin-left: 10px"
            @click="modalShow({ formObj: editRoleModal, paramObj: record })"
          ></a-icon>
        </a-tooltip>
        <a-tooltip placement="top">
          <template slot="title">
            <span>删除权限角色</span>
          </template>
          <a-popconfirm title="你确定要删除该权限角色吗？" @confirm="delRole(record.id)">
            <a-icon
              type="delete"
              theme="twoTone"
              two-tone-color="#eb2f96"
              style="margin-left: 10px; cursor: pointer"
            ></a-icon>
          </a-popconfirm>
        </a-tooltip>
      </span>
    </a-table>
  </div>
</template>
<script>
const roleTableColumns = [
  { title: "序号", dataIndex: "no" },
  { title: "姓名", dataIndex: "name" },
  { title: "最后操作人", dataIndex: "last_update" },
  { title: "最后修改时间", dataIndex: "updated_at" },
  {
    title: "权限人数",
    dataIndex: "user_list",
    scopedSlots: { customRender: "user_list" }
  },
  {
    title: "权限数量",
    dataIndex: "permission_list",
    scopedSlots: { customRender: "permission_list" }
  },
  {
    title: "操作",
    dataIndex: "action",
    scopedSlots: { customRender: "action" }
  }
];
export default {
  name: "PrivilegesRoleManage",
  data() {
    return {
      pagination: {
        pageNum: 1,
        pageSize: 10,
        showSizeChanger: true,
        showQuickJumper: true,
        pageSizeOptions: ["10", "20", "50", "100"],
        showTotal: total => `共 ${total} 条`,
        total: 0
      },
      tableLoading: false,
      roleTableColumns,
      roleTableData: [],
      searchRoleKeywords: null,
      createRoleModal: {
        visible: false,
        rules: {},
        form: {}
      },
      editRoleModal: {
        visible: false,
        rules: {},
        form: {}
      },
      viewRoleModal: {
        visible: false,
        form: {}
      }
    };
  },
  methods: {
    handleTableChange(pagination) {
      this.pagination.pageSize = pagination.pageSize;
      this.pagination.pageNum = pagination.current;
      this.getTableData();
    },
    tableSizeChange(pagination) {
      this.pagination.pageSize = pagination.pageSize;
      this.pagination.pageNum = pagination.pageNum;
      this.getTableData();
    }
  }
};
</script>
<style lang="less" scoped>
</style>
```

