<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">id</label>
        <el-input v-model="query.id" clearable placeholder="id" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">状态</label>
        <el-input v-model="query.status" clearable placeholder="状态" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">币种</label>
        <el-input v-model="query.currency" clearable placeholder="币种" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">用户名</label>
        <el-input v-model="query.username" clearable placeholder="用户名" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
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
          <el-form-item label="内部单号" prop="orderId">
            <el-input v-model="form.orderId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="三方单号" prop="platOrderId">
            <el-input v-model="form.platOrderId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="UID" prop="uid">
            <el-input v-model="form.uid" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="钱包id" prop="walletId">
            <el-input v-model="form.walletId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="租户id" prop="merchantId">
            <el-input v-model="form.merchantId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="状态" prop="status">
            <el-select v-model="form.status" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.deposit_status"
                :key="item.id"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="币种" prop="currency">
            <el-select v-model="form.currency" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.currency"
                :key="item.id"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="充值金额" prop="payAmount">
            <el-input v-model="form.payAmount" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="汇率">
            <el-input v-model="form.exchangeRate" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="到账金额">
            <el-input v-model="form.realAmount" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="渠道手续费">
            <el-input v-model="form.channelFee" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="渠道编码" prop="channelCode">
            <el-input v-model="form.channelCode" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="渠道名称">
            <el-input v-model="form.channelName" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="用户名">
            <el-input v-model="form.username" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="备注">
            <el-input v-model="form.mark" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="创建时间">
            <el-date-picker v-model="form.createTime" type="datetime" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="id" label="id" />
        <el-table-column prop="orderId" label="内部单号" />
        <el-table-column prop="platOrderId" label="三方单号" />
        <el-table-column prop="uid" label="UID" />
        <el-table-column prop="walletId" label="钱包id" />
        <el-table-column prop="merchantId" label="租户id" />
        <el-table-column prop="status" label="状态">
          <template slot-scope="scope">
            {{ dict.label.deposit_status[scope.row.status] }}
          </template>
        </el-table-column>
        <el-table-column prop="currency" label="币种">
          <template slot-scope="scope">
            {{ dict.label.currency[scope.row.currency] }}
          </template>
        </el-table-column>
        <el-table-column prop="payAmount" label="充值金额" />
        <el-table-column prop="exchangeRate" label="汇率" />
        <el-table-column prop="realAmount" label="到账金额" />
        <el-table-column prop="channelFee" label="渠道手续费" />
        <el-table-column prop="channelCode" label="渠道编码" />
        <el-table-column prop="channelName" label="渠道名称" />
        <el-table-column prop="username" label="用户名" />
        <el-table-column prop="mark" label="备注" />
        <el-table-column prop="createTime" label="创建时间" />
        <el-table-column prop="updateTime" label="修改时间" />
        <el-table-column v-if="checkPer(['admin','payDeposit:edit','payDeposit:del'])" label="操作" width="150px" align="center">
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
import crudPayDeposit from '@/api/payDeposit'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'
import DateRangePicker from '@/components/DateRangePicker/index.vue'

const defaultForm = { id: null, orderId: null, platOrderId: null, uid: null, walletId: null, merchantId: null, status: null, currency: null, payAmount: null, exchangeRate: null, realAmount: null, channelFee: null, channelCode: null, channelName: null, username: null, mark: null, createTime: null, updateTime: null }
export default {
  name: 'PayDeposit',
  components: { DateRangePicker, pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['deposit_status', 'currency'],
  cruds() {
    return CRUD({ title: 'payment', url: 'api/payDeposit', idField: 'id', sort: 'id,desc', crudMethod: { ...crudPayDeposit }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'payDeposit:add'],
        edit: ['admin', 'payDeposit:edit'],
        del: ['admin', 'payDeposit:del']
      },
      rules: {
        orderId: [
          { required: true, message: '内部单号不能为空', trigger: 'blur' }
        ],
        platOrderId: [
          { required: true, message: '三方单号不能为空', trigger: 'blur' }
        ],
        uid: [
          { required: true, message: 'UID不能为空', trigger: 'blur' }
        ],
        walletId: [
          { required: true, message: '钱包id不能为空', trigger: 'blur' }
        ],
        merchantId: [
          { required: true, message: '租户id不能为空', trigger: 'blur' }
        ],
        status: [
          { required: true, message: '状态不能为空', trigger: 'blur' }
        ],
        currency: [
          { required: true, message: '币种不能为空', trigger: 'blur' }
        ],
        payAmount: [
          { required: true, message: '充值金额不能为空', trigger: 'blur' }
        ],
        channelCode: [
          { required: true, message: '渠道编码不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'id', display_name: 'id' },
        { key: 'status', display_name: '状态' },
        { key: 'currency', display_name: '币种' },
        { key: 'username', display_name: '用户名' }
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
