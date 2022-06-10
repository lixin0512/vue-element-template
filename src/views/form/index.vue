<template>
  <div class="app-container">
    <el-upload ref="upload" action accept=".xls,.xlsx" :on-change="handleChange" :show-file-list="false" size="mini"
      :auto-upload="false">
      <el-button size="small" type="primary" style="margin-bottom:15px;">读取excel文件</el-button>
      <el-button size="mini" @click="insertData">插入数据</el-button>
      <div slot="tip" class="el-upload__tip">只能上传xls,.xlsx文件</div>
    </el-upload>
    <el-table ref="dragTable" key-row="ID" :data="tableData" border style="width: 100%">
      <!-- <el-table-column align="center" label="编号" prop="ID" width="60" show-overflow-tooltip> -->
      <!-- </el-table-column> -->
      <el-table-column label="编号" width="150" show-overflow-tooltip>
        <template slot-scope="{row}">
          <span>{{ row.ID }}</span>
        </template>
      </el-table-column>
      <el-table-column label="站名" width="150" show-overflow-tooltip>
        <template slot-scope="scope">
          {{ scope.row.StationName }}
        </template>
      </el-table-column>
      <el-table-column label="连接名称" show-overflow-tooltip width="140">
        <template slot-scope="scope">
          {{ scope.row.DataSource }}
        </template>
      </el-table-column>
      <el-table-column label="名称" width="160" show-overflow-tooltip>
        <template slot-scope="scope">
          {{ scope.row.Name }}
        </template>
      </el-table-column>
      <el-table-column label="全称" width="320" show-overflow-tooltip>
        <template slot-scope="scope">
          {{ scope.row.FullName }}
        </template>
      </el-table-column>
      <el-table-column label="修改时间" width="220" show-overflow-tooltip>
        <template slot-scope="scope">
          {{ scope.row.UpdateTime }}
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
      <el-table-column label="描述" width="240" show-overflow-tooltip>
        <template slot-scope="scope">
          {{ scope.row.Description }}
        </template>
      </el-table-column>
      <el-table-column label="地址" width="300" show-overflow-tooltip>
        <template slot-scope="scope">
          {{ scope.row.DataAddr }}
        </template>
      </el-table-column>
      <el-table-column fixed="right" align="center" label="操作" width="100">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="handleEditClick(scope.row)">编辑</el-button>
          <el-button type="text" size="small" @click="handleDeleteClick(scope.row)">删除</el-button>
        </template>
      </el-table-column>
      <el-table-column align="center" label="Drag" width="80">
        <template slot-scope="{}">
          <i class="el-icon-rank" />
        </template>
      </el-table-column>
    </el-table>
    <div class="show-d">
      <el-tag>The default order :</el-tag> {{ oldList }}
    </div>
    <div class="show-d">
      <el-tag>The after dragging order :</el-tag> {{ newList }}
    </div>
  </div>
</template>

<script>
import xlsx from 'xlsx'
import axios from 'axios'
import Sortable from 'sortablejs'
export default {
  data() {
    return {
      tableData: [],
      fileContent: '',
      file: '',
      data: '',
      oldList: [],
      newList: [],
      importLoading: false,
      sortable: null,
    }
  },
  methods: {
    handleDelete(item) {
      console.log('handleDelete', item)
    },
    readFile(file) {
      return new Promise(resolve => {
        const reader = new FileReader()
        reader.readAsBinaryString(file)
        reader.onload = ev => {
          resolve(ev.target.result)
        }
      })
    },
    insertData() {
      if (!this.tableData.length) {
        this.$message.info('请先读取excel在插入数据')
        return
      }
      var params = {
        tableData: JSON.stringify(this.tableData)
      }
      axios.get('http://192.168.0.40:8088/insertData', {
        params
      }).then(res => {
        this.$message({
          message: res.data.msg,
          type: res.data.state
        })
        this.tableData = []
      })
    },
    async handleChange(e) {
      this.inputFileName = e.name
      const file = e.raw
      if (!file) return
      // 读取file文件的内容(转换为json格式)
      const data = await this.readFile(file)
      //   // type :'binary' 类型为二进制
      let importData = xlsx.read(data, { type: 'binary' })
      console.log('e', importData)
      const importDataSheet = importData.Sheets[importData.SheetNames[0]]
      importData = xlsx.utils.sheet_to_json(importDataSheet) // 将解析出的数据转换为json格式（xlsx自带的方法）
      // eleData = eleData.length >1? eleData[1] : eleData[0]
      this.tableData = importData
      this.oldList = this.tableData.map(v => v.ID)
      this.newList = this.oldList.slice()
      this.$nextTick(() => {
        this.setSort()
      })
    },
    handelEdit(row) {
      console.log('handleDelete', row)
    },
    handleDeleteClick(row) {
      console.log('handleDeleteClick', row)
    },
    setSort() {
      const el = this.$refs.dragTable.$el.querySelectorAll('.el-table__body-wrapper > table > tbody')[0]
      this.sortable = Sortable.create(el, {
        ghostClass: 'sortable-ghost', // Class name for the drop placeholder,
        setData: function (dataTransfer) {
          // to avoid Firefox bug
          // Detail see : https://github.com/RubaXa/Sortable/issues/1012
          dataTransfer.setData('Text', '')
        },
        onEnd: evt => {
          const targetRow = this.tableData.splice(evt.oldIndex, 1)[0]
          console.log(this.tableData);
          this.tableData.splice(evt.newIndex, 0, targetRow)

          // for show the changes, you can delete in you code
          const tempIndex = this.newList.splice(evt.oldIndex, 1)[0]
          this.newList.splice(evt.newIndex, 0, tempIndex)
        }
      })
    }
  }
}
</script>

<style scoped>
.line {
  text-align: center;
}
.icon-star{
  margin-right:2px;
}
.drag-handler{
  width: 20px;
  height: 20px;
  cursor: pointer;
}
.show-d{
  margin-top: 15px;
}
.sortable-ghost{
  opacity: .8;
  color: #fff!important;
  background: #42b983!important;
}
</style>

