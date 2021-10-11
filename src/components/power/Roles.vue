<template>
  <div>
    <el-breadcrumb separator=">">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <el-row>
        <el-button type="primary" @click="addRoleDialogOpened">添加角色</el-button>
      </el-row>
      <el-table :data="rolesList" style="width: 100%" border stripe>
        <el-table-column type="expand">
          <template v-slot="scope">
            <div class="expand">
              <el-row v-for="firstRight in scope.row.children" :key="firstRight.id">
                <el-col :span="5">
                  <el-tag closable @close="removeRightById(scope.row, firstRight.id)">{{ firstRight.authName }}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <el-col :span="19">
                  <el-row v-for="secondRight in firstRight.children" :key="secondRight.id">
                    <el-col :span="6">
                      <el-tag type="success" closable @close="removeRightById(scope.row, secondRight.id)">{{
                          secondRight.authName
                        }}
                      </el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <el-col :span="18">
                      <el-tag v-for="thirdRight in secondRight.children" :key="thirdRight.id" type="warning" closable
                              @close="removeRightById(scope.row, thirdRight.id)">
                        {{ thirdRight.authName }}
                      </el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </div>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#" align="center"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column prop="level" label="操作">
          <template v-slot="scope">
            <div v-if="false">{{ scope.row }}</div>
            <el-button type="primary" icon="el-icon-edit" @click="editRoleDialogOpened(scope.row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" @click="deleteRole(scope.row.id)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" @click="setRoleRightDialogOpened(scope.row)">分配权限
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-dialog title="添加角色" :visible.sync="addRoleDialogVisible" width="30%" @close="addRoleDialogClosed"
               :close-on-click-modal="false">
      <el-form :model="addRoleFormData" label-width="80px" ref="addRoleFormRef">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRoleFormData.roleName" clearable></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleFormData.roleDesc" clearable></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="修改角色" :visible.sync="editRoleDialogVisible" width="30%" @close="editRoleDialogClosed"
               :close-on-click-modal="false">
      <el-form :model="editRoleFormData" label-width="80px" ref="editRoleFormRef">
        <el-form-item label="角色名称">
          <el-input v-model="editRoleFormData.roleName" clearable></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="editRoleFormData.roleDesc" clearable></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="分配权限" :visible.sync="setRoleRightDialogVisible" width="30%" :close-on-click-modal="false"
               @close="setRoleRightDialogClosed">
      <el-tree :data="rightsTree" :props="rightsTreeProps" show-checkbox default-expand-all node-key="id"
               :default-checked-keys="defKeys" ref="setRoleRightRef">
      </el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoleRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="setRoleRight">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: 'Roles',
  data () {
    return {
      rolesList: [],
      addRoleFormData: {
        roleName: '',
        roleDesc: ''
      },
      editRoleFormData: {
        roleId: '',
        roleName: '',
        roleDesc: ''
      },
      rightsTree: [],
      rightsTreeProps: {
        label: 'authName',
        children: 'children'
      },
      defKeys: [],
      roleId: '',
      addRoleDialogVisible: false,
      editRoleDialogVisible: false,
      setRoleRightDialogVisible: false
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('角色列表获取失败')
      this.rolesList = res.data
      this.$message.success('角色列表获取成功')
    },
    addRoleDialogOpened () {
      this.addRoleDialogVisible = true
    },
    async addRole () {
      const { data: res } = await this.$http.post('roles', this.addRoleFormData)
      if (res.meta.status !== 201) return this.$message.error('添加角色失败')
      this.$message.success('添加角色成功')
      this.addRoleDialogVisible = false
      this.getRolesList()
    },
    addRoleDialogClosed () {
      this.$refs.addRoleFormRef.resetFields()
    },
    async editRoleDialogOpened (id) {
      this.editRoleDialogVisible = true
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('角色信息查询失败')
      this.$message.success('角色信息查询成功')
      this.editRoleFormData.roleId = res.data.roleId
      this.editRoleFormData.roleName = res.data.roleName
      this.editRoleFormData.roleDesc = res.data.roleDesc
    },
    async editRole () {
      const { data: res } = await this.$http.put('roles/' + this.editRoleFormData.roleId, {
        roleName: this.editRoleFormData.roleName,
        roleDesc: this.editRoleFormData.roleDesc
      })
      if (res.meta.status !== 200) return this.$message.error('角色信息修改失败')
      this.$message.success('角色信息修改成功')
      this.editRoleDialogVisible = false
      this.getRolesList()
    },
    editRoleDialogClosed () {
      this.$refs.editRoleFormRef.resetFields()
    },
    async deleteRole (id) {
      const returnValue = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)
      if (returnValue === 'cancel') return this.$message.error('已取消删除')
      if (returnValue === 'confirm') {
        const { data: res } = await this.$http.delete('roles/' + id)
        if (res.meta.status !== 200) return this.$message.error('角色删除失败')
        this.getRolesList()
        this.$message.success('角色删除成功')
      }
    },
    async removeRightById (role, rightId) {
      const returnValue = await this.$confirm('是否要删除该权限', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)
      if (returnValue === 'cancel') return this.$message.error('已取消删除')
      if (returnValue === 'confirm') {
        const { data: res } = await this.$http.delete('roles/' + role.id + '/rights/' + rightId)
        if (res.meta.status !== 200) return this.$message.error('权限删除失败')
        role.children = res.data
        this.$message.success('权限删除成功')
      }
    },
    getDefKeys (node) {
      if (!node.children) return this.defKeys.push(node.id)
      if (node.children.length > 0) {
        node.children.forEach(item => this.getDefKeys(item))
      }
    },
    async setRoleRightDialogOpened (role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('权限获取失败')
      this.rightsTree = res.data
      this.getDefKeys(role)
      this.setRoleRightDialogVisible = true
    },
    async setRoleRight () {
      const rids = [...this.$refs.setRoleRightRef.getCheckedKeys(), ...this.$refs.setRoleRightRef.getHalfCheckedKeys()].join(',')
      const { data: res } = await this.$http.post('roles/' + this.roleId + '/rights', { rids: rids })
      if (res.meta.status !== 200) return this.$message.error('权限设置失败')
      this.$message.success('权限设置成功')
      this.getRolesList()
      this.rightsTree = res.data
      this.setRoleRightDialogVisible = false
    },
    setRoleRightDialogClosed () {
      this.defKeys = []
    }
  }
}
</script>
<style lang="scss" scoped>
.box-card {
  width: 100%;
  margin-top: 15px;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.15) !important;

  .el-table {
    margin-top: 15px;

    .expand {
      padding: 10px 48px;

      .el-row {
        border-bottom: 1px solid #eee;
        display: flex;
        justify-content: center;
        align-items: center;

        &:first-child {
          border-top: 1px solid #eee;
        }

        .el-tag {
          margin: 10px;
        }

        .el-row {
          border: none;
          border-top: 1px solid #eee;

          &:first-child {
            border-top: none;
          }
        }
      }
    }
  }
}
</style>
