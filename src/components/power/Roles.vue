<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>角色管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
              :class="['bd-bottom', i1 === 0 ? 'bd-top' : '', 'vcenter']"
            >
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRights(scope.row, item2.id)">
                  {{ item1.authName }}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                  :class="['vcenter', i2 === 0 ? '' : 'bd-top']"
                >
                  <el-col :span="7">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRights(scope.row, item2.id)"
                    >
                      {{ item2.authName }}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="14">
                    <el-tag
                      type="warning"
                      v-for="(item3, i3) in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRights(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column
          prop="roleName"
          label="角色名称"
          size="mini"
        ></el-table-column>
        <el-table-column
          prop="roleDesc"
          label="角色描述"
          size="mini"
        ></el-table-column>
        <el-table-column label="操作" size="mini">
          <template slot-scope="scope" width="300px">
            <el-button type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete">删除</el-button>
            <el-button
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>

      <el-dialog
        title="分配权限"
        :visible.sync="dialogVisible"
        width="50%"
        @close="setDialogClosed"
      >
        <el-tree
          :data="rightsList"
          :props="treeProps"
          show-checkbox
          node-key="id"
          default-expand-all
          :default-checked-keys="checkedKeys"
          ref="treeRef"
        ></el-tree>
        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allLotRights">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rolesList: [],
      dialogVisible: false,
      rightsList: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      checkedKeys: [],
      roleId: ''
    }
  },
  created() {
    this.getRoleslist()
  },
  methods: {
    async getRoleslist() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户角色列表失败')
      }
      this.rolesList = res.data
    },
    async removeRights(role, rightId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      role.children = res.data
    },
    async showSetRightDialog(role) {
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限失败')
      }
      this.rightsList = res.data
      console.log(this.rightsList)
      this.roleId = role.id
      this.getKeys(role, this.checkedKeys)
      this.dialogVisible = true
    },
    getKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getKeys(item, arr)
      })
    },
    setDialogClosed() {
      this.checkedKeys = []
    },
    async allLotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('角色授权失败')
      }
      this.$message.success('角色授权成功')
      this.getRoleslist()
      this.dialogVisible = false
    }
  }
}
</script>

<style scoped>
.el-tag {
  margin: 8px;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
