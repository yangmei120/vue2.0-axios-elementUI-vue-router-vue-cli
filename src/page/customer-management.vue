<template>
  <div>
    <menu-left routeIndex="5"></menu-left>
    <div class="cus-container">
      <div class="header">
        <div class="number">
          <span>手机号码</span>
          <input type="text" v-model="phoneNum" onkeyup="value=value.replace(/[^\d]/g,'')" maxlength="11">
        </div>
        <div class="screen">
          <span>购物次数</span>
          <el-select v-model="value" placeholder="不限">
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
        </div>
        <div class="search" @click="search(0)">
          <el-button
          size="small"
          type="success"
          >搜索</el-button>
        </div>
      </div>
      <div class="table">
        <template>
          <el-table
            :data="tableData"
            border>
            <el-table-column
              prop="name"
              label="姓名"
              width="200">
            </el-table-column>
            <el-table-column
              prop="mobile"
              label="手机号码">
            </el-table-column>
            <el-table-column
              prop="nick_name"
              label="微信昵称">
            </el-table-column>
            <el-table-column
              prop="order_count"
              label="拥有订单数"
              width="120">
              <template slot-scope="scope">
                <div>{{scope.row.order_count ? scope.row.order_count : 0}}</div>
              </template>
            </el-table-column>
            <el-table-column
              prop="address"
              label="操作">
              <template slot-scope="scope">
                <el-button type="text" @click="editDetails(scope.$index)">编辑</el-button>
                <el-button type="text" @click="orderDetails(scope.$index)" class="orange">订单详情</el-button>
              </template>
            </el-table-column>
          </el-table>
        </template>
        <div class="padding-top-20">
          <el-pagination
            background
            v-if="tableData.length"
            prev-text="<上一页"
            next-text="下一页>"
            :page-size="15"
            :current-page="pages"
            @current-change="changePage"
            layout="prev, pager, next"
            :total="totalPage * 15">
          </el-pagination>
        </div>
        <el-dialog
          title="编辑"
          :show-close="false"
          :visible.sync="dialogVisible"
          width="30%">
          <hr style="color: #EFEFEF;border: 1px solid #EFEFEF;border-bottom: 0;"/>
          <div>
            <span class="dialog_name">姓名</span><input type="text" maxlength="20" v-model="name" clearable>
          </div>
          <div slot="footer" class="dialog-footer">
            <el-button @click="saveInfo" type="success" style="border: 0;color: #ffffff;" size="small">保存</el-button>
            <el-button @click="dialogVisible = false" size="small">返回</el-button>
          </div>
        </el-dialog>
      </div>
    </div>
  </div>
</template>

<script>
import {user, userEditor} from '@/axios/api'
import menuLeft from '@/components/menu-left'
export default {
  data () {
    return {
      name: '',
      totalPage: 0,
      dialogVisible: false,
      phoneNum: '',
      wechatNum: '',
      value: '',
      pages: 1,
      options: [
        {
          value: '',
          label: '不限'
        },
        {
          value: '1',
          label: '1+'
        }, {
          value: '5',
          label: '5+'
        }, {
          value: '10',
          label: '10+'
        }, {
          value: '20',
          label: '20+'
        }, {
          value: '50',
          label: '50+'
        }, {
          value: '100',
          label: '100+'
        }
      ],
      tableData: [],
      // flag： 标记搜索状态
      flag: false
    }
  },
  mounted () {
    user().then(
      res => {
        this.tableData = res.data
        this.totalPage = parseInt(res.headers.page_count)
      }
    )
  },
  methods: {
    // 点击搜索
    search (value) {
      this.flag = true
      let params = {}
      const reg = /^[1][3,4,5,7,8][0-9]{9}$/
      if (this.phoneNum) {
        if (reg.test(this.phoneNum)) {
          params.mobile = this.phoneNum
        } else {
          this.$message.error('这不是一个正确的手机号码')
          return false
        }
      }
      params.order_count = this.value
      params.page = value
      user(params).then(
        res => {
          this.tableData = res.data
          this.totalPage = parseInt(res.headers.page_count)
          this.pages = 1
        }
      )
    },
    // 编辑列表
    editDetails (detail) {
      // 打开模态框
      this.dialogVisible = true
      this.detail = detail
    },
    // 保存姓名 关闭模态框
    saveInfo () {
      let that = this
      userEditor({
        name: this.name
      }, this.tableData[this.detail].id).then(
        res => {
          // console.log(res)
          if (res.status === 200) {
            that.tableData[that.detail].name = that.name
            that.dialogVisible = false
            that.name = ''
            this.$message({
              message: '修改成功',
              type: 'success'
            })
          } else {
            this.$message.error('修改失败')
          }
        }
      ).catch(() => {
        this.$message({
          message: '修改失败',
          type: 'error'
        })
      })
    },
    orderDetails (link) {
      this.$router.push({name: 'customerOrder', params: {id: this.tableData[link].id}})
      // this.$router.push({name: 'customerOrder', params: {id: 1}})
    },
    // 点击分页
    changePage (val) {
      let params = {}
      if (this.flag) {
        const reg = /^[1][3,4,5,7,8][0-9]{9}$/
        if (this.phoneNum) {
          if (reg.test(this.phoneNum)) {
            params.mobile = this.phoneNum
          } else {
            this.$message.error('这不是一个正确的手机号码')
            return false
          }
        }
        params.order_count = this.value
      }
      params.page = val - 1
      user(params).then(
        res => {
          this.tableData = res.data
          this.totalPage = parseInt(res.headers.page_count)
        })
    }
  },
  components: {
    menuLeft
  }
}
</script>

