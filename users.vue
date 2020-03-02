<template>
  <el-card class="box-card">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索 -->
    <el-row class="searchRow">
      <el-col>
        <el-input
          @clear="loadUserList()"
          clearable
          placeholder="请输入内容"
          v-model="query"
          class="inputSearch"
        >
          <el-button slot="append" icon="el-icon-search" @click="searchUser()"></el-button>
        </el-input>
        <el-button type="success" @click="dialogFormVisibleAdd = true">添加用户</el-button>
      </el-col>
    </el-row>
    <el-table :data="tableData" style="width: 100%">
      <el-table-column label="#" type="index"></el-table-column>
      <el-table-column prop="name" label="姓名" width="80"></el-table-column>
      <el-table-column prop="email" label="邮箱" width="80"></el-table-column>
      <el-table-column prop="address" label="地址"></el-table-column>
      <el-table-column label="创建日期">
        <template slot-scope="userList">{{userList.row.create_time | fmtdata}}</template>
      </el-table-column>
      <el-table-column label="用户状态">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.msg_state" active-color="#13ce66" inactive-color="#ff4949"></el-switch>
        </template>
      </el-table-column>
      <el-table-column prop="address" label="操作">
        <template slot-scope="scope">
          <el-button
            size="mini"
            @click="showEdit(scope.row)"
            plain
            type="primary"
            icon="el-icon-edit"
            circle
          ></el-button>
          <el-button size="mini" plain type="danger" icon="el-icon-delete" circle></el-button>
          <el-button size="mini" plain type="success" icon="el-icon-check" circle></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="20"
    ></el-pagination>
    <!-- 添加 -->
    <el-dialog title="添加用户" :visible.sync="dialogFormVisibleAdd">
      <el-form :model="form">
        <el-form-item label="用户名" :label-width="formLabelWidth">
          <el-input v-model="form.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="密码" :label-width="formLabelWidth">
          <el-input v-model="form.password" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisibleAdd = false">取 消</el-button>
        <el-button type="primary" @click="handleSubmit()">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 编辑 -->
    <el-dialog title="编辑用户" :visible.sync="dialogFormVisibleEdit
    ">
      <el-form :model="form">
        <el-form-item label="用户名" :label-width="formLabelWidth">
          <el-input v-model="form.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" :label-width="formLabelWidth">
          <el-input v-model="form.email" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" :label-width="formLabelWidth">
          <el-input v-model="form.mobile" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisibleAdd = false">取 消</el-button>
        <el-button type="primary" @click="editUser()">确 定</el-button>
      </div>
    </el-dialog>
  </el-card>
</template>
<script>
export default {
  data() {
    return {
      query: "",
      value: true,
      dialogFormVisibleAdd: false,
      dialogFormVisibleEdit: false,
      form: {
        username: "",
        password: "",
        email: "",
        mobile: ""
      },
      tableData: [
        {
          name: "劉",
          address: "廈門",
          email: "1111@qq.com"
        },
        {
          name: "劉",
          address: "廈門",
          email: "1111@qq.com"
        }
      ],
      userList: [],
      total: "20",
      pagenum: 1,
      pagesize: 2
    };
  },
  created() {
    this.getUserList();
  },
  methods: {
    showEdit(user) {
      this.form = user;
      // 将要修改的数据赋值给form，user中包含user id
      this.dialogFormVisibleEdit = true;
    },
    async editUser() {
      // 从form中获得userid
      const res = await this.$http.put('users/${this.form.id',this.form)
      if(res.data.status===200){
        this.$message.success('修改成功')
        this.dialogFormVisibleEdit = false
      }else{
        this.$message.warning('修改失败');
        
      }
    },
    //getUserList()方法复用性高
    async getUserList() {
      const AUTH_TOKEN = localStorage.getItem("token");
      this.$http.defaults.headers.common["Authorization"] = AUTH_TOKEN;
      const res = await this.$http.get(
        "users?query=${this.query}&pagenum=${this.pagenum}&pagesize=${this.pagesize}"
      );
      const {
        data: { users, total ,status,msg}
      } = res.data;
      if (status === 200) {
        //   给表格数据赋值
        //   给total赋值
        this.userList = users;
        this.total = total;
        this.$message({ msg });
      } else {
        this.$message({ msg });
      }
    },
    showDeteleUserMsgBox(userId) {
      this.$confirm("刪除？", "提示", {
        confirmButtonText: "確定",
        cancelButtonText: "取消",
        type: "Warning"
      })
        .then(async () => {
          const res = await this.$http.delete("user/${userId}");
          if (res.data.status === 200) {
            this.$message({
              type: "success",
              message: "成功"
            });
          }
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "取消"
          });
        });
    },
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
    },
    searchUser() {
      this.getUserList();
    },
    loadUserList() {
      this.getUserList();
    },
    async handleSubmit() {
      this.dialogFormVisibleAdd = false;
      const res = await this.$http.post("users", this.form);
      const {
        data:{ status, msg },
      } = res.data;
      if (status === 200) {
        this.$message.success(msg);
        this.getUserList();
        this.form = {};
      } else {
        this.$message.error(msg);
      }
    },
    async handleChange(user) {
      const res = await this.$$http.put("url/:uid/sate/:type");
    }
  }
};
</script>
<style>
.box-card {
  height: 100%;
}
.inputSearch {
  width: 300px;
}
.searchRow {
  margin-top: 20px;
}
</style>