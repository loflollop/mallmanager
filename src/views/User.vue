<template>
<el-card class="box-card">
    <!-- 面包屑 -->
    <el-breadcrumb separator="/">
        <el-breadcrumb-item>首页</el-breadcrumb-item>
        <el-breadcrumb-item>用户管理</el-breadcrumb-item>
        <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索框 -->
    <el-row class="searchArea">
        <el-col :span="24">
            <el-input placeholder="请输入内容" class="searchInput" v-model="searchVal">
                <el-button slot="append" icon="el-icon-search" @click="checkUser()"></el-button>
            </el-input>
            <el-button type="success" @click="showAddUserDia()">添加用户</el-button>
        </el-col>
    </el-row>
    <!-- 添加用户对话框 -->
    <el-dialog title="添加用户" :visible.sync="dialogFormVisibleAdduser">
        <el-form :model="formData">
            <el-form-item label="用户名" :label-width="formLabelWidth">
                <el-input v-model="formData.username" autocomplete="off"></el-input>

            </el-form-item>
            <el-form-item label="密码" :label-width="formLabelWidth">
                <el-input v-model="formData.password" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="邮箱" :label-width="formLabelWidth">
                <el-input v-model="formData.email" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="电话" :label-width="formLabelWidth">
                <el-input v-model="formData.mobile" autocomplete="off"></el-input>
            </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisible = false">取 消</el-button>
            <el-button type="primary" @click="addUser()">确 定</el-button>
        </div>
    </el-dialog>
    <!-- 编辑用户对话框 -->
    <el-dialog title="编辑用户" :visible.sync="dialogFormVisibleEdituser">
        <el-form :model="formData">
            <el-form-item label="用户名" :label-width="formLabelWidth">
                <el-input disabled v-model="formData.username" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="邮箱" :label-width="formLabelWidth">
                <el-input v-model="formData.email" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="电话" :label-width="formLabelWidth">
                <el-input v-model="formData.mobile" autocomplete="off"></el-input>
            </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUser()">确 定</el-button>
        </div>
    </el-dialog>
    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限" :visible.sync="dialogFormVisibleSetrole">
        <el-form :model="formData">
            <el-form-item label="用户名" :label-width="formLabelWidth">
                {{currUserName}}
            </el-form-item>

            <el-form-item label="角色" :label-width="formLabelWidth">
              <!-- el-select的v-model绑定的值如果和el-option的value值 一样
                就显示当前el-option的label
                注意: currRoleId是数值
               -->
               <!-- // currRoleId = 30 -->
                <el-select v-model="currRoleId">
                  <!-- 请选择 -->
                  <el-option disabled label="请选择" :value="-1">

                  </el-option>
                  <!-- 遍历数组 -->
                    <el-option v-for="(item,index) in roles"
                    :label="item.roleName"
                    :key="index"
                    :value="item.id">
                    </el-option>
                </el-select>

            </el-form-item>

        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisibleSetrole = false">取 消</el-button>
            <el-button type="primary" @click="setRole()">确 定</el-button>
        </div>
    </el-dialog>
    <!-- 表格 -->
    <el-table v-loading="loading" :data="list" style="width: 100%">
        <el-table-column type="index" label="#" width="80">
        </el-table-column>

        <el-table-column prop="username" label="姓名" width="100">
        </el-table-column>

        <el-table-column prop="email" label="邮箱" width="100">
        </el-table-column>

        <el-table-column prop="mobile" label="电话" width="200">
        </el-table-column>

        <el-table-column label="创建日期" width="180">
            <template slot-scope="scope">
                {{scope.row.create_time | fmtDate}}
            </template>
        </el-table-column>

        <el-table-column prop="mg_state" label="用户状态" width="100">
            <!-- 单元格中的内容不是字符串时,如果是其他组件比如开关,需要在这些组件外加容器 -->
            <template slot-scope="scope">
                <!--slot-scope用来传值的
                "scope"是随便的命名
                scope是list
              scope.row 就是list中的每个元素 ,这里是{}
              scope.row.mg_state  -->
                <el-switch @change="changeSwitchMgstate(scope.row)" v-model="scope.row.mg_state" active-color="#13ce66" inactive-color="#ff4949">
                </el-switch>
            </template>
        </el-table-column>

        <el-table-column label="操作" width="300">
            <template slot-scope="scope">
                <el-row>
                    <el-button type="primary" icon="el-icon-edit" size="mini" plain circle @click="showEditBox(scope.row.id)"></el-button>
                    <el-button type="success" icon="el-icon-check" size="mini" plain circle @click="showRoleBox(scope.row.id)"></el-button>
                    <el-button type="danger" icon="el-icon-delete" size="mini" plain circle @click="showDeleBox(scope.row.id)"></el-button>
                </el-row>
            </template>
        </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="currentPage" :page-sizes="[2, 4, 6, 8]" :page-size="pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
    </el-pagination>

</el-card>
</template>

