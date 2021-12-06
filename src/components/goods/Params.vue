/* eslint-disable vue/no-parsing-error */

<template lang="">
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item><a href="/">商品管理</a></el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区 -->
    <el-card>
      <!-- 头部警告区域 -->
      <el-alert
        title="注意：只允许为第三类分类设置相关参数！"
        type="warning"
        :alert="false"
        :show-icon="true"
      >
      </el-alert>
      <!-- 选择商品分类区 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类的级联选择框 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="{ expandTrigger: 'hover', ...cateProps }"
            @change="handleChange"
          ></el-cascader>
        </el-col>
      </el-row>

      <!-- tab页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many"
          ><el-button type="primary" size="mini" :disabled="buttonDisabled"
          @click="addDialogVisible=true"
            >添加参数</el-button
          >
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag @close="handleClose(i,scope.row)" closable v-for="(item,i) in scope.row.attr_vals" :key="i">
                  {{item}}
                </el-tag>
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
<el-button
  v-else
  class="button-new-tag"
  size="small"
  @click="showInput(scope.row)"
>+ New Tag</el-button>

              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <div>
                <el-button type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
     <el-button type="danger" icon="el-icon-delete" @click = " removeParams (scope.row.attr_id) " >删除</el-button>
                    </div>
                </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only"
          ><el-button type="primary" size="mini" :disabled="buttonDisabled"
          @click="addDialogVisible =true"
            >添加属性</el-button
          >
          <!-- 静态属性表格 -->
<el-table :data="onlyTableData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag @close="handleClose(i,scope.row)" closable v-for="(item,i) in scope.row.attr_vals" :key="i">
                  {{item}}
                </el-tag>
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
<el-button
  v-else
  class="button-new-tag"
  size="small"
  @click="showInput(scope.row)"
>+ New Tag</el-button>

              </template>
            </el-table-column>

            <el-table-column type="index"></el-table-column>
            <el-table-column
              label="属性名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
              <div>
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                  >编辑</el-button
                >
                <el-button type="danger" icon="el-icon-delete"             @click="removeParams(scope.row.attr_id)"
            >删除</el-button>
              </div>
            </template>
</el-table-column>
          </el-table>
</el-tab-pane
        >
      </el-tabs>
    </el-card>
    <!-- 添加参数的对话框 -->
    <el-dialog
  :title="'添加'+titleText"
  :visible.sync="addDialogVisible"
  width="50%"
  @close="addDialogClosed"
>
 <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" class="demo-ruleForm">
  <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="addForm.attr_name"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible
 = false">取 消</el-button>
    <el-button type="primary" @click="addParams">确 定</el-button>
  </span>
</el-dialog>
<!-- 修改参数的对话框 -->
<el-dialog
  :title="'修改' + titleText"
  :visible.sync="editDialogVisible"
  width="50%"
  @close="editDialogClosed"
>
 <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" class="demo-ruleForm">
  <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="editForm.attr_name"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible
 = false">取 消</el-button>
    <el-button type="primary" @click="editParams">确 定</el-button>
  </span>
</el-dialog>

  </div>
</template>
<script>
export default {
  data () {
    return {
      cateList: [],
      //   级联选择器的配置对象
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      //   级联选择器的绑定数组
      selectedCateKeys: [],
      //   被激活的页签名称
      activeName: 'many',
      manyTableData: [],
      onlyTableData: [],

      addDialogVisible: false,
      addForm: {},
      addFormRules: {
        attr_name: [
          {
            required: true,
            message: '请输入参数',
            trigger: 'blur'
          }
        ]
      },
      addFormRef: {},
      editDialogVisible: false,
      editForm: {},
      editFormRules: {
        attr_name: [{ required: true, message: '请输入参数', trigger: 'blur' }]
      }

    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.message.error('获取商品分类失败！')
      }
      this.cateList = res.data
      console.log(this.cateList)
    },
    async handleChange () {
      this.getParamsData()
    },
    handleTabClick () {
      console.log(this.activeName)
      this.getParamsData()
    },
    async getParamsData () {
      console.log(this.selectedCateKeys)
      if (
        // 证明选中的不是三级分类 if
        this.selectedCateKeys.length < 3
      ) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        this.$message.error('只允许选中三级分类设置相关参数！')
      }
      console.log(this.selectedCateKeys)
      // 根据所选分类的id和当前所处的面板获取对应的参数
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        { params: { sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败！')
      }
      console.log(res.data)
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals
          ? item.attr_vals = item.attr_vals.split(' ')
          : []
          // 控制文本框的显示与隐藏
        item.inputVisible = false
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    addParams () {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          { attr_name: this.addForm.attr_name, attr_sel: this.activeName }
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败！')
        }
        this.$message.success('添加参数成功！')
        this.getParamsData()
        this.addDialogVisible = false
      })
    },
    async showEditDialog (id) {
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${id}`,
        { params: { attr_sel: this.activeName } }
      )
      if (res.meta.status !== 200) { return this.$message.error('获取参数信息失败') }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    editParams () {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          { attr_name: this.editForm.attr_name, attr_sel: this.activeName }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败！')
        }
        this.$message.success('修改参数成功！')
        this.getParamsData()
        console.log(1)
        this.editDialogVisible = false
      })
    },
    async removeParams (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText:
'确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') { return this.$message.info('已取消删除！') }
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${id}`)
      if (res.meta.status !== 200) { return this.$message.error('删除参数失败！') }
      this.$message.success('删除参数成功！')
      this.getParamsData()
    },
    async handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      console.log(this.manyTableData)
      row.inputValue = ''
      row.inputVisible = false
      // 需要发起请求保存这次操作
      this.saveAttrVals(row)
    },
    // $nextTick方法的作用，就是当页面上元素被重新渲染之后，才会指定回调函数中的代码
    showInput (row) {
      row.inputVisible = true
      this.$nextTick(_ => { this.$refs.saveTagInput.$refs.input.focus() })
    },
    // 删除对应的参数选项
    handleClose (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    },
    async saveAttrVals (row) {
      const { data: res } = await
      this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals:
row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) return
      this.$message.error('修改参数项失败！')
      this.$message.success('修改参数项成功！')
      console.log(res.data)
    }
  },
  computed: {
    buttonDisabled: function () {
      if (this.selectedCateKeys.length === 3) {
        return false
      }
      return true
    },
    cateId () {
      if (this.selectedCateKeys.length === 3) return this.selectedCateKeys[2]
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
.el-cascader {
  width: 300px;
}
.el-tag{
  margin:10px;
}
.input-new-tag{
  width:150px;
}
</style>
