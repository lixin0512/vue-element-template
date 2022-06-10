<template>
  <div class="app-container">
    <div class="app-title">数据视图表格</div>
    <div class="option">
      <div>
        <el-button @click="importExcel" size="mini" type="primary">导入模板</el-button>
        <el-button @click="handleDownload" size="mini">导出模板</el-button>
      </div>
      <div>
        <el-input class="search" v-model="search" ref='searchInput' size="mini" @keyup.enter.native="submitSearch"
          placeholder="请输入关键字"></el-input>
      </div>
    </div>
    <el-table v-loading="listLoading" row-key='ID+1' stripe show-header
      :data="lists.slice((currentPage - 1) * pageSize, currentPage * pageSize)" element-loading-text="数据请求中..." border
      fit highlight-current-row tooltip-effect="dark" max-height="700">
      <el-table-column align="center" label="编号" prop="ID" width="60" show-overflow-tooltip>
      </el-table-column>
      <el-table-column label="站名" width="150" prop="StationName" show-overflow-tooltip>
      </el-table-column>
      <el-table-column label="连接名称" width="140" prop="DataSource" show-overflow-tooltip>
      </el-table-column>
      <el-table-column label="名称" width="160" prop="Name" show-overflow-tooltip>
      </el-table-column>
      <el-table-column label="全称" width="320" prop="FullName" show-overflow-tooltip>
      </el-table-column>
      <el-table-column label="修改时间" width="220" show-overflow-tooltip>
        <template slot-scope="scope">
          {{ scope.row.UpdateTime.split('.')[0].replace(/T/, ' ') }}
        </template>
      </el-table-column>
      <el-table-column label="类别" width="80" show-overflow-tooltip>
        <template slot-scope="scope">
          {{ scope.row.Type }}
        </template>
      </el-table-column>
      <el-table-column label="默认值" width="80" show-overflow-tooltip>
        <template slot-scope="scope">
          {{ scope.row.Value }}
        </template>
      </el-table-column>
      <el-table-column label="描述" width="240" prop="Description" show-overflow-tooltip>
      </el-table-column>
      <el-table-column label="地址" prop="DataAddr" show-overflow-tooltip width="300">
      </el-table-column>
      <el-table-column fixed="right" align="center" label="操作" width="100">

        <template slot-scope="scope">
          <el-button @click="handleEditClick(scope.row)" type="text" size="small">编辑</el-button>
          <el-button @click="handleDeleteClick(scope.row)" type="text" size="small">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="block" v-if="lists.length">
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
        :current-page.sync="currentPage" :page-sizes="[50, 100, 150, 200]" :page-size="pageSize" background
        layout="total, sizes, prev, pager, next, jumper" :total="lists.length">
      </el-pagination>
    </div>
    <el-drawer title="属性配置" :visible.sync="drawer" :direction="direction" :before-close="handleClose" ref="drawer">
      <el-form :model="ruleForm" ref="ruleForm" label-width="100px">
        <el-form-item label="站名" prop="StationName" required>
          <el-input v-model="ruleForm.StationName"></el-input>
        </el-form-item>
        <el-form-item label="名称" prop="Name" required>
          <el-input v-model="ruleForm.Name"></el-input>
        </el-form-item>
        <el-form-item label="连接名称" prop="DataSource" required>
          <el-input v-model="ruleForm.DataSource"></el-input>
        </el-form-item>
        <el-form-item label="全称" prop="FullName" required>
          <el-input v-model="ruleForm.FullName"></el-input>
        </el-form-item>
        <el-form-item label="修改时间" prop="UpdateTime" required>
          <el-input v-model="ruleForm.UpdateTime"></el-input>
        </el-form-item>
        <el-form-item label="类别" prop="Type" required>
          <el-input v-model="ruleForm.Type"></el-input>
        </el-form-item>
        <el-form-item label="默认值" prop="Value">
          <el-input v-model="ruleForm.Value"></el-input>
        </el-form-item>
        <el-form-item label="描述" prop="Description">
          <el-input type="textarea" v-model="ruleForm.Description"></el-input>
        </el-form-item>
        <el-form-item label="地址" prop="DataAddr" required>
          <el-input v-model="ruleForm.DataAddr"></el-input>
        </el-form-item>
      </el-form>
      <el-divider></el-divider>
      <div class="drawer-footer">
        <el-button type="primary" @click="submitForm()" :drawerLoading="drawerLoading">{{ drawerLoading ?
            '提交中 ...' : '确 定'
        }}</el-button>
        <el-button @click="drawer = false">取 消</el-button>
      </div>
    </el-drawer>

  </div>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return {
      search: '',
      lists: [],
      listLoading: true,
      downloadLoading: false,
      drawerLoading: false,
      getInfo: 'http://192.168.0.40:8088/',
      pageSize: 100,
      currentPage: 1,
      filename: 'asset-type(opcua)',
      bookType: 'xlsx',
      drawer: false,
      direction: 'rtl',
      num: 0,
      ruleForm: {
        StationName: '',
        Name: '',
        DataSource: '',
        FullName: '',
        UpdateTime: '',
        Type: '',
        Description: '',
        DataAddr: ''

      }
    }
  },
  created() {
    this.getData();
    this.currentPage = parseInt(localStorage.getItem('pagination') || 1)
    this.handleCurrentChange(this.currentPage)
  },
  watch: {

  },
  methods: {
    getData() {
      this.listLoading = true
      axios.get(this.getInfo + 'getInfo').then(response => {
        this.lists = response.data.recordset
        this.listLoading = false
      })
    },
    submitSearch() {
      if (this.search.trim().length) {
        let params = {
          search: this.search.trim()
        }
        this.lists = []
        axios.get(this.getInfo + 'search', {
          params
        }).then(res => {
          this.lists = res.data
          // let map = new Map();
          // for (let item of res.data.data) {
          //   if (!map.has(item.id)) {
          //     map.set(item.id, item);
          //   };
          // };
          // this.lists = [...map.values()];
        })
      } else {
        this.getData()
      }
    },
    handleSizeChange(val) {
      // console.log(`每页 ${val} 条`);
      this.pageSize = val
      this.getData()
    },
    beforeDestroy() {
      localStorage.setItem('pg', 200)
    },
    handleCurrentChange(val) {
      // console.log(`当前页: ${val}`);
      this.currentPage = val
      localStorage.setItem('pagination', val)
      this.getData()
    },
    handleEditClick(row) {
      this.drawer = true
      for (let key in row) {
        this.ruleForm[key] = row[key]
      }
    },
    handleClose(done) {
      this.$confirm('确认关闭？')
        .then(_ => {
          done();
        })
        .catch(_ => { });
    },
    handleDeleteClick(row) {
      console.log(row.ID);
      let id = row.ID;
      this.$confirm('此操作将永久删除该点位, 是否继续?', '提示', {
        cancelButtonText: '取消',
        confirmButtonText: '确定',
        type: 'warning'
      }).then(() => {
        axios.get(this.getInfo + 'deleteInfo?' + id).then(res => {
          this.$message({
            type: res.data.state,
            message: res.data.msg
          });
          this.getData()
        })

      }).catch(() => {
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },

    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['编号', '站名', '连接名', '全称', '修改时间', '类别', '默认值',  '描述', '地址']//
        const filterVal = ['ID', 'StationName', 'DataSource', 'FullName', 'UpdateTime', 'Type', 'Value', 'Description ', 'DataAddr']
        const lists = this.lists
        const data = this.formatJson(filterVal, lists)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: this.filename,
          autoWidth: true,
          bookType: this.bookType
        })
        this.downloadLoading = false
      })
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => {
        if (j === 'timestamp') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    },
    importExcel() { },
    submitForm() {
      // console.log(this.ruleForm)
      if (this.drawerLoading) {
        return;
      }
      this.$confirm('确定要修改数据吗？')
        .then(_ => {
          this.drawerLoading = true;
          let params = this.ruleForm
          console.log(this.ruleForm);
          axios.get(this.getInfo + 'updateInfo', { params }).then(res => {
            this.drawerLoading = false
            this.$message({
              message: res.data.msg,
              type: res.data.state
            })

          }).then(() => {
            let timer = setTimeout(async () => {
              this.drawerLoading = false
              this.drawer = false
              this.getData()
            }, 1000);

          })

        })
        .catch(_ => { });
    }
  }
}
</script>
<style>
.el-table td,
.el-table th {
  padding: 6px 0;
}

.option {
  margin: 8px 0 4px;
  display: flex;
  justify-content: space-between;
}

.el-table td:hover {
  cursor: pointer;
}

.block {
  float: right;
  padding-top: 16px;
}

.drawer-footer {
  text-align: center;
}

.search {
  width: 260px;
}
</style>
