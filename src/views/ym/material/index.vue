<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <el-input v-model="query.materialCode" clearable placeholder="原料编码" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <el-input v-model="query.materialName" clearable placeholder="原料名称" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <el-input v-model="query.materialInci" clearable placeholder="INCI" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <el-input v-model="query.materialPrice" clearable placeholder="价格" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <el-input v-model="query.materialType" clearable placeholder="分类" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="原料编码">
            <el-input v-model="form.materialCode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="原料名称">
            <el-input v-model="form.materialName" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="INCI">
            <el-input v-model="form.materialInci" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="价格">
            <el-input v-model="form.materialPrice" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="分类">
            <el-select v-model="form.materialType" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.material_type"
                :key="item.id"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="供应商">
            <el-input v-model="form.materialProvider" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="厂家">
            <el-input v-model="form.materialVender" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="效果">
            <el-input v-model="form.materialEffect" :rows="3" type="textarea" style="width: 370px;" />
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
        <el-table-column prop="materialCode" label="原料编码" />
        <el-table-column prop="materialName" label="原料名称" />
        <el-table-column prop="materialInci" label="INCI" />
        <el-table-column prop="materialPrice" label="价格" />
        <el-table-column prop="materialType" label="分类">
          <template slot-scope="scope">
            {{ dict.label.material_type[scope.row.materialType] }}
          </template>
        </el-table-column>
        <el-table-column prop="materialProvider" label="供应商" />
        <el-table-column prop="materialVender" label="厂家" />
        <el-table-column prop="materialEffect" label="效果" />
        <el-table-column v-permission="['admin','material:edit','material:del']" label="操作" width="150px" align="center">
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
import crudYmMaterial from '@/api/ym/ymMaterial'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { pkMaterial: null, materialCode: null, materialName: null, materialInci: null, materialPrice: null, materialType: null, materialProvider: null, materialVender: null, materialEffect: null, createTime: null, updateTime: null }
export default {
  name: 'YmMaterial',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['material_type'],
  cruds() {
    return CRUD({ title: '原料管理', url: 'api/ymMaterial', idField: 'pkMaterial', sort: 'pkMaterial,desc', crudMethod: { ...crudYmMaterial }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'material:add'],
        edit: ['admin', 'material:edit'],
        del: ['admin', 'material:del']
      },
      rules: {
      },
      queryTypeOptions: [
        { key: 'materialCode', display_name: '原料编码' },
        { key: 'materialName', display_name: '原料名称' },
        { key: 'materialInci', display_name: 'INCI' },
        { key: 'materialPrice', display_name: '价格' },
        { key: 'materialType', display_name: '分类' },
        { key: 'materialProvider', display_name: '供应商' },
        { key: 'materialVender', display_name: '厂商' },
        { key: 'materialEffect', display_name: '功效' }
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
