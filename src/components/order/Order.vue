<template>
    <div>
        <!--面包屑导航区域-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
        <el-row>
            <el-col :span="8">
                <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getOrderList">
            <el-button slot="append" icon="el-icon-search" @click="getOrderList"></el-button>
          </el-input>
            </el-col>
        </el-row>

        <!-- 订单列表数据 -->
        <el-table :data="orderlist" border stripe>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="订单编号" prop="order_number"></el-table-column>
            <el-table-column label="订单价格" prop="goods_price"></el-table-column>
            <el-table-column label="是否付款" prop="pay_status">
                <template slot-scope="scope">
                    <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
                    <el-tag type="danger">未付款</el-tag>
                </template>
            </el-table-column>
            <el-table-column label="是否发货" prop="is_send">
                <template slot-scope="scope">
                    {{scope.row.is_send}}
                </template>
            </el-table-column>
            <el-table-column label="下单时间" prop="create_time">
                <template slot-scope="scope">
                    {{scope.row.create_time | dataFormat}}
                </template>
            </el-table-column>
            <el-table-column label="操作">
                <el-button type="primary" icon="el-icon-edit" @click="showDialog"></el-button>
                <el-button type="success" icon="el-icon-location" @click="showprogress"></el-button>
            </el-table-column>
        </el-table>

        <!--底部分页区域-->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5, 10, 15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>

<el-dialog
  title="修改地址"
  :visible.sync="dialogVisible"
  width="50%"
  @close = "addressFormDialogClosed">
  <el-form ref="addressFormRef" :model="addressForm"
  :rules="addressRules"  label-width="100px" >
  <el-form-item label="省市区/县" prop="address1">
    <el-cascader :options="cityData"
    v-model="addressForm.address1"
    :props="{ expandTrigger: 'hover' }"></el-cascader>
  </el-form-item>
  <el-form-item label="详细地址" prop="address2">
    <el-input v-model="addressForm.address2"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
  </span>
</el-dialog>

<!-- 展示物流信息的对话框 -->
<el-dialog
  title="物流信息"
  :visible.sync="progressVisible"
  width="50%">
  <!-- 物流信息时间线 -->
  <el-timeline >
    <el-timeline-item
      v-for="(activity, index) in progressInfo"
      :key="index"
      :timestamp="activity.time">
      {{activity.context}}
    </el-timeline-item>
  </el-timeline>
</el-dialog>
    </div>
</template>

<script>
import cityData from './citydata.js'
export default {
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderlist: [],
      dialogVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressRules: {
        address1: [{
          required: true,
          message: '请选择省市区县',
          tirgger: 'blur'
        }],
        address2: [{
          required: true,
          message: '请填写详细地址',
          tirgger: 'blur'
        }]
      },
      cityData,
      progressVisible: false,
      progressInfo: []

    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message('获取订单列表失败!')
      }
      this.total = res.data.total
      this.orderlist = res.data.goods
    },
    handleSizeChange (newsize) {
      this.pagesize = newsize
      this.getOrderList()
    },
    handleCurrentChange (newpage) {
      this.pagenum = newpage
      this.getOrderList()
    },
    // 展示修改对话框
    showDialog () {
      this.dialogVisible = true
    },
    addressFormDialogClosed () {
      this.$refs.addressFormRef.resetFields()
    },
    async showprogress () {
      const { data: res } = await this.$http.get('/kuaidi/804909574412544580')
      if (res.meta.status !== 200) {
        return this.$message.error('获取物流信息失败!')
      }
      this.progressInfo = res.data
      this.progressVisible = true
    }
  }
}
</script>

<style lang="less" scoped>

.el-cascader {
    width: 100%;
}
</style>
