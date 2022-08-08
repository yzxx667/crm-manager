<template>
  <div class="deptPage">
    <div class="query-form">
      <el-form
        ref="deptDialog"
        :model="deptForm"
        label-width="80px"
        :inline="true"
      >
        <el-form-item label="部门名称">
          <el-input
            v-model="deptForm.deptName"
            placeholder="请输入部门名称"
          ></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary">查询</el-button>
          <el-button @click="handleReset('deptDialog')">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="base-table">
      <div class="query-form" style="padding-bottom: 20px">
        <el-button type="primary" @click="handleOpen">创建</el-button>
      </div>
      <el-table
        :data="deptData"
        style="width: 100%"
        row-key="_id"
        default-expand-all
        :tree-props="{ children: 'children', hasChildren: 'hasChildren' }"
      >
        <el-table-column
          v-for="item in columns"
          :key="item.prop"
          :prop="item.prop"
          :label="item.label"
        >
        </el-table-column>
        <el-table-column label="操作">
          <template #default="scope">
            <el-button type="primary" @click="handleEdit(scope.row)"
              >编辑</el-button
            >
            <el-button type="danger" @click="handleDel(scope.row._id)"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </div>
    <el-dialog v-model="showModal" title="创建部门">
      <el-form
        :model="handleDeptForm"
        :rules="rules"
        label-width="120px"
        ref="deptdia"
      >
        <el-form-item label="上级部门名称" prop="parentId">
          <el-cascader
            v-model="handleDeptForm.parentId"
            :options="deptData"
            :props="{ checkStrictly: true, value: '_id', label: 'deptName' }"
            :popper-append-to-body="false"
            clearable
          ></el-cascader>
        </el-form-item>
        <el-form-item label="部门名称" prop="deptName">
          <el-input
            placeholder="请输入部门名称"
            v-model="handleDeptForm.deptName"
          ></el-input>
        </el-form-item>
        <el-form-item label="负责人" prop="userName">
          <el-select
            v-model="handleDeptForm.user"
            clearable
            placeholder="请选择"
            @change="handleChange()"
            :popper-append-to-body="false"
          >
            <el-option
              v-for="item in allUserList"
              :key="item.value"
              :label="item.userName"
              :value="`${item.userId}/${item.userName}/${item.userEmail}`"
            >
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="负责人邮箱">
          <el-input
            placeholder="请输入负责人邮箱"
            disabled
            v-model="handleDeptForm.userEmail"
          ></el-input>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="handleClose">取消</el-button>
          <el-button type="primary" @click="handleSubmit">确定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "dept",
  data() {
    return {
      //查询query
      deptForm: {
        deptName: "",
      },
      // 列表数据
      deptData: [],
      columns: [
        {
          label: "部门名称",
          prop: "deptName",
        },
        {
          label: "负责人",
          prop: "userName",
        },
        {
          label: "更新时间",
          prop: "updateTime",
        },
        {
          label: "创建时间",
          prop: "createTime",
        },
      ],
      //分页
      pager: {
        pageNum: 1,
        pageSize: 10,
      },
      showModal: false,
      action: "",
      rules: {
        parentId: [
          {
            required: true,
            message: "请选择上级部门",
            trigger: "blur",
          },
        ],
        deptName: [
          {
            required: true,
            message: "请输入部门名称",
            trigger: "blur",
          },
        ],
        userName: [
          {
            required: true,
            message: "请输入部门负责人",
            trigger: "blur",
          },
        ],
      },
      handleDeptForm: {},
      //角色列表
      allUserList: [],
      selectPer: "",
      selectEmail: "",
    };
  },
  mounted() {
    this.getDeptList();
    this.getAllUserList();
  },
  methods: {
    async getDeptList() {
      let list = await this.$api.getDeptList({
        ...this.pager,
        ...this.deptForm,
      });
      this.deptData = list;
    },
    async getAllUserList() {
      this.allUserList = await this.$api.getAllUserList();
    },
    handleReset(form) {
      this.$refs[form].resetFields();
    },
    handleOpen() {
      this.showModal = true;
      this.action = "create";
    },
    handleClose() {
      this.showModal = false;
      this.handleReset("deptdia");
    },
    handleSubmit() {
      this.$refs["deptdia"].validate(async (valid) => {
        if (valid) {
          let params = { ...this.handleDeptForm, action: this.action };
          let res = await this.$api.deptOperate(params);
          if (res) {
            this.showModal = false;
            this.handleClose();
            this.$message.success("操作成功");
            this.getDeptList();
          }
        }
      });
    },
    handleChange() {
      let [userId, userName, userEmail] = this.selectPer.split("/");
      this.handleDeptForm = Object.assign(
        { ...this.handleDeptForm },
        { userId, userName, userEmail }
      );
    },
    handleEdit(row) {
      console.log(row);
      this.showModal = true;
      this.action = "edit";
      this.$nextTick(() => {
        Object.assign(this.handleDeptForm, row, {
          user: `${row.userId}/${row.userName}/${row.userEmail}`,
        });
      });
    },
    async handleDel(row) {
      this.action = "delete";
      await this.$api.deptOperate({ _id: row, action: this.action });
      this.$message.success("操作成功");
      this.showModal = false;
      this.getDeptList();
    },
  },
};
</script>

<style lang="scss" scoped>
</style>