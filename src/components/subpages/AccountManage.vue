<template>
  <div class="account-manage">
    <el-dialog :title="'账号管理'" :before-close="close" :show-close="false" :visible.sync="dialogVisible" width="1000px">
      <el-tabs ref="accountTabs" v-model="tabNum">
        <!-- 账号列表 -->
        <el-tab-pane label="账号列表" style="text-align:center" name="1">
          <el-table :data="userData" style="width: 100%">
            <el-table-column prop="account" label="账号" width="180">
            </el-table-column>
            <el-table-column prop="name" label="昵称" width="180">
            </el-table-column>
            <el-table-column label="权限" width="180">
              <template slot-scope="scope">
                <el-tag type="success" v-show="scope.row.auth.canCharge">充值</el-tag>
                <el-tag v-show="scope.row.auth.canNotify">通知</el-tag>
                <p v-show="!scope.row.auth.canNotify && !scope.row.auth.canCharge">暂无</p>
              </template>
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="small" type="primary" @click="showAuthChange(scope.row)">修改权限</el-button>
                <el-button size="small" type="danger" @click="openDeleteConfirm(scope.row)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加账号页 -->
        <el-tab-pane label="添加账号" name="2">
          <el-form class="account-dialog" ref="addAccountForm" :rules="rules" :model="addAccountForm" label-width="80px" width="300px">
            <el-form-item label="账号" prop="account">
              <el-input v-model="addAccountForm.account"></el-input>
            </el-form-item>
            <el-form-item label="昵称" prop="name">
              <el-input v-model="addAccountForm.name"></el-input>
            </el-form-item>
            <el-form-item label="密码" prop="pass">
              <el-input type="password" v-model="addAccountForm.pass"></el-input>
            </el-form-item>
            <el-form-item label="重复密码" prop="repassword">
              <el-input type="password" v-model="addAccountForm.repassword"></el-input>
            </el-form-item>
            <el-form-item label="权限选择" prop="auth">
              <el-checkbox v-model="addAccountForm.auth.canCharge">充值权限</el-checkbox>
              <el-checkbox v-model="addAccountForm.auth.canNotify">通知权限</el-checkbox>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="submitAddAccount">添加账号</el-button>
              <el-button @click="close">关闭</el-button>
            </el-form-item>
          </el-form>
        </el-tab-pane>
      </el-tabs>
      <!-- 修改权限 -->
      <el-dialog width="400" title="修改账号权限" :visible.sync="changeAuthVisible" append-to-body>
        <el-form ref="changeAuth" :model="changeAuthForm" label-width="80px">
          <el-form-item label="账号">
            <span>{{changeAuthForm.account}}</span>
          </el-form-item>
          <el-form-item label="昵称">
            <span>{{changeAuthForm.name}}</span>
          </el-form-item>
          <el-form-item label="权限">
            <el-checkbox v-model="changeAuthForm.auth.canCharge">充值</el-checkbox>
            <el-checkbox v-model="changeAuthForm.auth.canNotify">通知</el-checkbox>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="submitAuthChange">确认修改</el-button>
            <el-button @click="changeAuthVisible = false">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>
    </el-dialog>
  </div>
</template>



<script>
import { isLogin, getAdmins, editAdmin, delAdmin } from "../../api/getData";

export default {
  props: {
    dialogVisible: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      addAccountForm: {
        id: "", // 新增用户时传空
        account: "",
        name: "",
        pass: "",
        repassword: "",
        // 充值权限
        auth: {
          canCharge: false,
          canNotify: false
        }
      },
      rules: {
        account: [{ required: true, message: "请输入账号", trigger: "blur" }],
        name: [{ required: true, message: "请输入昵称", trigger: "blur" }],
        pass: [
          { required: true, message: "请输入密码", trigger: "blur" },
          { min: 6, message: "密码不能少于6个字符", trigger: "blur" }
        ],
        repassword: [
          { required: true, validator: this.validaterePass, trigger: "blur" }
        ]
      },
      userData: [],
      changeAuthForm: {
        id: "",
        account: "",
        name: "",
        auth: {
          canCharge: false,
          canNotify: false
        }
      },
      // 是否显示权限修改界面
      changePasswordVisible: false,
      changeAuthVisible: false,
      // 初始化的标签的index
      tabNum: "1"
    };
  },
  mounted() {
    //   this.getUsers();
  },
  methods: {
    close() {
      this.$emit("close");
    },
    /* 获取用户列表 */
    async getUsers() {
      const res = await getAdmins();
      console.log(res);
      if (res.ok) {
        this.userData = res.list;
      }
    },
    /* 展示权限修改 */
    showAuthChange(data) {
      let copy = JSON.parse(JSON.stringify(data));
      console.log(copy);
      this.changeAuthVisible = true;
      this.changeAuthForm.id = copy.id;
      this.changeAuthForm.account = copy.account;
      this.changeAuthForm.name = copy.name;
      this.changeAuthForm.auth = copy.auth;
    },
    /* 确认权限修改 */
    async submitAuthChange() {
      console.log(this.changeAuthForm);
      let res = await editAdmin(this.changeAuthForm);
      if (res.ok) {
        this.$message({
          type: "success",
          message: "修改成功"
        });
        this.getUsers();
        this.changeAuthVisible = false;
      } else {
        this.$message({
          type: "error",
          message: res.errorMsg
        });
      }
    },
    /* 确认添加账号 */
    async submitAddAccount() {
      this.$refs.addAccountForm.validate(async valid => {
        if (valid) {
          let res = await editAdmin(this.addAccountForm);
          console.log(res);
          if (res.ok) {
            this.$message({
              type: "success",
              message: "添加成功"
            });
            this.getUsers();
            this.tabNum = "1";
            this.$refs.addAccountForm.resetFields();
          } else {
            this.$message({
              type: "error",
              message: res.errorMsg
            });
          }
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    async openDeleteConfirm(data) {
      this.$confirm(`您确定要删除${data.name}的账号吗？`, "确认操作", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "info"
      })
        .then(async () => {
          let res = await delAdmin({ id: data.id });
          if (res.ok) {
            this.$message({
              type: "success",
              message: "删除成功"
            });
            this.getUsers();
          } else {
            this.$message({
              type: "error",
              message: res.errorMsg
            });
          }
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消修改"
          });
        });
    },
    validaterePass(rule, value, callback) {
      if (value === "") {
        callback(new Error("请再次输入密码"));
      } else if (value !== this.addAccountForm.pass) {
        callback(new Error("两次输入密码不一致!"));
      } else {
        callback();
      }
    }
  },
  watch: {
    dialogVisible(newVal) {
      if (newVal) {
        this.getUsers();
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.account-dialog {
  width: 500px;
  margin: 0 auto;
}
.el-table th > .cell {
  text-align: center;
}
</style>
