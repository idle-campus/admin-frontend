<template>
    <!-- 功能区 -->
      <div>
        <!-- 搜索功能框 -->
        <div style="margin: 10px 0">
          <el-input style="width: 200px" placeholder="请输入学号" suffix-icon="el-icon-search" v-model="username"></el-input>
          <el-input style="width: 200px" placeholder="请输入学校" suffix-icon="el-icon-message" class="ml-5" v-model="school"></el-input>
          <el-input style="width: 200px" placeholder="请输入年级" suffix-icon="el-icon-position" class="ml-5" v-model="grade"></el-input>
          <el-button class="ml-5" type="primary" @click="load">搜索</el-button>
          <el-button type="warning" @click="reset">重置</el-button>
        </div>
        <!-- 新增用户按钮 -->
        <div style="margin: 10px 0">
          <el-button type="primary" @click="handleAdd">新增 <i class="el-icon-circle-plus-outline"></i></el-button>
          <!-- 批量删除功能 -->
          <el-popconfirm
              class="ml-5"
              confirm-button-text='确定'
              cancel-button-text='我再想想'
              icon="el-icon-info"
              icon-color="red"
              title="您确定批量删除这些数据吗？"
              @confirm="delBatch"
          >
            <el-button type="danger" slot="reference">批量删除 <i class="el-icon-remove-outline"></i></el-button>
          </el-popconfirm>
          <!-- 导入导出数据功能按钮 -->
          <el-upload :action="'http://' + serverIp + ':8282/student/import'" :show-file-list="false" accept="xlsx" :on-success="handleExcelImportSuccess" style="display: inline-block">
            <el-button type="primary" class="ml-5">导入 <i class="el-icon-bottom"></i></el-button>
          </el-upload>
          <el-button type="primary" @click="exp" class="ml-5">导出 <i class="el-icon-top"></i></el-button>
        </div>
        <!-- 表单显示 -->
        <el-table :data="tableData" border stripe :header-cell-class-name="'headerBg'"  @selection-change="handleSelectionChange">
          <el-table-column type="selection" width="55"></el-table-column>
          <el-table-column prop="id" label="ID" width="50"></el-table-column>
          <el-table-column prop="username" label="学号" width="100"></el-table-column>
          <el-table-column prop="nickname" label="昵称" width="120"></el-table-column>
          <el-table-column prop="email" label="邮箱"></el-table-column>
          <el-table-column prop="phone" label="电话" width="100"></el-table-column>
          <el-table-column prop="qq" label="QQ账号"></el-table-column>
          <el-table-column prop="wechat" label="微信账号"></el-table-column>
          <el-table-column prop="school" label="学校"></el-table-column>
          <el-table-column prop="academy" label="学院"></el-table-column>
          <el-table-column prop="grade" label="年级"></el-table-column>
          <el-table-column label="操作"  width="200" align="center">
            <template slot-scope="scope">
              <el-button type="success" @click="handleEdit(scope.row)">编辑 <i class="el-icon-edit"></i></el-button>
              <el-popconfirm
                  class="ml-5"
                  confirm-button-text='确定'
                  cancel-button-text='我再想想'
                  icon="el-icon-info"
                  icon-color="red"
                  title="您确定删除吗？"
                  @confirm="del(scope.row.id)"
              >
                <el-button type="danger" slot="reference">删除 <i class="el-icon-remove-outline"></i></el-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <div style="padding: 10px 0">
          <el-pagination
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="pageNum"
              :page-sizes="[2, 5, 10, 20]"
              :page-size="pageSize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="total">
          </el-pagination>
        </div>
    
        <el-dialog title="用户信息" :visible.sync="dialogFormVisible" width="30%" >
          <el-form label-width="80px" size="small">
            
              <el-form-item label="学号">
              <el-input v-model="form.username"  autocomplete="off"></el-input>
            </el-form-item>
          
            
            <el-form-item label="昵称">
              <el-input v-model="form.nickname" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="邮箱">
              <el-input v-model="form.email" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="电话">
              <el-input v-model="form.phone" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="QQ">
              <el-input v-model="form.qq" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="微信">
              <el-input v-model="form.wechat" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="学校">
              <el-input v-model="form.school" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="学院">
              <el-input v-model="form.academy" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="年级">
              <el-input v-model="form.grade" autocomplete="off"></el-input>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisible = false">取 消</el-button>
            <el-button type="primary" @click="save">确 定</el-button>
          </div>
        </el-dialog>
    

      </div>
    </template>
    
    <script>
    import {serverIp} from "../../public/config";
    
    export default {
      name: "Student",
      data() {
        return {
          serverIp: serverIp,
          tableData: [],
          total: 0,
          pageNum: 1,
          pageSize: 10,
          username: "",
          school: "",
          grade: "",
          form: {},
          dialogFormVisible: false,
          multipleSelection: [],
          // roles: [],
          // courses: [],
          // vis: false,
          // stuCourses: [],
          // stuVis: false
        }
      },
      created() {
        this.load()
      },
      methods: {
        load() {
          this.request.get("/student/page", {
            params: {
              pageNum: this.pageNum,
              pageSize: this.pageSize,
              username: this.username,
              school: this.school,
              grade: this.grade,
            }
          }).then(res => {
    
            this.tableData = res.data.records
            this.total = res.data.total
    
          })

      //     this.request.get("/user/role/").then(res => {
      //   this.teachers = res.data
      // })
    
          // this.request.get("/student").then(res => {
          //   this.roles = res.data
          // })
        },
        save() {
          this.request.post("/student", this.form).then(res => {
            if (res.code === '200') {
              this.$message.success("保存成功")
              this.dialogFormVisible = false
              this.load()
            } else {
              this.$message.error("保存失败")
            }
          })
        },
        handleAdd() {
          this.dialogFormVisible = true
          this.form = {}
        },
        handleEdit(row) {
          this.form = JSON.parse(JSON.stringify(row))
          this.dialogFormVisible = true
        },
        del(id) {
          this.request.delete("/student/" + id).then(res => {
            if (res.code === '200') {
              this.$message.success("删除成功")
              this.load()
            } else {
              this.$message.error("删除失败")
            }
          })
        },
        handleSelectionChange(val) {
          console.log(val)
          this.multipleSelection = val
        },
        delBatch() {
          let ids = this.multipleSelection.map(v => v.id)  // [{}, {}, {}] => [1,2,3]
          this.request.post("/student/del/batch", ids).then(res => {
            if (res.code === '200') {
              this.$message.success("批量删除成功")
              this.load()
            } else {
              this.$message.error("批量删除失败")
            }
          })
        },
        reset() {
          this.username = ""
          this.school = ""
          this.grade = ""
          this.load()
        },
        handleSizeChange(pageSize) {
          console.log(pageSize)
          this.pageSize = pageSize
          this.load()
        },
        handleCurrentChange(pageNum) {
          console.log(pageNum)
          this.pageNum = pageNum
          this.load()
        },
        exp() {
          window.open(`http://${serverIp}:8282/student/export`)
        },
        handleExcelImportSuccess() {
          this.$message.success("导入成功")
          this.load()
        }
      }
    }
    </script>
    
    
    <style>
    .headerBg {
      background: #eee!important;
    }
    </style>
    