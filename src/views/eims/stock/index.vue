<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">产品编码</label>
        <el-input v-model="query.productCode" clearable placeholder="产品编码" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">产品名称</label>
        <el-input v-model="query.productName" clearable placeholder="产品名称" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <!-- <el-form-item label="产品编码">
            <el-input v-model="form.product.productCode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="产品名称">
            <el-input v-model="form.product.productName" style="width: 370px;" />
          </el-form-item> -->
          <el-form-item label="产品编码" prop="productCode">
            <el-select v-model="form.productCode" filterable style="width:370px">
              <el-option v-for="item in productArr" :key="item.productCode" :label="item.productCode" :value="item.productCode" />
            </el-select>
          </el-form-item>
          <el-form-item label="单位">
            <el-input v-model="form.unit" style="width: 370px;" disabled />
          </el-form-item>
          <el-form-item label="总重量(吨)">
            <el-input v-model="form.totalWeight" style="width: 370px;" />
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
        <el-table-column prop="product.productCode" label="产品编码" />
        <el-table-column prop="product.productName" label="产品名称" />
        <el-table-column prop="unit" label="单位" />
        <el-table-column prop="totalWeight" label="总重量(吨)" />
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
        <el-table-column v-permission="['admin','stock:edit','stock:del']" label="操作" width="150px" align="center">
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
import crudStock from '@/api/stock'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'
import request from '@/utils/request'

const defaultForm = { id: null, productCode: null, unit: '吨', totalWeight: null }
export default {
  name: 'Stock',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  cruds() {
    return CRUD({ title: '库存', url: 'api/stock', sort: 'id,desc', crudMethod: { ...crudStock }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'stock:add'],
        edit: ['admin', 'stock:edit'],
        del: ['admin', 'stock:del']
      },
      rules: {
        productCode: [
          { required: true, message: '产品编号不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'productCode', display_name: '产品编码' },
        { key: 'productName', display_name: '产品名称' }
      ],
      productArr: []
    }
  },
  created() {
    request({
      url: 'api/product?page=0&size=999',
      method: 'get'
    }).then(res => {
      this.productArr = res.content
      console.log(this.productArr)
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

<style scoped>

</style>
