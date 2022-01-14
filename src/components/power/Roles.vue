<template>
  <div>
    <!--面包屑导航区域-->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--卡片视图-->
    <el-card>
      <el-row :gutter="20">
            <!--添加区域-->
        <el-col :span="4">
            <el-button type="primary" @click="addRolesDialog = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!--角色列表区域-->
      <el-table :data="rolelist" stripe border>
          <!-- 展开列 -->
          <el-table-column type="expand">
            <template slot-scope="scope">
              <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                <!-- 渲染一级权限 -->
                <el-col :span="5">
                  <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <!-- 渲染二级和三级权限 -->
                <el-col :span="19">
                  <!-- 通过for 循环嵌套渲染二级权限 -->
                  <el-row :class="[i2 === 0 ? '': 'bdtop','vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                    <el-col :span="6">
                      <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <el-col :span="18">
                      <el-tag closable type="warning"
                      @close="removeRightById(scope.row, item3.id)"
                      v-for="(item3) in item2.children" :key="item3.id">
                        {{item3.authName}}
                      </el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </template>
          </el-table-column>
      <!-- 索引列 -->
      <el-table-column type="index"></el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
      <el-table-column label="操作" width="300px">
       <template slot-scope="scope">
           <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
           <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleById(scope.row.id)">删除</el-button >
           <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRoleRightDialog(scope.row)">分配权限</el-button>
           </template>
      </el-table-column>
    </el-table>
    <!--添加用户的对话框-->
      <el-dialog
  title="添加用户"
  :visible.sync="addRolesDialog"
  @close="addDialogClose"
  width="50%">
  <!--内容主题区域-->
  <el-form ref="addFormRef" :model="addForm" label-width="70px" :rules="addFormRules">
  <el-form-item label="角色名称" prop="roleName" label-width='100px'>
    <el-input v-model="addForm.roleName"></el-input>
  </el-form-item>
  <el-form-item label="角色描述" prop="roleDesc" label-width='100px'>
    <el-input v-model="addForm.roleDesc"></el-input>
  </el-form-item>
  </el-form>
  <!--底部区域-->
  <span slot="footer" class="dialog-footer">
    <el-button @click="addRolesDialog = false">取 消</el-button>
    <el-button type="primary" @click="addRole">确 定</el-button>
  </span>
</el-dialog>
<!--修改用户的对话框-->
<el-dialog
  title="修改用户信息"
  :visible.sync="editRolesDialog"
  @close="editDialoGClose"
  width="50%">
  <!--内容主题区域-->
  <el-form ref="eidtFormRef" :model="editForm" label-width="70px" :rules="addFormRules">
  <el-form-item label="角色名称" prop="roleName" label-width='100px'>
    <el-input v-model="editForm.roleName"></el-input>
  </el-form-item>
  <el-form-item label="角色描述" prop="roleDesc" label-width='100px'>
    <el-input v-model="editForm.roleDesc"></el-input>
  </el-form-item>
  </el-form>
  <!--底部区域-->
  <span slot="footer" class="dialog-footer">
    <el-button @click="editRolesDialog = false">取 消</el-button>
    <el-button type="primary" @click="eidtRoleInfo">确 定</el-button>
  </span>
</el-dialog>
<el-dialog
  title="分配权限"
  :visible.sync="showSetRightDialogVisible"
  width="40%"
  @close="SetRightDialogVisibleClosed">
  <!-- 树形控件 -->
<el-tree :data="rightslist"
:props="treeProps"
show-checkbox
node-key="id"
default-expand-all
:default-checked-keys="defKeys"
ref="treeRef"
></el-tree>
  <span slot="footer" class="dialog-footer">
    <el-button @click="showSetRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
</el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 分配角色权限对话框
      showSetRightDialogVisible: false,
      // 所有角色列表数据
      rolelist: [],
      // 添加用户对话框的可见性
      addRolesDialog: false,
      // 添加用户的表单数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      // 添加用户的表单规则
      addFormRules: {
        roleName: [{
          required: true,
          message: '角色名称不能为空!',
          trigger: 'blur'
        }]
      },
      // 控制修改用户信息对话框的可见性
      editRolesDialog: false,
      // 查询到的用户信息对象
      editForm: {},
      // 所有权限的数据
      rightslist: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      defKeys: [],
      // 当前即将分配角色的id
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色列表
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')

      if (res.meta.status !== 200) {
        return this.$message.errer('获取角色列表失败')
      }
      this.rolelist = res.data
    },
    addDialogClose () {
      this.$refs.addFormRef.resetFields()
    },
    addRole () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        // 可以发起添加用户的网络请求
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) { this.$message.error('添加角色失败!') }
        this.getRolesList()
        this.$message.success('添加角色成功!')
        this.addRolesDialog = false
      })
    },
    // 展示用户信息编辑对话框
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('/roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败!')
      }
      this.editForm = res.data
      this.editRolesDialog = true
    },
    // 编辑用户信息对话框关闭的重置操作
    editDialoGClose () {
      this.$refs.eidtFormRef.resetFields()
    },
    // 修改用户信息的预验证和提交
    eidtRoleInfo () {
      this.$refs.eidtFormRef.validate(async valid => {
        if (!valid) return
        // 发起修改用户信息的提交请求
        const { data: res } = await this.$http.put(
          'roles/' + this.editForm.roleId, {
            roleDesc: this.editForm.roleDesc,
            roleName: this.editForm.roleName
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改用户信息失败!')
        }
        // 关闭对话框
        this.editRolesDialog = false
        // 更新用户数据
        this.getRolesList()
        // 提示修改成功
        this.$message.success('修改用户信息成功!')
      }
      )
    },
    // 删除角色前的提示操作
    async removeRoleById (id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该角色，是否继续？',
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
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败!')
      }
      this.getRolesList()
      this.$message.success('已删除此角色!')
    },
    // 删除角色权限前的提示操作
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该权限，是否继续？',
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

      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色权限失败!')
      }
      role.children = res.data
      this.$message.success('删除角色权限成功!')
    },
    // 角色权限信息弹窗
    async showSetRoleRightDialog (role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色权限信息失败!')
      }
      // 把获取到的数据保存到data中
      this.rightslist = res.data
      console.log(this.rightslist)
      this.getLeafKeys(role, this.defKeys)
      this.showSetRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有权限的id，并保存到数组中
    getLeafKeys (node, arr) {
      // 如果当前node没有children属性则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件
    SetRightDialogVisibleClosed () {
      // 清空 defkeys 数组  避免累积
      this.defKeys = []
    },
    // 点击分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()

      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('更改角色权限失败!')
      }
      this.$message.success('更改角色权限成功!')
      this.getRolesList()
      this.showSetRightDialogVisible = false
    }
  }
}

</script>

<style lang="less" scoped>
.el-tag{
  margin: 7px;
}
.bdtop{
  border-top: 1px solid #eee;
}
.bdbottom{
  border-bottom: 1px solid #eee;
}
.vcenter{
  display: flex;
  align-items: center;
}
</style>