<script>
export default {
  data () {
    return {
      list: [],
      // 加载动画
      loading: false,
      // 分页相关的变量
      pagenum: 1,
      pagesize: 2,
      currentPage: 1,
      total: 0,
      // 查询关键字
      searchVal: '',
      // 添加用户对话框的属性
      dialogFormVisibleAdduser: false,
      // 表单数据
      formData: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 对话框中input 宽度
      formLabelWidth: '100px',
      // 编辑用户对话框属性显示
      dialogFormVisibleEdituser: false,
      // 分配权限对话框属性
      dialogFormVisibleSetrole: false,
      // 当前用户名
      currUserName: '',
      // 当前的角色名
      currRoleId: -1,
      // 所有的角色
      roles: [],
      // 用户id
      currUserId: -1
    }
  },
  created () {
    this.loadTableData()
  },
  methods: {
    // 分配权限 -发送请求
    // 主管 -> 测试角色
    async setRole () {
      // 发送请求
      const res = await this.$http.put(`users/${this.currUserId}/role`, {
        rid: this.currRoleId
      })
      // console.log(res)
      const {
        meta: { status, msg }
      } = res.data
      // 提示框
      this.$message.success(msg)
      // 关闭对话框
      this.dialogFormVisibleSetrole = false
      // users/:id/role   (rid:角色id)
      // 重置
      this.currRoleId = -1
    },
    // 分配权限 - 显示对话框
    async showRoleBox (user) {
      console.log('-----')

      console.log(user)

      // 分配角色时 url中要用的userId
      this.currUserId = user.id
      // 显示用户名
      this.currUserName = user.username

      this.dialogFormVisibleSetrole = true

      const res = await this.$http.get('roles')
      console.log(res) // res.data.data array

      const res2 = await this.$http.get(`users/${user.id}`)
      // console.log(res2);
      // console.log(res2.data.data.rid)
      // 接口文档有问题!!
      // currRoleId 某一个角色(主管等)对应的id
      this.currRoleId = res2.data.data.rid

      this.roles = res.data.data
      // console.log(this.roles)
    },
    // 编辑用户-发送请求
    async editUser () {
      // 关闭对话框
      this.dialogFormVisibleEdituser = false
      // 发送请求put
      const res = await this.$http.put(`users/${this.formData.id}`, this.formData)
      console.log(res)
      this.loadTableData()
      this.$message.success(res.data.meta.msg)
    },
    // 编辑用户-对话框的显示
    async showEditBox (userId) {
      // 显示对话框
      this.dialogFormVisibleEdituser = true
      // 发送请求
      const res = await this.$http.get(`users/${userId}`)
      this.formData = res.data.data
      // 提示框
    },
    // 添加用户
    async addUser () {
      // 关闭对话框
      this.dialogFormVisibleAdduser = false
      // 发送请求
      const res = await this.$http.post('users', this.formData)
      const {
        meta: {
          status,
          msg
        }
      } = res.data
      // 刷新视图
      this.loadTableData()
      // 提示框
      this.$message.success(msg)
      //  清空表单
      // this.formData.username = ''
      this.formData = {}
    },
    // 显示添加的对话框
    showAddUserDia () {
      this.dialogFormVisibleAdduser = true
    },
    // 显示删除提示框
    showDeleBox (userId) {
      this.$confirm('sure?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const res = await this.$http.delete(`users/${userId}`)
        // console.log(res)
        const {
          meta: {
            msg,
            status
          }
        } = res.data
        if (status === 200) {
          // 刷新表格
          this.loadTableData()
          // 提示框
          this.$message({
            type: 'success',
            message: msg
          })
        }
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    async changeSwitchMgstate (user) {
      // 改变用户状态
      // users/:uId/state/:type
      // :uid -> user.id
      // :type ->true 或者false
      const res = await this.$http.put(`users/${user.id}/state/${user.mg_state}`)
      console.log(res)
      const {
        meta: {
          status,
          msg
        }
      } = res.data
      if (status === 200) {
        // 提示框
        this.$message.success(msg)
      }
    },
    // 查询用户
    checkUser () {
      this.loadTableData()
    },
    // 分页相关的方法
    handleSizeChange (val) {
      this.pagesize = val
      this.loadTableData()
      //   console.log(`每页 ${val} 条`)
    },
    handleCurrentChange (val) {
      this.pagenum = val
      this.loadTableData()
      //   console.log(`当前页: ${val}`)
    },
    async loadTableData () {
      this.loading = true
      // 除了登录功能, 其他功能的接口都需要加入token 才能请求
      // header添加token
      const AUTH_TOKEN = sessionStorage.getItem('token')
      this.$http.defaults.headers.common['Authorization'] = AUTH_TOKEN
      const res = await this.$http.get(`users?pagenum=${this.pagenum}&pagesize=${this.pagesize}&query=${this.searchVal}`)
      console.log(res)
      this.total = res.data.data.total
      const {
        meta: {
          msg,
          status
        },
        data: {
          users
        }
      } = res.data
      if (status === 200) {
        this.loading = false
        this.list = users
        // 默认显示
        this.pagenum = 1
        // this.pagesize = 2
        // this.currentPage = 1
        // console.log(this.list)
      }
    }
  }
}
</script>

<style>
.box-card {
    height: 100%;
}

.searchArea {
    margin-top: 10px;
    margin-bottom: 10px;
}

.searchArea .searchInput {
    width: 350px;
}
</style>
