<template>
  <page-header-wrapper>
    <a-card :bordered="false">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="48">
            <a-col :md="8" :sm="24">
              <a-form-item label="path">
                <a-input v-model="queryParam.path" placeholder="按path查找"/>
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <a-form-item label="类型">
                <a-select v-model="queryParam.path_type" placeholder="按path类型查找" default-value="0">
                  <a-select-option value="1">menu</a-select-option>
                  <a-select-option value="2">api</a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <template v-if="advanced">
              <a-col :md="8" :sm="24">
                <a-form-item label="名称">
                  <a-input v-model="queryParam.name" placeholder="按name查找"/>
                </a-form-item>
              </a-col>
            </template>
            <a-col :md="!advanced && 8 || 24" :sm="24">
              <span class="table-page-search-submitButtons" :style="advanced && { float: 'right', overflow: 'hidden' } || {} ">
                <a-button type="primary" @click="$refs.table.refresh(true)">查询</a-button>
                <a-button style="margin-left: 8px" @click="() => this.queryParam = {}">重置</a-button>
                <a @click="toggleAdvanced" style="margin-left: 8px">
                  {{ advanced ? '收起' : '展开' }}
                  <a-icon :type="advanced ? 'up' : 'down'"/>
                </a>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>

      <div class="table-operator">
        <a-button type="primary" v-if="$shareAuth('/rbac/path.add')" icon="plus" @click="handleEdit(0)">新建</a-button>

      </div>

      <s-table
        ref="table"
        size="default"
        rowKey="id"
        :columns="columns"
        :data="loadData"
        showPagination="auto"
      >
        <span slot="serial" slot-scope="text, record, index">
          {{ index + 1 }}
        </span>

        <span slot="action" slot-scope="text, record">
          <template>
            <a v-if="$shareAuth('/rbac/path.edit')" @click="handleEdit(record.id)">修改</a>
            <a-divider type="vertical" />
          </template>
          <a-dropdown v-if="$shareAuth('/rbac/path.delete')">
            <a class="ant-dropdown-link">
              更多 <a-icon type="down" />
            </a>
            <a-menu slot="overlay">

              <a-menu-item v-if="$shareAuth('/rbac/path.delete')">
                <a href="javascript:;">删除</a>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </s-table>

    </a-card>
  </page-header-wrapper>
</template>

<script>
	import moment from 'moment'
	import { STable, Ellipsis } from '@/components'
	import { userList } from '@/api/rbac/user'

	const columns = [
		{
			title: 'account',
			dataIndex: 'account'
		},
		{
			title: 'name',
			dataIndex: 'name'
		},
		{
			title: '状态',
			dataIndex: 'status',
			needTotal: true,
			customRender: t => {
				if (t === 1) {
					return '启用'
				} else {
					return '禁用'
				}
			}
		},
		{
			title: '更新时间',
			dataIndex: 'updatedAt'
		},
		{
			title: '操作',
			dataIndex: 'action',
			width: '150px',
			scopedSlots: { customRender: 'action' }
		}
	]
	export default {
		name: 'PathList',
		components: {
			STable,
			Ellipsis

		},
		data () {
			this.columns = columns
			return {
				// create model
				visible: false,
				confirmLoading: false,
				mdl: null,
				// 高级搜索 展开/关闭
				advanced: false,
				// 查询参数
				queryParam: {},
				// 加载数据方法 必须为 Promise 对象
				loadData: parameter => {
					return userList(parameter, this.queryParam)
						.then(res => {
							return res.result
						})
				}

			}
		},

		created () {

		},

		methods: {

			toggleAdvanced () {
				this.advanced = !this.advanced
			},
			resetSearchForm () {
				this.queryParam = {
					date: moment(new Date())
				}
			},
			handleEdit (id) {
				if (id) {
					this.$router.push({ path: '/rbac/path/edit', query: { id: id } })
				} else {
					this.$router.push({ path: '/rbac/path/edit' })
				}
			}
		}
	}
</script>
