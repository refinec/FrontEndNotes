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

## 在上面基础上一个总的简单模板

```vue
<template>
  <div>
    <a-button type="primary" @click="modalShow({ modalObj: createQaModal, type: 'add' })">
      <a-icon type="plus"></a-icon>新增答疑视频
    </a-button>
    <a-divider></a-divider>
    <a-table
      :loading="tableLoading"
      :columns="qaTableColumns"
      :data-source="qaTableData"
      :pagination="pagination"
      rowKey="id"
      @change="handleTableChange"
      @showSizeChange="tableSizeChange"
    >
      <span slot="tags" slot-scope="text, record">
        <a-tag color="blue" v-for="(tag, i) in record.tags" :key="i">{{ tag }}</a-tag>
      </span>
      <span slot="action" slot-scope="text, record">
        <a-tooltip placement="top">
          <template slot="title">
            <span>查看答疑视频详情</span>
          </template>
          <a-icon
            type="eye"
            theme="twoTone"
            two-tone-color="#52c41a"
            style="margin-left: 10px;"
            @click="modalShow({ modalObj: viewQaModal, paramObj: record })"
          ></a-icon>
        </a-tooltip>
        <a-tooltip placement="top">
          <template slot="title">
            <span>编辑答疑视频</span>
          </template>
          <a-icon
            type="edit"
            theme="twoTone"
            style="margin-left: 10px"
            @click="modalShow({ modalObj: createQaModal, paramObj: record, type: 'edit' })"
          ></a-icon>
        </a-tooltip>
        <a-tooltip placement="top">
          <template slot="title">
            <span>删除答疑视频</span>
          </template>
          <a-popconfirm title="你确定要删除该答疑视频吗？" @confirm="delQa(record.id)">
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
    <!-- 新增/编辑 -->
    <a-modal
      :maskClosable="false"
      width="50vw"
      :title="`${type}答疑视频`"
      :visible="createQaModal.visible"
      @ok="formSubmit('createQaForm')"
      @cancel="
        modalHidden({
          modalObj: createQaModal,
          refName: 'createQaForm',
          needReset: true
        })
      "
    >
      <a-form-model
        ref="createQaForm"
        :model="createQaModal.form"
        :rules="createQaModal.rules"
        :label-col="{ span: 4 }"
        :wrapper-col="{ span: 20 }"
      >
        <a-form-model-item label="视频名称" prop="name">
          <a-input placeholder="请填写视频名称" v-model="createQaModal.form.name"></a-input>
        </a-form-model-item>
        <a-form-model-item label="视频地址" prop="video_url">
          <a-input placeholder="请填写视频地址" v-model="createQaModal.form.video_url"></a-input>
        </a-form-model-item>
        <a-form-model-item label="视频标签" prop="tags">
          <a-select
            mode="tags"
            style="width: 100%"
            placeholder="请填写视频标签"
            v-model="createQaModal.form.tags"
          ></a-select>
        </a-form-model-item>
        <a-form-model-item label="备注" prop="remark">
          <a-textarea
            placeholder="请填写备注"
            v-model="createQaModal.form.remark"
            :auto-size="{ minRows: 2, maxRows: 6 }"
          />
        </a-form-model-item>
      </a-form-model>
    </a-modal>
    <!-- 详情 -->
    <a-modal
      width="70vw"
      :title="`${viewQaModal.form.name || ''}答疑视频详情`"
      :footer="null"
      :visible="viewQaModal.visible"
      @cancel="viewQaModal.visible = false"
    >
      <a-descriptions bordered>
        <a-descriptions-item
          v-for="(v, k) in viewQaModal.form"
          :key="k"
          :label="descTitle(k)"
          :span="(k === 'tags') ? 3 : 1.5"
        >
          <template v-if="k === 'tags'">
            <a-tag v-for="(tag, i) in v" color="blue" :key="i">{{tag}}</a-tag>
          </template>
          <template v-else>{{v}}</template>
        </a-descriptions-item>
      </a-descriptions>
    </a-modal>
  </div>
</template>
<script>
const qaTableColumns = [
  { title: "ID", dataIndex: "id" },
  { title: "视频名称", dataIndex: "name" },
  {
    title: "视频地址",
    dataIndex: "video_url"
  },
  {
    title: "标签",
    dataIndex: "tags",
    scopedSlots: { customRender: "tags" }
  },
  {
    title: "备注",
    dataIndex: "remark"
  },
  { title: "最后修改时间", dataIndex: "updated_at" },
  { title: "最后操作人", dataIndex: "last_update" },
  {
    title: "操作",
    dataIndex: "action",
    scopedSlots: { customRender: "action" }
  }
];
export default {
  name: "AQVideoManage",
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
      qaTableColumns,
      qaTableData: [],
      type: "添加",
      createQaModal: {
        visible: false,
        rules: {
          name: [
            { required: true, message: "请填写视频名称", trigger: "blur" }
          ],
          video_url: [
            { required: true, message: "请填写视频地址", trigger: "blur" }
          ],
          tags: [{ required: true, message: "请填写视频标签", trigger: "blur" }]
        },
        form: {
          name: null,
          video_url: null,
          tags: [],
          remark: null
        }
      },
      viewQaModal: {
        visible: false,
        form: {}
      }
    };
  },
  created() {
    this.getTableData();
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
    },
    getTableData() {
      this.tableLoading = true;
      this.$axios
        .get(
          `/video/admin/qa/?page_size=${this.pagination.pageSize}&page_num=${this.pagination.pageNum}`
        )
        .then(res => {
          const { video_qa_list, page_count } = res.data;
          this.qaTableData = video_qa_list;
          this.pagination.total = page_count;
        })
        .catch(err => {
          this.$notification.error({
            message: "答疑视频列表获取失败",
            description: err.response.data.detail,
            duration: 3
          });
        })
        .finally(() => {
          this.tableLoading = false;
        });
    },
    formSubmit(type = "createQaForm") {
      this.$refs[type].validate(valid => {
        if (!valid) return;
        let request = null;
        let form = { ...this.createQaModal.form };
        if (this.type === "添加") {
          request = this.$axios.post("/video/admin/qa/", form);
        } else if (this.type === "编辑") {
          request = this.$axios.put("/video/admin/qa/", {
              video_qa_id: form.id,
              name: form.name,
              video_url: form.video_url,
              remark: form.remark,
              tags: form.tags
            });
        }
        request
          .then(() => {
            this.$message.success(this.type + "成功!");
            this.getTableData();
          })
          .catch(err => {
            this.$notification.error({
              message: this.type + "失败",
              description: err.response.data.detail,
              duration: 3
            });
          })
          .finally(() => {
            this.modalHidden({
              modalObj: this.createQaModal,
              refName: "createQaForm",
              needReset: true
            });
          });
      });
    },
    delQa(video_qa_id) {
      this.$axios
        .delete("/video/admin/qa/", {
          data: {
            video_qa_id
          }
        })
        .then(() => {
          this.$message.success("删除成功!");
          this.getTableData();
        })
        .catch(err => {
          this.$notification.error({
            message: "答疑视频删除失败",
            description: err.response.data.detail,
            duration: 3
          });
        });
    },

    modalShow({ modalObj, paramObj, type }) {
      this.type = type === "add" ? "添加" : "编辑";
      if (paramObj) {
        modalObj.form = { ...Object.assign({}, modalObj.form, paramObj) };
      }
      modalObj.visible = true;
    },
    modalHidden({ modalObj, refName, needReset }) {
      if (needReset) {
        modalObj.form = {
          name: null,
          video_url: null,
          tags: [],
          remark: null
        };
        this.$refs[refName].resetFields();
      }
      modalObj.visible = false;
    },
    descTitle(k) {
       let lab = qaTableColumns.find(t => t.dataIndex == k)?.title;
       if (!lab) {
           if (k === "created_at") {
            lab = "创建时间"
           } else if (k === "creator") {
               lab = "创建者"
           }
       }
       return lab;
    }
  }
};
</script>
```

