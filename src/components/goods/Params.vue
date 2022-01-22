<template>
    <div>
        <!--面包屑导航区域-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
        <!-- 警告区域 -->
        <el-alert
    title="注意: 只允许为第三级分类设置相关参数!"
    type="warning"
    :closable="false"
    show-icon>
    </el-alert>
    <!-- 选择商品分类区域 -->
    <el-row class="cat_opt">
        <el-col>
            <span>选择商品分类: </span>
            <!-- 选择商品分类的级联选择框 -->
         <!-- options用来绑定数组 -->
    <!-- props用来指定配置对象 -->
    <el-cascader
    :options="catelist"
    :props="cateprops"
    v-model="selectedCateKeys"
    ref="cascaderRef"
    @change="handleChanged"
    clearable
    filterable
    placeholder="请选择分类">
  </el-cascader>
        </el-col>
    </el-row>
    <!-- tab页签区域 -->
    <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数的面板 -->
    <el-tab-pane label="动态参数" name="many">
        <!-- 添加参数的按钮 -->
        <el-button type="primary" size="mini"
        :disabled="isBtnDiabled" @click="addDialogVisible = true">添加参数</el-button>
        <!-- 动态参数表格 -->
        <el-table :data="manyTableData" stripe border>
          <!-- 展开列 -->
          <el-table-column type="expand">
            <template slot-scope="scope">
              <!-- 循环渲染按钮 -->
              <el-tag v-for="(item, i) in scope.row.attr_vals"
              :key="i" closable
              @close="handleClosed(i,scope.row)">{{item}}</el-tag>
              <!-- 按钮-文本框集合体渲染 -->
              <el-input
               class="input-new-tag"
               v-if="scope.row.inputVisible"
               v-model="scope.row.inputValue"
               ref="saveTagInput"
               size="small"
               @keyup.enter.native="handleInputConfirm(scope.row)"
               @blur="handleInputConfirm(scope.row)"
              >
              </el-input>
              <el-button v-else class="button-new-tag" size="small"
              @click="showInput(scope.row)">
              + 添加参数</el-button>
            </template>
          </el-table-column>
          <!-- 索引列 -->
          <el-table-column type="index"></el-table-column>
          <el-table-column label="参数名称" prop="attr_name">
          </el-table-column>
          <el-table-column label="操作">
            <template  slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini"
            @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini"
            @click="removeParams(scope.row.attr_id)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
    </el-tab-pane>
    <!-- 添加静态参数的面板 -->
    <el-tab-pane label="静态属性" name="only">
        <!-- 添加属性的按钮 -->
        <el-button type="primary" size="mini"
        :disabled="isBtnDiabled" @click="addDialogVisible = true">添加属性</el-button>
        <!-- 静态属性表格 -->
        <el-table :data="onlyTableData" stripe border>
          <!-- 展开列 -->
          <el-table-column type="expand">
            <template slot-scope="scope">
              <!-- 循环渲染按钮 -->
              <el-tag v-for="(item, i) in scope.row.attr_vals"
              :key="i" closable
              @close="handleClosed(i,scope.row)">{{item}}</el-tag>
              <!-- 按钮-文本框集合体渲染 -->
              <el-input
               class="input-new-tag"
               v-if="scope.row.inputVisible"
               v-model="scope.row.inputValue"
               ref="saveTagInput"
               size="small"
               @keyup.enter.native="handleInputConfirm(scope.row)"
               @blur="handleInputConfirm(scope.row)"
              >
              </el-input>
              <el-button v-else class="button-new-tag" size="small"
              @click="showInput(scope.row)">
              + 添加参数</el-button>
            </template>
          </el-table-column>
          <!-- 索引列 -->
          <el-table-column type="index"></el-table-column>
          <el-table-column label="属性名称" prop="attr_name">
          </el-table-column>
          <el-table-column label="操作">
            <template  slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini"
            @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini"
            @click="removeParams(scope.row.attr_id)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
    </el-tab-pane>
  </el-tabs>

    </el-card>
    <!-- 添加属性或参数的对话框 -->
    <el-dialog
  :title="'添加' + titleText"
  :visible.sync="addDialogVisible"
  width="50%"
  @close="addDialogClose">
  <el-form ref="addDialogForm"
  :model="addForm" label-width="80px"
  :rules="addFormRules">
  <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="addForm.attr_name"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addParams">确 定</el-button>
  </span>
