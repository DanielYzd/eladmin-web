<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">员工编号</label>
        <el-input
          v-model="query.code"
          clearable
          placeholder="员工编号"
          style="width: 185px;"
          class="filter-item"
          @keyup.enter.native="crud.toQuery"
        />
        <label class="el-form-item-label">员工姓名</label>
        <el-input
          v-model="query.name"
          clearable
          placeholder="员工姓名"
          style="width: 185px;"
          class="filter-item"
          @keyup.enter.native="crud.toQuery"
        />
        <label class="el-form-item-label">状态</label>
        <el-select
          v-model="query.status"
          clearable
          placeholder="状态"
          style="width:185px"
          class="filter-item"
          @change="crud.toQuery"
        >
          <el-option :value="0" label="在职" />
          <el-option :value="1" label="离职" />
        </el-select>
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog
        :close-on-click-modal="false"
        :before-close="crud.cancelCU"
        :visible.sync="crud.status.cu > 0"
        :title="crud.status.title"
        width="500px"
      >
        <el-form
          ref="form"
          :model="form"
          :rules="rules"
          size="small"
          label-width="80px"
        >
          <el-form-item label="员工编号" prop="code">
            <el-input v-model="form.code" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="员工姓名" prop="name">
            <el-input v-model="form.name" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="岗位" prop="employeeJob.jobCode">
            <el-select
              v-model="form.employeeJob.jobCode"
              style="width: 370px;"
              filterable
            >
              <el-option v-for="item in employeeJobs" :key="item.jobCode" :value="item.jobCode" :label="item.jobName" />
            </el-select>
          </el-form-item>
          <!-- <el-form-item label="年龄">
            <el-input v-model="form.age" style="width: 370px;" />
          </el-form-item> -->
          <el-form-item label="性别">
            <!-- <el-input v-model="form.sex" style="width: 370px;" /> -->
            <el-select v-model="form.sex" style="width:370px">
              <el-option :value="0" label="男" />
              <el-option :value="1" label="女" />
            </el-select>
          </el-form-item>
          <el-form-item label="工资">
            <el-input v-model="form.salary" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="身份证号">
            <el-input v-model="form.idcard" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="补贴">
            <el-input v-model="form.subsidy" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="状态">
            <el-select v-model="form.status" style="width:370px">
              <el-option :value="0" label="在职" />
              <el-option :value="1" label="离职" />
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button
            :loading="crud.cu === 2"
            type="primary"
            @click="crud.submitCU"
          >确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table
        ref="table"
        v-loading="crud.loading"
        :data="crud.data"
        size="small"
        style="width: 100%;"
        @selection-change="crud.selectionChangeHandler"
      >
        <el-table-column type="selection" width="55" />
        <el-table-column prop="code" label="员工编号" />
        <el-table-column prop="name" label="员工姓名" />
        <el-table-column prop="employeeJob" label="岗位名称">
          <template slot-scope="scope">
            <span>{{
              scope.row.employeeJob ? scope.row.employeeJob.jobName : null
            }}</span>
          </template>
        </el-table-column>
        <!-- <el-table-column prop="age" label="年龄" /> -->
        <el-table-column prop="sex" label="性别">
          <template slot-scope="scope">
            <span>{{ scope.row.sex === 0 ? "男" : "女" }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="salary" label="工资" />
        <el-table-column prop="idcard" label="身份证号" />
        <el-table-column prop="subsidy" label="补贴" />
        <el-table-column prop="status" label="状态">
          <template slot-scope="scope">
            <span>{{ scope.row.status === 0 ? "在职" : "离职" }}</span>
          </template>
        </el-table-column>
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
        <el-table-column
          v-permission="['admin', 'employee:edit', 'employee:del']"
          label="操作"
          width="150px"
          align="center"
        >
          <template slot-scope="scope">
            <udOperation :data="scope.row" :permission="permission" />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudEmployee from '@/api/employee'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'
import request from '@/utils/request'

const defaultForm = {
  code: null,
  name: null,
  employeeJob: { jobCode: null },
  age: null,
  sex: null,
  salary: null,
  idcard: null,
  subsidy: null,
  status: null
}
export default {
  name: 'Employee',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  cruds() {
    return CRUD({
      title: '员工管理',
      url: 'api/employee',
      idField: 'code',
      sort: 'code,desc',
      crudMethod: { ...crudEmployee }
    })
  },
  data() {
    return {
      permission: {
        add: ['admin', 'employee:add'],
        edit: ['admin', 'employee:edit'],
        del: ['admin', 'employee:del']
      },
      rules: {
        code: [
          { required: true, message: '员工编号不能为空', trigger: 'blur' }
        ],
        name: [
          { required: true, message: '员工姓名不能为空', trigger: 'blur' }
        ],
        'employeeJob.jobCode': [
          {
            required: true,
            message: '岗位不能为空',
            trigger: 'blur'
          }
        ]
      },
      queryTypeOptions: [
        { key: 'code', display_name: '员工编号' },
        { key: 'name', display_name: '员工姓名' },
        { key: 'status', display_name: '状态' }
      ],
      employeeJobs: []
    }
  },
  created() {
    // 获取所有岗位
    request({
      url: 'api/employeeJob?page=0&size=999',
      method: 'get'
    }).then(res => {
      console.log(res)
      this.employeeJobs = res.content
    })
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped></style>
