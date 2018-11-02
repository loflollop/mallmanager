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
            <el-button type="success">添加用户</el-button>
        </el-col>
    </el-row>
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
                <el-switch v-model="scope.row.mg_state" active-color="#13ce66" inactive-color="#ff4949">
                </el-switch>
            </template>
        </el-table-column>

        <el-table-column label="操作" width="300">
            <template slot-scope="scope">
                <el-row>
                    <el-button type="primary" icon="el-icon-edit" size="mini" plain circle></el-button>
                    <el-button type="success" icon="el-icon-check" size="mini" plain circle></el-button>
                    <el-button type="danger" icon="el-icon-delete" size="mini" plain circle></el-button>
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
      searchVal: ''
    }
  },
  created () {
    this.loadTableData()
  },
  methods: {
    // 查询用户
    checkUser () {
      this.loadTableData()
    },
    // 分页相关的方法
    handleSizeChange (val) {
      this.pagesize = val
      this.loadTableData()
      console.log(`每页 ${val} 条`)
    },
    handleCurrentChange (val) {
      this.pagenum = val
      this.loadTableData()
      console.log(`当前页: ${val}`)
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
        this.pagesize = 2
        this.currentPage = 1
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