</el-dialog>

 <!-- 修改属性或参数的对话框 -->
    <el-dialog
  :title="'修改' + titleText"
  :visible.sync="editDialogVisible"
  width="50%"
  @close="editFormClosed">
  <el-form ref="editDialogForm"
  :model="editForm" label-width="80px"
  :rules="addFormRules">
  <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="editForm.attr_name"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editParams">确 定</el-button>
  </span>
</el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    return {
    // 动态参数数据
      manyTableData: [],
      // 静态属性的数据
      onlyTableData: [],
      // 商品分类列表
      catelist: [],
      // 级联选择框的配置对象
      cateprops: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover',
        checkStrictly: true
      },
      // 级联选择框双向绑定到的数组
      selectedCateKeys: [],
      // 被激活的页签的名称
      activeName: 'many',
      // 添加参数或属性的对话框的可见性
      addDialogVisible: false,
      // 添加对话框的绑定对象
      addForm: {
        attr_name: ''
      },
      // 添加表单的验证规则
      addFormRules: {
        attr_name: [{
          required: true,
          message: '添加内容不能为空',
          trigger: 'blur'
        }]
      },
      // 控制修改对话框的可见性
      editDialogVisible: false,
      // 修改对话框的表单数据
      editForm: {}
    }
  },
  created () {
    // 获取所有商品分类列表
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败!')
      }
      this.catelist = res.data
    },
    // 级联选择框选择项变化，会触发这个函数
    handleChanged () {
      this.getParamsData()
      this.$refs.cascaderRef.dropDownVisible = false
    },
    // 获取参数列表的函数
    async getParamsData () {
    // 证明选中的不是三级分类
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return this.$message.error('选择三级分类(╯▔O▔)╯')
      }
      // 证明选中的是三级分类,根据所选分类的Id，和当前所处面板，获取对应的参数
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
        params: { sel: this.activeName }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败!')
      }
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals
          ? item.attr_vals.split(' ') : []
          // 控制文本框的显示与隐藏
        item.inputVisible = false
        // 控制文本框的输入值
        item.inputValue = ''
      })
      console.log(res.data)
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      }
      this.onlyTableData = res.data
    },
    // tab页签的点击事件的处理函数
    handleTabClick () {
      console.log(this.activeName)
      this.getParamsData()
    },
    // 监听添加对话框的关闭，清除表单
    addDialogClose () {
      this.$refs.addDialogForm.resetFields()
    },
    addParams () {
      this.$refs.addDialogForm.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加失败!')
        }
        this.$message.success('添加成功!')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    editFormClosed () {
      this.$refs.editDialogForm.resetFields()
    },
    // 点击编辑按钮，展示修改的对话框
    // eslint-disable-next-line camelcase
    async showEditDialog (attr_id) {
      // 查询分类信息
      // eslint-disable-next-line camelcase
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`,
        { params: { attr_sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('查询信息失败!')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 点击按钮，编辑分类
    editParams () {
      this.$refs.editDialogForm.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.cateId}}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName
          })
        if (res.meta.status !== 200) {
          return this.$message.error('修改内容失败!')
        }
        this.$message.success('修改内容成功!')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // 删除参数前的提示操作
    // eslint-disable-next-line camelcase
    async removeParams (attr_id) {
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
      const { data: res } = await this.$http.delete(`categories/${this.cateId}}/attributes/${attr_id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败!')
      }
      this.getParamsData()
      this.$message.success('已成功删除!')
    },
    // 点击按钮，展示文本框
    showInput (row) {
      row.inputVisible = true
      // 鼠标聚焦文本框
      // $nextTict 方法的作用， 就是当页面上的元素
      // 被重新渲染后，才会指定回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 文本输入后的操作
    async  handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      // 确认无误，push
      row.attr_vals.push(row.inputValue.trim())
      // 输入文本后的提交操作
      row.inputValue = ''
      row.inputVisible = false
      // 发起提交
      this.saveAttrVals(row)
    },
    // 保存并提交修改后的参数
    async saveAttrVals (row) {
      // 发起提交
      const { data: res } = await this.$http.put(`categories/${this.cateId}}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        })
      if (res.meta.status !== 200) {
        return this.message.error('修改失败!')
      }
      this.$message.success('修改成功!')
    },
    handleClosed (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
    // 删除参数标签
  },
  computed: {
    // 如果按钮需要被禁用则返回true，否则返回false
    isBtnDiabled () {
      if (this.selectedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    // 当前选中的三级分类的ID
    cateId () {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
    margin: 15px 0;
}

.el-tag {
  margin: 10px;
}

.input-new-tag {
  width: 120px;
}
</style>
