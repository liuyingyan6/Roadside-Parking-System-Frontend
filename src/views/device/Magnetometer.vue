<template>
  <div>
    <el-row :gutter="12">
      <el-card shadow="always">
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
          <el-breadcrumb-item>设备管理</el-breadcrumb-item>
          <el-breadcrumb-item>地磁管理</el-breadcrumb-item>
        </el-breadcrumb>
      </el-card>
    </el-row>
    <el-row>
      <el-card shadow="always">
        <el-row>
          <el-form :inline="true">
            <el-form-item label="地磁名称">
              <el-input v-model="nameKeyword" placeholder="地磁名称"></el-input>
            </el-form-item>
            <el-form-item label="所属路段">
              <el-input v-model="roadNameKeyword" placeholder="所属路段"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" icon="el-icon-search" @click="getPage">查询</el-button>
              <el-button type="primary" icon="el-icon-search" @click="resetKeyword">重置</el-button>
            </el-form-item>
          </el-form>
          <div>
            <el-button type="primary" plain @click="showDialog()">添加</el-button>
            <el-button type="primary" plain icon="el-icon-s-flag" @click="exportExcel">批量导出</el-button>
            <el-button type="primary" plain icon="el-icon-s-flag" @click="dialogImportVisible = true">批量导入
            </el-button>
          </div>
        </el-row><!--搜索-->

        <!-- 表格 data：要绑定的数据  handleSelectionChange 多选的方法 -->
        <el-table
            :data="tableData"
            tooltip-effect="dark"
            style="width: 100%; text-align: center; font-size: 18px"
            @selection-change="handleSelectionChange">
          <el-table-column
              type="selection"
              width="55">
          </el-table-column> <!-- 复选框 -->
          <el-table-column
              prop="id"
              label="地磁编号"
              width="120">
          </el-table-column> <!-- 地磁编号 -->
          <el-table-column
              prop="name"
              label="地磁名称"
              width="120">
          </el-table-column> <!-- 地磁名称 -->
          <el-table-column
              prop="roadName"
              label="所属路段"
              width="120">
          </el-table-column> <!-- 所属路段 -->
          <el-table-column
              prop="parkingName"
              label="绑定泊位"
              width="120">
          </el-table-column> <!-- 绑定泊位 -->
          <el-table-column
              prop="parkingStatus"
              label="泊位情况"
              width="120">
            <template slot-scope="scope">
              <span>
                {{ scope.row.parkingStatus === 0 ? '有车' : scope.row.parkingStatus === 1 ? '无车' : '未激活' }}
              </span>
            </template>
          </el-table-column> <!-- 泊位情况 -->
          <el-table-column
              prop="status"
              label="地磁状态"
              width="120"
          >
            <template slot-scope="scope">
              <span>
                {{ scope.row.status === 0 ? '在线' : scope.row.status === 1 ? '离线' : '未激活' }}
              </span>
            </template>
          </el-table-column> <!-- 地磁状态 -->
          <el-table-column
              prop="createTime"
              label="创建时间"
              width="120">
          </el-table-column> <!-- 创建时间 -->
          <el-table-column
              prop="updateTime"
              label="状态更新时间"
              width="120">
          </el-table-column> <!-- 状态更新时间 -->
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-button-group>
                <el-button
                    type="success" icon="el-icon-edit" plain
                    @click="handleEdit(scope.row)">编辑
                </el-button>
                <el-button
                    type="danger" icon="el-icon-circle-close" plain
                    @click="handleDelete(scope.row)">删除
                </el-button>
                <el-button
                    type="danger" icon="el-icon-circle-close" plain
                    @click="toLogPage(scope.row.id)">地磁日志
                </el-button> <!-- 地磁日志 -->
              </el-button-group>
            </template>
          </el-table-column>
        </el-table>

        <!-- 分页组件 total总条数 page-size：每页大写 current-page 当前页码 page-sizes：下拉列表 page-size：页面大小 -->
        <el-pagination style="margin-top: 15px"
                       background
                       @size-change="handleSizeChange"
                       @current-change="handleCurrentChange"
                       :current-page="pageNum"
                       :page-sizes="[3, 5, 10, 20]"
                       :page-size="pageSize"
                       layout="total, sizes, prev, pager, next, jumper"
                       :total="total">
        </el-pagination>
      </el-card>
    </el-row>


    <el-dialog title="地磁管理" :visible.sync="dialogFormVisible" width="500px">
      <!--添加表单-->
      <el-form label-width="120px" :model="formData" ref="form">
        <el-form-item label="地磁名字" :label-width="formLabelWidth" prop="name">
          <el-input v-model="formData.name" autocomplete="off" style="width: 300px"></el-input>
        </el-form-item>
        <el-form-item label="绑定泊位" :label-width="formLabelWidth" prop="parkingId">
          <el-select v-model="formData.parkingId" style="width: 300px">
            <el-option v-for="parking in parkings" :label="parking.name"
                       :value="parking.id"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="状态" :label-width="formLabelWidth" prop="status" value-key="value">
          <el-select v-model="formData.status" style="width: 300px">
            <el-option label="在线" :value="0"></el-option>
            <el-option label="离线" :value="1"></el-option>
            <el-option label="未激活" :value="2"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer" style="text-align: center">
        <el-button type="primary" @click="saveOrUpdate()">确 定</el-button>
        <el-button type="danger" @click="resetForm()">返 回</el-button>
      </div>
    </el-dialog> <!-- 添加和修改的对话框 -->

    <el-dialog title="批量导入" :visible.sync="dialogImportVisible" width="500px" @close="getPage()">
      <div>* 第一步：下载模版</div>
      <div>
        <el-button type="primary" @click="downloadTemplate()">下载</el-button>
      </div>
      <div>* 第二步：填入信息</div>
      <div>* 第三步：上传文件</div>
      <div>
        <el-upload
            class="upload-demo"
            ref="upload"
            action="#"
            accept="xlsx"
            :show-file-list="false"
            :limit="1"
            :http-request="selectFile">
          <el-button slot="trigger" size="small" type="primary">
            选取文件
          </el-button>

          <el-button size="small" type="success" @click="submitUpload" :disabled="disableUpload">
            上传
          </el-button>
          <div slot="tip" class="el-upload__tip">只能上传xlsx文件，且不超过500kb</div>
        </el-upload>
      </div>
    </el-dialog> <!-- 上传的对话框 -->
  </div>
