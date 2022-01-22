<template>
    <div>
        <!--面包屑导航区域-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
        <el-row>
            <el-col>
                <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
            </el-col>
        </el-row>

        <!-- 表格 -->
        <tree-table :data = "catelist"
        :columns = "columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
        class = "treeTable">
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false"
          style="color: lightgreen;"></i>
          <i class="el-icon-error" v-else
          style="color: red;"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditCatesDialog(scope.row.cat_id)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeCateById(scope.row.cat_id)">删除</el-button>
        </template>
        </tree-table>

        <!--底部分页区域-->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[3, 5, 10, 15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog
  title="添加分类"
  :visible.sync="addCateDialogVisible"
  width="50%"
  @close="addCateDialogClosed">
  <!--添加分类的表单-->
  <el-form :model="addCateForm" :rules="addCateFormRules"
  ref="addCateFormRef" label-width="100px">
  <el-form-item label="分类名称: " prop="cat_name">
    <el-input v-model="addCateForm.cat_name"></el-input>
  </el-form-item>
  <el-form-item label="父级分类: ">
    <!-- options用来绑定数组 -->
    <!-- props用来指定配置对象 -->
    <el-cascader
    :options="ParentCateList"
    :props="cascaderprops"
    v-model="selectedKeys"
    @change="parentCateChanged"
    clearable
    filterable
    placeholder="请选择分类">
  </el-cascader>
  </el-form-item>
  </el-form>
  <!--底部区域-->
  <span slot="footer" class="dialog-footer">
    <el-button @click="addCateDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCate">确 定</el-button>
  </span>
</el-dialog>
<!--编辑分类的对话框-->
<el-dialog
  title="修改商品分类信息"
  :visible.sync="editcatesdialog"
  width="50%"
  @close="editCatesDialogClosed">
  <!--内容主题区域-->
  <el-form ref="editFormRef" :model="editForm" label-width="100px" :rules="addCateFormRules">
  <el-form-item label="分类名称"  prop="cat_name" label-width='100px'>
    <el-input v-model="editForm.cat_name"></el-input>
  </el-form-item>
  </el-form>
  <!--底部区域-->
  <span slot="footer" class="dialog-footer">
    <el-button @click="editcatesdialog = false">取 消</el-button>
    <el-button type="primary" @click="eidtCateInfo">确 定</el-button>
  </span>
</el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表，默认为空
      catelist: [],
      // 总数据条数
      total: 0,
      // 为table指定列的数据
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'isok'
        },
        {
          label: '排序',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'order'
        },
        {
          label: '操作',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'opt'
        }
      ],
      // 控制添加分类对话框的可见性
      addCateDialogVisible: false,
      // 添加分类的表单对象
      addCateForm: {
        // 将要添加的分类的名称
        cat_name: '',
        // 父级分类的id
        cat_pid: 0,
        // 当前分类等级,默认为一级分类
        cat_level: 0
      },
      // 添加分类表单的验证规则对象
      addCateFormRules: {
        cat_name: [
          {
            required: true,
            message: '请输入分类名称',
            tigger: 'blur'
          }
        ]
      },
      // 父级分类的列表
      ParentCateList: [],
      // 指定级联选择器的配置对象
      cascaderprops: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover',
        checkStrictly: true
      },
      // 选中的父级分类的Id数组
      selectedKeys: [],
      // 修改商品信息的表单对象
      editForm: {},
      editcatesdialog: false,
      defKeys: []
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories',
        { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败!')
      }
      // 把数据列表赋值catelist
      this.catelist = res.data.result
      // 为总数据条数赋值
      this.total = res.data.total
    },
    // 监听pagesize改变的事件
    handleSizeChange (newsize) {
      this.queryInfo.pagesize = newsize
      this.getCateList()
    },
    // 监听pagenum的改变事件
    handleCurrentChange (newpage) {
      this.queryInfo.pagenum = newpage
      this.getCateList()
    },
    // 点击按钮展示添加对话框，先获取父级分类数据列表
    showAddCateDialog () {
      this.addCateDialogVisible = true
      this.getParentCateList()
    },
    // 获取父级分类的数据列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })

      if (res.meta.status !== 200) {
        return this.$message.error('获取父级列表失败!')
      }
      this.ParentCateList = res.data
    },
    // 选择项发生变化触发此事件
    parentCateChanged () {
      console.log(this.selectedKeys)
      // 如果此数组中length大于0，证明选中了父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类的Id
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1]
        // 为 当前分类的等级
        this.addCateForm.cat_level = this.selectedKeys.length
      } else { // 父级分类的Id
        this.addCateForm.cat_pid = 0
        // 为当前分类的等级
        this.addCateForm.cat_level = 0
      }
    },
    // 点击按钮添加新的分类
    addCate () {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories',
          this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败!')
        }
        this.$message.success('添加分类成功!')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 监听对话框的关闭事件，重置表单数据
    addCateDialogClosed () {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cate_level = 0
      this.addCateForm.cate_pid = 0
    },
    editCatesDialogClosed () {
      this.selectedKeys = []
      this.editForm.cate_level = 0
      this.editForm.cate_pid = 0
      this.cascaderprops.children = 'children'
    },
    // 点击按钮，展示修改信息的对话框
    // eslint-disable-next-line camelcase
    async showEditCatesDialog (cat_id) {
      // eslint-disable-next-line camelcase
      const { data: res } = await this.$http.get('categories/' + cat_id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询商品信息失败!')
      }
      this.editForm = res.data
      this.getCateList()
      this.editcatesdialog = true
    },
    // 点击按钮，修改信息
    eidtCateInfo () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'categories/' + this.editForm.cat_id, {
            cat_name: this.editForm.cat_name
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('更改商品分类失败!')
        }
        this.$message.success('修改用户信息成功!')
        this.getCateList()
        this.editcatesdialog = false
      }
      )
    },
    // 删除角色前的提示操作
    // eslint-disable-next-line camelcase
    async removeCateById (cat_id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该分类，是否继续？',
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
      const { data: res } = await this.$http.delete('categories/' + cat_id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除分类失败!')
      }
      this.getCateList()
      this.$message.success('已删除此分类!')
    }

  }
}
</script>

<style scoped>
.treeTable {
  margin-top: 15px;
}

.el-cascader {
  width: 100%;
}
</style>
