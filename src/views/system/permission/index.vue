<template>
  <div class="app-container">
    <search :permissions="permissions" :query="query"/>
    <!--表格渲染-->
    <tree-table v-loading="loading" :data="data" :expand-all="true" :columns="columns" border size="small">
      <el-table-column prop="createTime" label="创建日期">
        <template slot-scope="scope">
          <span>{{ time(scope.row.createTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="150px" align="center">
        <template slot-scope="scope">
          <edit v-if="checkPermission(['ADMIN','PERMISSION_ALL','PERMISSION_EDIT'])" :permissions="permissions" :data="scope.row" :sup_this="sup_this"/>
          <el-popover
            v-if="checkPermission(['ADMIN','PERMISSION_ALL','PERMISSION_DELETE'])"
            v-model="scope.row.delPopover"
            placement="top"
            width="200">
            <p>确定删除吗,如果存在下级节点则一并删除，此操作不能撤销！</p>
            <div style="text-align: right; margin: 0">
              <el-button size="mini" type="text" @click="scope.row.delPopover = false">取消</el-button>
              <el-button :loading="delLoading" type="primary" size="mini" @click="subDelete(scope.$index, scope.row)">确定</el-button>
            </div>
            <el-button slot="reference" type="danger" size="mini" @click="scope.row.delPopover = true">删除</el-button>
          </el-popover>
        </template>
      </el-table-column>
    </tree-table>
  </div>
</template>

<script>
import checkPermission from '@/utils/permission' // 权限判断函数
import treeTable from '@/components/TreeTable'
import initData from '../../../mixins/initData'
import { del } from '@/api/permission'
import { getPermissionTree } from '@/api/permission'
import { parseTime } from '@/utils/index'
import search from './module/search'
import edit from './module/edit'
export default {
  components: { search, edit, treeTable },
  mixins: [initData],
  data() {
    return {
      columns: [
        {
          text: '名称',
          value: 'name'
        },
        {
          text: '别名',
          value: 'alias'
        }
      ],
      delLoading: false, sup_this: this, permissions: []
    }
  },
  created() {
    this.getPermissions()
    this.$nextTick(() => {
      this.init()
    })
  },
  methods: {
    checkPermission,
    beforeInit() {
      this.url = 'api/permissions'
      const sort = 'id,desc'
      const query = this.query
      const value = query.value
      this.params = { page: this.page, size: this.size, sort: sort }
      if (value) { this.params['name'] = value }
      return true
    },
    subDelete(index, row) {
      this.delLoading = true
      del(row.id).then(res => {
        this.delLoading = false
        row.delPopover = false
        this.init(search.data().query)
        this.$notify({
          title: '删除成功',
          type: 'success',
          duration: 2500
        })
      })
    },
    time(time) {
      return parseTime(time)
    },
    getPermissions() {
      getPermissionTree().then(res => {
        this.permissions = []
        const permission = { id: 0, label: '顶级类目', children: [] }
        permission.children = res
        this.permissions.push(permission)
      })
    }
  }
}
</script>

<style scoped>

</style>
