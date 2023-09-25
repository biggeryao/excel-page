<template>
  <div>
    <div style="display: flex;align-items: baseline">
      <!--      <el-upload-->
      <!--          class="upload-demo"-->
      <!--          :on-change="handleChange"-->
      <!--          :show-file-list="false"-->
      <!--          :auto-upload="false"-->
      <!--          multiple-->
      <!--          :file-list="fileList"-->
      <!--          action=""-->
      <!--      >-->
      <!--        <el-button size="small" type="primary" style="margin-bottom:15px;"-->
      <!--        >读取excel文件-->
      <!--        </el-button-->
      <!--        >-->

      <!--      </el-upload>-->
      <el-upload
          class="upload-demo"
          action="#"
          :auto-upload="false"
          :on-change="handleChange"
          :before-remove="handleBeforeRemove"
          multiple
          :file-list="fileList">
        <el-button size="small" type="primary">点击上传</el-button>
      </el-upload>
      <div class="ml" >
        <span>提取列名称：</span>
        <el-input  style="width: 200px" v-model="excelCellName"></el-input>
        <span class="ml" >提取方式：</span>
        <el-radio v-model="radio" label="1">提取单个单词</el-radio>
        <el-radio v-model="radio" label="2">完整名称</el-radio>
        <el-button class="ml" @click="handleGenerate" type="primary">生成</el-button>
      </div>
    </div>

    <el-table
        border
        height="600"
        :data="tableData"
        style="margin-top: 20px;"
        :default-sort="{prop: 'cnt', order: 'descending'}"
    >
      <el-table-column align="center" label="关键词" prop="name"></el-table-column>
      <el-table-column align="center" label="出现次数" prop="cnt" sortable></el-table-column>
    </el-table>
  </div>

</template>

<script>
import * as xlsx from "xlsx";
import XLSX from "xlsx";

export default {
  name: "tool",
  data() {
    return {
      fileList: [],
      tableData: [],
      fileContent: '',
      file: '',
      data: '',
      excelCellName: '热搜关键词',
      excelArr: [],
      radio: '1'
    }
  },
  watch: {},
  methods: {
    handleChange(file, fileList) {
      this.fileList = fileList
      this.excelArr = []
      this.dealFile()


    },
    async handleGenerate() {
      let obj = {}
      this.tableData = []
      if (this.fileList.length === 0) {
        return
      }

      if (this.radio === '1') {
        let arr = []
        this.excelArr.map(item => {
          if (item[this.excelCellName]) {
            arr = arr.concat(item[this.excelCellName].split(" "))
          }
        })
        arr.forEach(item => {
          if (obj[item]) {
            obj[item] += 1
          } else {
            obj[item] = 1
          }
        })
        this.tableData = Object.keys(obj).map((key) => {
          return {name: key, cnt: obj[key]}
        })
      } else {
        let arr = []
        this.excelArr.map(item => {
          if (item[this.excelCellName]) {
            arr = arr.concat(item[this.excelCellName])
          }
        })
        arr.forEach(item => {
          if (obj[item]) {
            obj[item] += 1
          } else {
            obj[item] = 1
          }
        })
        this.tableData = Object.keys(obj).map((key) => {
          return {name: key, cnt: obj[key]}
        })
      }
    },
    handleBeforeRemove(file) {
      this.excelArr = []
      this.fileList = this.fileList.filter(item => item.uid !== file.uid);
      this.dealFile()
    },
    // handleChange(file, fileList) {
    //   this.fileContent = file.raw
    //   const fileName = file.name
    //   const fileType = fileName.substring(fileName.lastIndexOf('.') + 1)
    //   if (this.fileContent) {
    //     if (fileType === 'xlsx' || fileType === 'xls') {
    //       this.importfile(this.fileContent)
    //     } else {
    //       this.$message({
    //         type: 'warning',
    //         message: '附件格式错误，请重新上传！'
    //       })
    //     }
    //   } else {
    //     this.$message({
    //       type: 'warning',
    //       message: '请上传附件！'
    //     })
    //   }
    // },
    importfile(obj) {
      const that = this
      const reader = new FileReader()
      reader.readAsArrayBuffer(obj)
      reader.onload = function () {
        const buffer = reader.result
        const bytes = new Uint8Array(buffer)
        const length = bytes.byteLength
        let binary = ''
        for (let i = 0; i < length; i++) {
          binary += String.fromCharCode(bytes[i])
        }
        const XLSX = require('xlsx')
        const wb = XLSX.read(binary, {
          type: 'binary'
        })
        const outdata = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]])
        let arr = []
        that.tableData = []
        outdata.map(item => {
          if (item[that.excelCellName]) {
            arr = arr.concat(item[that.excelCellName].split(" "))
          }
        })
        let obj = {}
        arr.forEach(item => {
          if (obj[item]) {
            obj[item] += 1
          } else {
            obj[item] = 1
          }
        })
        that.tableData = Object.keys(obj).map((key) => {
          return {name: key, cnt: obj[key]}
        })
      }
    },
    dealFile() {
      const that = this
      this.fileList.map((item) => {
        const reader = new FileReader()
        reader.readAsArrayBuffer(item.raw)
        reader.onload = function () {
          const buffer = reader.result
          const bytes = new Uint8Array(buffer)
          const length = bytes.byteLength
          let binary = ''
          for (let i = 0; i < length; i++) {
            binary += String.fromCharCode(bytes[i])
          }
          const XLSX = require('xlsx')
          const wb = XLSX.read(binary, {
            type: 'binary'
          })
          const outdata = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]])
          that.excelArr.push(...outdata)
        }
      })
    }
  }
}
</script>

<style scoped>
.ml {
  margin-left: 20px;
}
</style>