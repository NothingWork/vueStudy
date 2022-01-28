<template>
  <div>
    <!--面包屑导航区域-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <!--搜索与添加区域-->
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
            <el-button type="primary" @click="goAddPage">添加商品</el-button>
        </el-col>
      </el-row>
      <!--商品列表区域-->
      <el-table :data="goodlist" stripe border>
      <el-table-column type="index"></el-table-column>
      <el-table-column prop="goods_name" label="商品名称"></el-table-column>
      <el-table-column prop="goods_price" label="价格(元)" width="95px"></el-table-column>
      <el-table-column prop="goods_weight" label="商品重量" width="70px"></el-table-column>
      <el-table-column prop="add_time" label="创建时间" width="140px">
          <template slot-scope="scope">
            {{scope.row.add_time | dataFormat}}
          </template>
      </el-table-column>
      <el-table-column  label="操作" width="130px">
        <template slot-scope="scope">
      <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
      <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeGood(scope.row.goods_id)"></el-button >
        </template>
      </el-table-column>
      </el-table>
      <!--底部分页区域-->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>

  </div>
</template>

<script>
export default {
  data () {
    return {
      // 获取商品数据列表
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      goodlist: [],
      total: 0
    }
  },
  created () {
    this.getGoodList()
  },
  methods: {
    async getGoodList () {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败!')
      }
      this.goodlist = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange (newpagesize) {
      this.queryInfo.pagesize = newpagesize
      this.getGoodList()
    },
    handleCurrentChange (newpagenum) {
      this.queryInfo.pagenum = newpagenum
      this.getGoodList()
    },
    // 删除商品
    // eslint-disable-next-line camelcase
    async removeGood (goods_id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户，是否继续？',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 如果用户确认删除，则返回值为字符串 confirm
      // 如果用户取消删除，则返回值为字符串 cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // 删除用户操作
      // eslint-disable-next-line camelcase
      const { data: res } = await this.$http.delete('goods/' + goods_id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除商品失败!')
      }
      this.getGoodList()
      this.$message.success('已删除此商品!')
    },
    goAddPage () {
      this.$router.push('goods/add')
    }
  }
}
</script>

<style lang="less" scoped>
</style>
