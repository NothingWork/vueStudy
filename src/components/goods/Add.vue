<template>
    <div>
        <!--面包屑导航区域-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item :to="{ path: '/goods' }">商品列表</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 提示区域 -->
      <el-alert
    title="添加商品信息"
    type="info"
    center
    show-icon
    :closable="false">
  </el-alert>

  <!-- 步骤条区域 -->
  <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
  <el-step title="基本信息"></el-step>
  <el-step title="商品参数"></el-step>
  <el-step title="商品属性"></el-step>
  <el-step title="商品图片"></el-step>
  <el-step title="商品内容"></el-step>
  <el-step title="完成"></el-step>
</el-steps>

<!-- tab侧边栏区域 -->
<el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position = "top">
<el-tabs v-model="activeIndex" :tab-position="'left'" :before-leave="beforeTabLeave"
@tab-click="tabClicked">
    <el-tab-pane label="基本信息" name="0">
      <el-form-item label="商品名称" prop="goods_name">
    <el-input v-model="addForm.goods_name" ref="input"></el-input>
  </el-form-item>
  <el-form-item label="商品价格" prop="goods_price">
    <el-input v-model="addForm.goods_price" type="number"></el-input>
  </el-form-item>
  <el-form-item label="商品重量" prop="goods_weight">
    <el-input v-model="addForm.goods_weight" type="number"></el-input>
  </el-form-item>
  <el-form-item label="商品数量" prop="goods_number">
    <el-input v-model="addForm.goods_number" type="number"></el-input>
  </el-form-item>
  <el-form-item label="商品分类" prop="goods_cat">
    <el-cascader
    v-model="addForm.goods_cat"
    :options="catelist"
    :props="casprops"
    @change="handleChange"
    clearable
    filterable
    placeholder="请选择三级分类(必填)"
    ref="cascaderRef"></el-cascader>
  </el-form-item>
    </el-tab-pane>
    <el-tab-pane label="商品参数" name="1">
      <!-- 渲染表单的item项 -->
      <el-form-item :label="item.attr_name" v-for="item in manyTableData"
      :key="item.attr_id"
      >
      <!-- 渲染复选框组 -->
    <el-checkbox :label="cb" v-for="(cb, i) in item.attr_vals" :key="i" border
    @change="checked=>boxChanged(checked,item,cb)">
    </el-checkbox>
  </el-form-item>
    </el-tab-pane>
    <el-tab-pane label="商品属性" name="2">
      <el-form-item :label="item.attr_name" v-for="item in onlyTabData" :key="item.attr_id">
        <el-input v-model="item.attr_vals"></el-input>
      </el-form-item>
    </el-tab-pane>
    <el-tab-pane label="商品图片" name="3">
       <el-upload
            class="upload-demo"
            :action="uploadURL"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            list-type="picture"
            :headers="headersObj"
            :on-success="handleSuccess">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
      </el-tab-pane>
    <el-tab-pane label="商品内容" name="4">
      <!-- 富文本编辑器组件 -->
      <quill-editor
    v-model="addForm.goods_introduce"></quill-editor>
    <!-- 添加商品的按钮 -->
    <el-button type="primary" class="btn" @click="add">添加商品</el-button>
  </el-tab-pane>
  </el-tabs>
  </el-form>

  <el-row >
    <el-col :span="1" :offset="11">
  <el-button type="primary" icon="el-icon-caret-left" circle @click="perstep"></el-button>
    </el-col>
    <el-col :span="1">
  <el-button type="primary" icon="el-icon-caret-right" circle @click="nextstep"></el-button>
    </el-col>
  </el-row>
    </el-card>

    <!-- 图片预览对话框 -->
    <el-dialog
  title="图片预览"
  :visible.sync="previewVisible"
  width="50%">
  <img :src="previewPath" alt="" class="previewImg">
</el-dialog>

    </div>
</template>

