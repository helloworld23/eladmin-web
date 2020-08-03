<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">名称</label>
        <el-input v-model="query.name" clearable placeholder="名称" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">原因</label>
        <el-input v-model="query.reason" clearable placeholder="原因" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">支付状态</label>
        <el-input v-model="query.status" clearable placeholder="支付状态" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker
          v-model="query.money"
          start-placeholder="moneyStart"
          end-placeholder="moneyStart"
          class="date-item"
        />
        <date-range-picker
          v-model="query.createTime"
          start-placeholder="createTimeStart"
          end-placeholder="createTimeStart"
          class="date-item"
        />
        <date-range-picker
          v-model="query.updateTime"
          start-placeholder="updateTimeStart"
          end-placeholder="updateTimeStart"
          class="date-item"
        />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="名称">
            <el-input v-model="form.name" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="原因">
            <el-input v-model="form.reason" :rows="3" type="textarea" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="金额">
            <el-input v-model="form.money" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="创建时间">
            <el-date-picker v-model="form.createTime" type="datetime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="更新时间">
            <el-date-picker v-model="form.updateTime" type="datetime" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="支付状态">
            <el-select v-model="form.status" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.pay_status"
                :key="item.id"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
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
        <el-table-column prop="name" label="名称" />
        <el-table-column prop="reason" label="原因" />
        <el-table-column prop="money" label="金额" />
        <el-table-column prop="createTime" label="创建时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.createTime) }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="updateTime" label="更新时间">
          <template slot-scope="scope">
            <span>{{ parseTime(scope.row.updateTime) }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="status" label="支付状态">
          <template slot-scope="scope">
            {{ dict.label.pay_status[scope.row.status] }}
          </template>
        </el-table-column>
        <el-table-column v-permission="['admin','busiCredit:edit','busiCredit:del']" label="操作" width="150px" align="center">
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
import crudBusiCredit from '@/api/system/busiCredit'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { creditId: null, name: null, reason: null, money: null, createTime: null, updateTime: null, status: null }
export default {
  name: 'BusiCredit',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['pay_status'],
  cruds() {
    return CRUD({ title: 'credit', url: 'api/busiCredit', sort: 'creditId,desc', crudMethod: { ...crudBusiCredit }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'busiCredit:add'],
        edit: ['admin', 'busiCredit:edit'],
        del: ['admin', 'busiCredit:del']
      },
      rules: {
        creditId: [
          { required: true, message: 'ID不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'name', display_name: '名称' },
        { key: 'reason', display_name: '原因' },
        { key: 'status', display_name: '支付状态' }
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
