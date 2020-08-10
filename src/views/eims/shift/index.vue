<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">班次编号</label>
        <el-input v-model="query.shiftCode" clearable placeholder="班次编号" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">班次名称</label>
        <el-input v-model="query.shiftName" clearable placeholder="班次名称" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="班次编号" prop="shiftCode">
            <el-input v-model="form.shiftCode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="班次名称" prop="shiftName">
            <el-input v-model="form.shiftName" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="shiftCode" label="班次编号" />
        <el-table-column prop="shiftName" label="班次名称" />
        <el-table-column prop="createBy" label="创建者" />
        <el-table-column prop="updateBy" label="更新者" />
        <el-table-column prop="createTime" label="创建日期">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.createTime) }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="updateTime" label="更新时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.updateTime) }}</span>
          </template>
        </el-table-column>
        <el-table-column v-permission="['admin','shift:edit','shift:del']" label="操作" width="150px" align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudShift from '@/api/shift'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { shiftCode: null, shiftName: null, createBy: null, updateBy: null, createTime: null, updateTime: null }
export default {
  name: 'Shift',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  cruds() {
    return CRUD({ title: '班次管理', idField: 'shiftCode', url: 'api/shift', sort: 'shiftCode,desc', crudMethod: { ...crudShift }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'shift:add'],
        edit: ['admin', 'shift:edit'],
        del: ['admin', 'shift:del']
      },
      rules: {
        shiftCode: [
          { required: true, message: '班次编号不能为空', trigger: 'blur' }
        ],
        shiftName: [
          { required: true, message: '班次名称不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'shiftCode', display_name: '班次编号' },
        { key: 'shiftName', display_name: '班次名称' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>
