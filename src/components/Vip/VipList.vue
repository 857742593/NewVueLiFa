<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>会员中心</el-breadcrumb-item>
      <el-breadcrumb-item>会员列表</el-breadcrumb-item>
      <el-breadcrumb-item>会员信息管理</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card class="box-card">
      <el-form :inline="true">
        <el-form-item>
          <el-input placeholder="会员卡卡号" v-model="search" clearable></el-input>
        </el-form-item>
        <el-button type="primary" style="width: 100px" @click="getVipById()">查询</el-button>
        <el-button type="primary" style="width: 100px" @click="getAllList()">显示全部</el-button>
      </el-form>

      <el-table
        ref="multipleTable"
        :data="tableData.slice((currentPage-1)*pageSize,currentPage*pageSize)"
        highlight-current-row
        style="width: 100%;margin-top:-10px;">
        <el-table-column label="卡号">
          <template slot-scope="scope">
            <span>{{ scope.row.vipId }}</span>
          </template>
        </el-table-column>
        <el-table-column label="会员姓名">
          <template slot-scope="scope">
            <span>{{ scope.row.vipName }}</span>
          </template>
        </el-table-column>
        <el-table-column label="电话号码">
          <template slot-scope="scope">
            <span>{{ scope.row.phone }}</span>
          </template>
        </el-table-column>
        <el-table-column label="卡类型">
          <template slot-scope="scope">
            <span v-if="scope.row.cardId === 'T001'">9折卡</span>
            <span v-if="scope.row.cardId === 'T002'">8.5折卡</span>
            <span v-if="scope.row.cardId === 'T003'">8折卡</span>
            <span v-if="scope.row.cardId === 'T004'">7.5折卡</span>
            <span v-if="scope.row.cardId === 'T005'">7折卡</span>
          </template>
        </el-table-column>
        <el-table-column label="卡内余额">
          <template slot-scope="scope">
            <span>{{ scope.row.cardMoney }}</span>
          </template>
        </el-table-column>
        <el-table-column label="消费金额">
          <template slot-scope="scope">
            <span>{{ scope.row.consumeMoney }}</span>
          </template>
        </el-table-column>
        <el-table-column label="开卡时间">
          <template slot-scope="scope">
            <span>{{ scope.row.cardTime }}</span>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button type="primary" size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
            <el-button type="danger" size="small" @click="deleteVip(scope.$index, scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="80px" class="demo-ruleForm" size="medium">
        <el-dialog title="编辑" :append-to-body='true' :visible.sync="dialogUpdate" :before-close="handleClose">
          <el-form-item label="卡号">
            <el-input v-model="ruleForm.vipId" disabled="disabled"></el-input>
          </el-form-item>
          <el-form-item label="会员姓名">
            <el-input v-model="ruleForm.vipName"></el-input>
          </el-form-item>
          <el-form-item label="电话号码">
            <el-input v-model="ruleForm.phone"></el-input>
          </el-form-item>
          <el-form-item label="卡类型">
            <el-select v-model="ruleForm.cardId" placeholder="请选择会员卡类型" prop="cardType">
              <el-option label="9折卡" value="T001"></el-option>
              <el-option label="8.5折卡" value="T002"></el-option>
              <el-option label="8折卡" value="T003"></el-option>
              <el-option label="7.5折卡" value="T004"></el-option>
              <el-option label="7折卡" value="T005"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="卡内余额">
            <el-input v-model="ruleForm.cardMoney" disabled="disabled"></el-input>
          </el-form-item>
          <el-form-item label="消费金额">
            <el-input v-model="ruleForm.consumeMoney" disabled="disabled"></el-input>
          </el-form-item>
          <el-form-item label="开卡时间">
            <el-input v-model="ruleForm.cardTime" disabled="disabled"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button @click="updateVip()" type="primary" size="medium">确 定</el-button>
            <el-button @click="emptyUserData()" size="medium">取 消</el-button>
          </el-form-item>
        </el-dialog>
      </el-form>

      <el-pagination
        align="center"
        :current-page.sync="currentPage"
        :page-size.sync="pageSize"
        layout="total, prev, pager, next, jumper"
        :total="total"
        background>
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
    export default {
        data() {
            return {
                ruleForm: {
                    vipId: null,//卡号
                    vipName: null,//会员姓名
                    phone: null,//电话号码
                    cardId: null,//卡类型
                    cardMoney: null,//卡内余额
                    consumeMoney: null,//消费金额
                    cardTime: null
                },
                rules: {},
                tableData: [],
                dialogUpdate: false,
                disablePage: false,
                multipleSelection: [],
                pageSize: 10,     //每页多少条记录
                currentPage: 1,  //当前页
                total: 0,       //总记录数
                disablePage: false,
                search: ''
            }
        },

        created() {
            this.getAllList()
        },
        formatDate(time) {
            console.log(time)
            const year = time.getFullYear();
            const month = (time.getMonth() + 1).toString().padStart(2, 0);
            const data = time.getDate().toString().padStart(2, 0);
            return year + '-' + month + '-' + data
        },

        methods: {
            handleSizeChange(val) {
                console.log(`每页 ${val} 条`);
            },
            handleCurrentChange(val) {
                console.log(`当前页: ${val}`);
            },
            /*
            * 点击编辑按钮
            */
            handleEdit(index, row) {
                this.dialogUpdate = true;   //打开表单
                this.ruleForm = Object.assign({}, row, index); //这句是关键！！！
            },
            /*
            * 表单右上角的 X 按钮
            */
            handleClose(done) {
                this.dialogUpdate = false
            },

            /**
             * 清空绑定数据
             */
            emptyUserData() {
                this.dialogUpdate = false;
                this.ruleForm = {
                    vipId: null,//卡号
                    vipName: null,//会员姓名
                    phone: null,//电话号码
                    cardId: null,//卡类型
                    cardMoney: null,//卡内余额
                    consumeMoney: null//消费金额
                };
            },

            /*
              显示所有会员信息
            */
            getAllList() {
                this.$axios({
                    method: 'get',
                    url: 'http://localhost:8081/lifamanagement/vip/getAllList',
                }).then(response => {
                    this.tableData = response.data
                    this.total = this.tableData.length
                    console.log(response.data)
                }).catch(error => {
                    console.log(error);
                })
            },
            /*
              根据卡号查询会员信息
            */
            getVipById() {
                let postData = this.qs.stringify({
                    vipId: this.search//卡号
                })
                console.log(typeof this.search)
                this.$axios({
                    method: 'post',
                    url: 'http://localhost:8081/lifamanagement/vip/getVipById',
                    data: postData
                }).then(response => {
                    this.tableData = response.data
                    this.total = this.tableData.length
                    this.$message({
                        type: 'success',
                        message: '查询完成!'
                    });
                }).catch(error => {
                    console.log(error)
                })
            },
            /*
              修改会员信息
             */
            updateVip() {
                let postData = this.qs.stringify({
                    vipId: this.ruleForm.vipId,//卡号
                    vipName: this.ruleForm.vipName,//会员姓名
                    phone: this.ruleForm.phone,//电话号码
                    cardId: this.ruleForm.cardId,//卡类型
                });
                this.$axios({
                    method: 'post',
                    url: 'http://localhost:8081/lifamanagement/vip/updateVipList',
                    data: postData
                }).then(response => {
                    this.dialogUpdate = false   //关闭表单
                    this.$message({
                        type: 'success',
                        message: '修改成功!'
                    });
                    this.getAllList()
                }).catch(error => {
                    console.log(error);
                });
            },
            /**
             * 根据 vipId 删除会员（销卡）
             * @param index
             * @param row
             */
            deleteVip(index, row) {
                this.$confirm('删除操作, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    let postData = this.qs.stringify({
                        vipId: row.vipId
                    });
                    this.$axios({
                        method: 'post',
                        url: 'http://localhost:8081/lifamanagement/vip/deleteVip',
                        data: postData
                    }).then(response => {
                        this.$message({
                            type: 'success',
                            message: '删除成功!'
                        });
                        this.getAllList()
                    }).catch(error => {
                        console.log(error);
                    });
                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
            }
        }
    }
</script>
<style scoped>
</style>