</template>
<script>
import axios from 'axios';

export default {
  data() {
    return {
      nameKeyword: '',
      roadNameKeyword: '',
      form: {},
      formData: {},
      dialogFormVisible: false,
      formLabelWidth: '120px',
      parkings: [],
      //分页查询提交的参数
      pageNum: 1,
      pageSize: 5,
      tableData: [],
      total: 0,
      multipleSelection: [],
      dialogImportVisible: false,
      disableUpload: true,
      uploadFile: {},
    }
  },
  methods: {
    handleSelectionChange(val) {
      this.multipleSelection = val;
    }, //复选框的方法
    exportExcel() {
      const token = localStorage.getItem('accessToken');
      const xhr = new XMLHttpRequest();
      xhr.open("GET", "http://localhost:9090/magnetometer/export");
      xhr.setRequestHeader("Authorization", "Bearer " + token);
      xhr.responseType = 'blob'; // 设置响应类型为blob
      xhr.onload = function () {
        if (xhr.status === 200) {
          const blob = new Blob([xhr.response]); // 创建blob对象
          const link = document.createElement('a');
          link.href = window.URL.createObjectURL(blob); // 创建下载链接
          link.download = '地磁信息.xlsx'; // 设置下载的文件名
          link.click(); // 模拟点击下载链接
          window.URL.revokeObjectURL(link.href); // 释放URL对象
        }
      };
      xhr.send();
    }, //导出按钮
    showDialog() {
      this.getParkings();
      this.dialogFormVisible = true;
    }, //显示添加/修改的对话框
    resetForm() {
      this.$refs.form.resetFields();
      this.dialogFormVisible = false;
    },// 取消显示的对话框
    handleSizeChange(val) {
      this.pageSize = val;
      this.getPage();
    }, // 切换页大小
    handleCurrentChange(val) {
      this.pageNum = val;
      this.getPage();
    }, // 切换页数
    getPage() {
      axios.get("/magnetometer/getPageByKeyword", {
        params: {
          pageNum: this.pageNum,
          pageSize: this.pageSize,
          magnetometerName: this.nameKeyword,
          roadName: this.roadNameKeyword
        }
      })
          .then(resp => {
            console.log(resp);
            this.tableData = resp.data.records;
            this.total = resp.data.total
          });
    }, // 分页查询
    toLogPage(id) {
      this.$router.push('/magnetometerLog/' + id);
    }, // 跳转到地磁日志
    getParkings() {
      axios.get('/parking/list')
          .then(resp => {
            this.parkings = resp.data;
          })
    },
    saveOrUpdate() {
      if (this.formData.id) {
        axios.put("/magnetometer/update", this.formData)
            .then(resp => {
              this.$message.success('修改成功～')
            }).finally(() => {
          this.dialogFormVisible = false;
          this.formData = {};
          this.getPage();
        })
      } else {
        //没有就是添加
        axios.post("/magnetometer/add", this.formData)
            .then(resp => {
              this.$message.success('添加成功～')
            })
            .catch(error => {
              this.$message.error(error);
            })
            .finally(() => {
              this.dialogFormVisible = false;
              this.formData = {};
              this.getPage();
            })
      }
    }, // dialog确定按钮
    handleEdit(row) {
      console.log(row);
      // 转换防止该行和fromData指向同一地址
      let obj = {};
      Object.assign(obj, row);
      this.formData = obj;
      this.showDialog();

    }, // 编辑
    handleDelete(row) {
      this.$confirm(`此操作将删除${row.name}, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        //调用后台服务器删除
        axios.delete(`/magnetometer/delete?id=${row.id}`)
            .then(resp => {
              this.$message.success('删除成功～');
            })
            .catch(error => {
              this.$message.error(error);
            })
            .finally(() => {
              this.getPage();
            });
      })
    }, // 删除
    downloadTemplate() {
      const token = localStorage.getItem('accessToken');
      const xhr = new XMLHttpRequest();
      xhr.open("GET", "http://localhost:9090/magnetometer/exportTemplate");
      xhr.setRequestHeader("Authorization", "Bearer " + token);
      xhr.responseType = 'blob'; // 设置响应类型为blob
      xhr.onload = function () {
        if (xhr.status === 200) {
          const blob = new Blob([xhr.response]); // 创建blob对象
          const link = document.createElement('a');
          link.href = window.URL.createObjectURL(blob); // 创建下载链接
          link.download = '地磁导入模版.xlsx'; // 设置下载的文件名
          link.click(); // 模拟点击下载链接
          window.URL.revokeObjectURL(link.href); // 释放URL对象
        }
      };
      xhr.send();
    }, // 导出模版
    selectFile(item) {
      this.uploadFile = item.file;
      this.disableUpload = false;
    },
    submitUpload() {
      let formData = new FormData();
      formData.append('file', this.uploadFile);
      axios.post('/magnetometer/fileAdd', formData)
          .then(resp => {
            this.$message.success('成功上传～');
          })
          .catch(error => {
            this.$message.error(error);
          })
          .finally(() => {

          });
    },
    resetKeyword() {
      this.nameKeyword = '';
      this.roadNameKeyword = '';
    }
  },
  created() {
    this.getPage();
  }
};
</script>
<style lang="less" scoped>

</style>