<style scoped lang="less">
.cus-container{
  min-width: 1000px;
  margin: 0 20px 0 200px;
  padding-top: 20px;
  position: relative;
  .header{
    min-width: 1000px;
    background: #ffffff;
    padding: 20px 0;
    overflow: hidden;
    span{
      font-size: 12px;
      color: #999999;
      margin-left: 20px;
      margin-right: 10px;
    }
    .number{
      height: 30px;
      float: left;
      }
      input{
        width: 230px;
        height: 30px;
        border: 1px solid #D5D5D5;
        border-radius: 2px;
        padding-left: 10px;
    }
    .screen{
      height: 30px;
      float: left;
    }
    .search{
      width: 80px;
      height: 30px;
      float: left;
      margin-left: 30px;
    }
  }
  .table{
    min-width: 1000px;
    margin-top: 20px;
    padding: 20px;
    background: #ffffff;
    min-height: 281px;
    overflow: hidden;
  }
  .el-button--small {
    width: 80px;
    height: 30px;
    padding: 0;
  }
}
</style>
<style lang="less">
  .cus-container{
    .el-table{
      font-size: 12px;
      color: #666666;
      .el-button{
        font-size: 12px;
      }
    }
    .el-input__inner{
      width: 128px;
      height: 30px;
      color: #333333;
      border-radius: 0;
    }
    .el-table__header-wrapper thead{
      color: #333333;
    }
    .el-table::before{
      background-color: #D5D5D5;
    }
    .el-table--border::after{
      background-color: #D5D5D5;
    }
    .el-table__header-wrapper thead tr th{
      background: #EFEFEF;
    }
    .el-table__body-wrapper,.el-table__header-wrapper tr th div{
      text-align: center;
    }
    .el-table--group, .el-table--border {
      border: 1px solid #D5D5D5;
    }
    .el-table--border {
      border-right: none;
      border-bottom: none;
    }
    .el-table th.is-leaf, .el-table td {
      border-bottom: 1px solid #D5D5D5;
    }
    .el-table--border th, .el-table--border td {
      border-right: 1px solid #D5D5D5;
    }
    .el-table__body-wrapper tr td .el-button--text{
      border: 1px solid #63A4FF;
      padding: 4px 8px;
    }
    .el-table__body-wrapper tr td .el-button--text:last-child{
      border: 1px solid @orange;
      padding: 4px 8px;
    }
    .el-dialog .el-dialog__header .el-dialog__title {
      font-size: 14px;
      color: #333333;
      font-weight: 600;
    }
    .el-dialog .el-dialog__body div{
      padding-top: 20px;
    }
    .el-dialog .el-dialog__body .dialog_name{
      font-size: 12px;
      color: #999999;
    }
    .el-dialog .el-dialog__body input{
      width: 240px;
      height: 30px;
      padding-left: 10px;
      border: 1px solid #D5D5D5;
      border-radius: 2px;
      font-size: 12px;
      color: #333333;
      line-height: 30px;
      margin-left: 10px;
    }
    .el-dialog .el-dialog__footer button{
      width: 80px;
      height: 30px;
      border: 1px solid #333333;
      border-radius: 2px;
      color: #333333;
    }
    .el-button.is-plain:hover, .el-button.is-plain:focus {
      background: #DE5B67;
      border-color: #DE5B67;
      color: #ffffff;
    }
  }
</style>