<script>
import _ from 'lodash'
export default {
  data () {
    return {
      activeIndex: '0',
      // 添加表单绑定的数据对象
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      // 添加表单的规则
      addFormRules: {
        goods_name: [{
          required: true,
          message: '请输入商品名称',
          trigger: 'blur'
        }],
        goods_price: [{
          required: true,
          message: '请输入商品价格',
          trigger: 'blur'
        }],
        goods_weight: [{
          required: true,
          message: '请输入商品重量',
          trigger: 'blur'
        }],
        goods_number: [{
          required: true,
          message: '请输入商品数量',
          trigger: 'blur'
        }],
        goods_cat: [{
          required: true,
          message: '请输入商品分类',
          trigger: 'blur'
        }]
      },
      // 商品分类列表
      catelist: [],
      // 级联选择器的属性设置
      casprops: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover',
        checkStrictly: true
      },
      // 动态参数数据列表
      manyTableData: [],
      // 静态参数数据
      onlyTabData: [],
      // 上传图片的URL地址
      uploadURL: 'https://www.liulongbin.top:8888/api/private/v1/upload',
      // 图片上传组件的headers请求头对象
      headersObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      // 图片预览路径
      previewPath: '',
      //  图片预览窗口
      previewVisible: false
    }
  },
  created () {
    this.getCateList()
    this.getFocus()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        this.$message.error('获取商品分类数据失败!')
      }
      this.catelist = res.data
    },
    // 页面创建时让商品名称的文本框获得焦点
    getFocus () {
      this.$nextTick(() => {
        this.$refs.input.focus()
      })
    },
    // 级联选择器选中时的触发事件
    handleChange () {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
        return this.$message.error('选择三级分类(=.=)')
      }
      this.$refs.cascaderRef.dropDownVisible = false
    },
    // 点击按钮切换上一页面
    perstep () {
      let num = this.activeIndex - 0
      if (num === 0) {
        return this.$message.warning('已经是第一页了')
      }
      num--
      this.activeIndex = num.toString()
      this.tabClicked()
    },
    // 点击按钮，切换上一页
    nextstep () {
      let num = this.activeIndex - 0
      if (num === 4) {
        return this.$message.warning('已经是最后一页了')
      }
      num++
      this.activeIndex = num.toString()
      this.tabClicked()
    },
    // tabs页面转换前检查信息
    beforeTabLeave (activeName, oldActiveName) {
      let re = ''
      if (oldActiveName === '0') {
        this.$refs.addFormRef.validate(valid => {
          if (!valid) {
            re = valid
            this.$message.error('请完善表单项!')
            this.activeIndex = '0'
          }
        })
        return re
      }
    },
    // 转换至第二页获取商品动态参数
    async tabClicked () {
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'many' } })
        if (res.meta.status !== 200) {
          return this.$message.error('获取参数失败!')
        }
        res.data.forEach(item => {
          item.attr_vals =
          item.attr_vals.length === 0
            ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
        console.log(this.manyTableData)
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'only' } })
        if (res.meta.status !== 200) {
          return this.$message.error('获取参数失败!')
        }
        this.onlyTabData = res.data
      }
    },
    // 处理图片的预览效果
    handlePreview (file) {
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    // 处理图片的移除效果
    handleRemove (file) {
      // 1.获得删除图片的临时路径
      const filePath =
      file.response.data.tmp_path
      // 2找到图片索引值
      const i =
      this.addForm.pics.findIndex(x =>
        x.pic === filePath)
      // 3.从pics数组中删除
      this.addForm.pics.splice(i, 1)
      console.log(this.addForm)
    },
    // 监听图片上传成功事件
    // 监听图片上传成功的事件
    handleSuccess (response) {
      // 1. 拼接得到一个图片信息对象
      const picInfo = { pic: response.data.tmp_path }
      // 2. 将图片信息对象,push到pics数组中
      this.addForm.pics.push(picInfo)
      console.log(response)
      console.log(this.addForm)
    },
    async add () {
      // 对addform对象进行深拷贝
      const form = _.cloneDeep(this.addForm)
      form.goods_cat = form.goods_cat.join(',')
      // 处理静态参数
      this.onlyTabData.forEach(item => {
        const newInfo = {
          attr_id: item.attr_id,
          attr_value: item.attr_vals
        }
        this.addForm.attrs.push(newInfo)
      })
      form.attrs = this.addForm.attrs
      console.log(form)
      const { data: res } = await this.$http.post('goods', form)
      if (res.meta.status !== 201) {
        this.addForm.attrs = []
        console.log(res)
        return this.$message.error('添加商品失败!')
      }
      this.$message.success('添加商品成功!')
      console.log(res)
      this.addForm.attrs = []
      this.$router.push('/goods')
    },
    // 通过checkbox的点击事件处理动态参数
    boxChanged (checked, item, cb) {
      if (checked === true) {
        const newInfo = {
          attr_value: cb,
          attr_id: item.attr_id
        }
        this.addForm.attrs.push(newInfo)
      } else if (checked === false) {
        this.addForm.attrs = _.pullAllWith(this.addForm.attrs, [
          { attr_value: cb, attr_id: item.attr_id }], _.isEqual
        )
      }
    }
  },
  computed: {
    cateId () {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
  .el-steps {
    margin: 15px 0;
  }

  /deep/ .el-step__title {
    font-size: 13px;
  }

  .el-checkbox {
    margin: 10px 10px 0 0 !important;
  }

  .previewImg {
    width: 100%;
  }

  .btn {
    margin-top: 15px;
  }
</style>
