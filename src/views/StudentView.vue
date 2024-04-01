<template>
  <div>
    <!-- 搜索框和按钮 -->
    <div>
      <el-input v-model="searchKeyword" style="width: 200px; margin-right: 10px;" placeholder="请输入学生姓名"></el-input>
      <el-button @click="searchStudents" type="warning">搜索</el-button>
      <el-button @click="showAddDialog" type="primary">新增</el-button>
    </div>

    <!-- 学生信息表格 -->
    <div>
      <el-table :data="students" style="width: 100%; margin-top: 15px;">
        <el-table-column prop="studentId" label="学号" width="80"></el-table-column>
        <el-table-column prop="name" label="姓名" width="80"></el-table-column>
        <el-table-column prop="sex" label="性别" width="80"></el-table-column>
        <el-table-column prop="dateOfBirth" label="出生日期" width="120"></el-table-column>
        <el-table-column prop="nativePlace" label="籍贯" width="80"></el-table-column>
        <el-table-column prop="mobilePhone" label="手机号" width="120"></el-table-column>
        <el-table-column prop="departmentId" label="学院编号" width="80"></el-table-column>
        <el-table-column prop="status" label="状态"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button @click="editStudent(scope.row)" type="primary" size="mini">编辑</el-button>
            <el-button @click="deleteStudent(scope.row)" type="danger" size="mini">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!-- 分页 -->
    <div class="block" style="margin-top: 10px;">
      <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :page-sizes="[10, 20, 30, 40]"
          :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
      </el-pagination>
    </div>

    <!-- 新增/编辑学生信息的对话框 -->
    <el-dialog :title="dialogTitle" :visible.sync="dialogVisible">
      <el-form :model="formData" ref="studentForm" label-width="80px">
        <el-form-item label="姓名" prop="name">
          <el-input v-model="formData.name"></el-input>
        </el-form-item>
        <el-form-item label="性别" prop="sex">
          <el-radio-group v-model="formData.sex">
            <el-radio label="男">男</el-radio>
            <el-radio label="女">女</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="出生日期" prop="dateOfBirth">
          <el-date-picker v-model="formData.dateOfBirth" type="date"></el-date-picker>
        </el-form-item>
        <el-form-item label="籍贯" prop="nativePlace">
          <el-input v-model="formData.nativePlace"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="saveStudent">确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import request from '@/utils/request';

export default {
  name: 'StudentView',
  data() {
    return {
      students: [], // 学生信息列表
      total: 0, // 总记录数
      pageSize: 10, // 每页显示的记录数
      currentPage: 1, // 当前页码
      searchKeyword: '', // 搜索关键字
      dialogVisible: false, // 新增/编辑对话框可见性
      dialogTitle: '', // 对话框标题
      formData: { // 表单数据
        name: '',
        sex: '',
        dateOfBirth: '',
        nativePlace: ''
      }
    };
  },
  created() {
    this.loadStudents();
  },
  methods: {
    // 加载学生信息列表
    loadStudents() {
      const params = {
        keyword: this.searchKeyword,
        page: this.currentPage,
        size: this.pageSize
      };
      request.get('/student', {params}).then(res => {
        this.students = res.data;
        this.total = res.total;
      });
    },
    // 搜索学生信息
    searchStudents() {
      this.currentPage = 1; // 重新搜索时，重置页码为第一页
      this.loadStudents();
    },
    // 分页变化时的处理函数
    handleSizeChange(size) {
      this.pageSize = size;
      this.loadStudents();
    },
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage;
      this.loadStudents();
    },
    // 显示新增学生信息对话框
    showAddDialog() {
      this.dialogTitle = '新增学生信息';
      this.dialogVisible = true;
      // 清空表单数据
      this.formData = {
        name: '',
        sex: '男',
        dateOfBirth: '',
        nativePlace: ''
      };
    },
    // 编辑学生信息
    editStudent(row) {
      this.dialogTitle = '编辑学生信息';
      this.dialogVisible = true;
      // 设置表单数据
      this.formData = {...row};
    },
    // 保存学生信息
    saveStudent() {
      // 提交表单数据到后端保存
      request.post('/student/save', this.formData).then(() => {
        this.$message.success('保存成功');
        this.dialogVisible = false;
        this.loadStudents(); // 保存成功后重新加载学生信息列表
      });
    },
    // 删除学生信息
    deleteStudent(row) {
      // 向后端发送删除请求
      request.delete(`/student/delete/${row.id}`).then(() => {
        this.$message.success('删除成功');
        this.loadStudents(); // 删除成功后重新加载学生信息列表
      });
    }
  }
};
</script>

<style scoped>
/* 在这里添加样式 */
</style>
