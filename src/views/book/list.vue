<template>
  <div class="app-container">
    <div class="filter-container"><!--查询条件-->
      <el-input
        v-model="listQuery.title"
        placeholder="书名"
        style="width: 200px"
        class="filter-item"
        clearable
        @keyup.enter.native="handleFilter"
        @clear="handleFilter"
        @blur="handleFilter"
      />
      <el-input
        v-model="listQuery.author"
        placeholder="作者"
        style="width: 200px"
        class="filter-item"
        clearable
        @keyup.enter.native="handleFilter"
        @clear="handleFilter"
        @blur="handleFilter"
      />
      <el-select
        v-model="listQuery.category"
        placeholder="分类"
        style="width: 200px"
        class="filter-item"
        clearable
        @change="handleFilter"
      >
        <el-option
          v-for="item in categoryList"
          :key="item.value"
          :label="item.label + '('+ item.num + ')'"
          :value="item.label"
        />

      </el-select>
      <el-button
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        style="margin-left: 10px"
        @click="handleFilter"
      >查询
      </el-button>
      <el-button
        class="filter-item"
        type="primary"
        icon="el-icon-edit"
        style="margin-left: 5px"
        @click="handleCreate"
      >新增
      </el-button>
      <el-checkbox
        v-model="showCover"
        class="filter-item"
        style="margin-left: 5px"
        @change="changeShowCover"
      >显示封面
      </el-checkbox>
    </div>
    <el-table
      :key="tableKey"
      :v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%"
      :default-sort="defaultSort"
      @sort-change="sortChange"
    >
      <el-table-column
        label="ID"
        prop="id"
        sortable="custom"
        align="center"
        width="80"
      />
      <el-table-column
        label="书名"
        align="center"
        width="150"
      >
        <template slot-scope="{row: { titleWrapper }}">
          <span v-html="titleWrapper" />
        </template>
      </el-table-column>
      <el-table-column
        label="作者"
        align="center"
        width="150"
      >
        <template slot-scope="{row: { authorWrapper }}">
          <span v-html="authorWrapper" />
        </template>
      </el-table-column>
      <el-table-column
        label="出版社"
        prop="publish"
        align="center"
        width="150"
      />
      <el-table-column
        label="分类"
        prop="categoryText"
        align="center"
        width="100"
      />
      <el-table-column
        label="语言"
        prop="language"
        align="center"
      />
      <el-table-column
        v-if="showCover"
        label="封面"
        align="center"
        width="150"
      >
        <template slot-scope="{row :{ cover }}">
          <a :href="cover" target="_blank">
            <img :src="cover" alt="" style="width: 120px;height: 180px;">
          </a>
        </template>
      </el-table-column>
      <el-table-column
        label="文件名"
        prop="fileName"
        align="center"
        width="100"
      />
      <el-table-column
        label="文件路径"
        prop="filePath"
        align="center"
        width="100"
      />
      <el-table-column
        label="封面路径"
        prop="coverPath"
        align="center"
        width="100"
      />
      <el-table-column
        label="解压路径"
        prop="unzipPath"
        align="center"
        width="100"
      />
      <el-table-column
        label="上传人"
        prop="createUser"
        align="center"
        width="100"
      />
      <el-table-column
        label="上传时间"
        prop="createDt"
        align="center"
        width="100"
      >
        <template slot-scope="{row: {createDt}}">
          <span>{{ createDt | timeFilter }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        width="120"
        fixed="right"
      >
        <template slot-scope="{ row }">
          <el-button
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(row)"
          />
          <el-button
            type="text"
            icon="el-icon-delete"
            style="color:#f56c6c"
            @click="handleDelete(row)"
          />
        </template>
      </el-table-column>
    </el-table>
    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.pageSize"
      @pagination="refresh"
    /><!--框架自带（页面跳转信息）-->
  </div>
</template>

<script>
import Pagination from '../../components/Pagination/index'
import { getCategory, listBook, deleteBook } from "../../api/book";
import { parseTime } from "../../utils/index.js";

export default {
  name: "list",
  components: {
    Pagination
  },
  filters: {
    timeFilter(time) {
      return time ? parseTime(time, '{y}-{m}-{d} {h}:{i}') : '无'
    }
  },
  data() {
    return {
      tableKey: 0,
      listLoading: true,
      list: [],
      listQuery: {},
      showCover: false,
      categoryList: [],
      total: 0,
      defaultSort: {}
    }
  },
  mounted() {
    this.getList()
    this.getCategoryList()
  },
  beforeRouteUpdate(to, from, next) {
    if (to.path === from.path) {
      const newQuery = Object.assign({}, to.query)
      const oldQuery = Object.assign({}, from.query)
      if (JSON.stringify(newQuery) !== JSON.stringify(oldQuery)) {
        this.getList()
      }
    }
    next()
  },
  created() {
    this.parseQuery()
  },
  methods: {
    refresh() {
      this.$router.push({
        path: '/book/list',
        query: this.listQuery
      })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.refresh()
      // this.getList()
    },
    handleCreate() {
      this.$router.push('/book/create')
    },
    handleUpdate(row) {
      this.$router.push(`/book/edit/${row.fileName}`)
    },
    handleDelete(row) {
      this.$confirm('将要永久删除该电子书吗，是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消'
      }).then(() => {
        deleteBook(row.fileName).then(response => {
          this.$notify({
            title: '成功',
            message: response.msg || '删除成功',
            type: 'success',
            duration: 2000
          })
        })
      })
    },
    changeShowCover(value) {
      this.showCover = value
    },
    getCategoryList() {
      getCategory().then(response => { // 向后端请求数据
        this.categoryList = response.data
      })
    },
    sortChange(data) {
      const { prop, order } = data
      this.sortBy(prop, order)
    },
    sortBy(prop, order) {
      if (order === 'ascending') {
        this.listQuery.sort = `+${prop}`
      } else {
        this.listQuery.sort = `-${prop}`
      }
      this.handleFilter()
    },
    getList() {
      this.listLoading = true
      listBook(this.listQuery).then(response => { // 向后端请求数据
        const { list, count } = response.data
        this.list = list
        this.total = count
        this.listLoading = false
        // 查询字高亮
        this.list.forEach(book => {
          book.titleWrapper = this.wrapperKeyword('title', book.title)
          book.authorWrapper = this.wrapperKeyword('author', book.author)
        })
      })
    },
    parseQuery() {
      const query = Object.assign({}, this.$route.query)// 获取页面地址栏信息，优化刷新后操作
      let sort = '+id'
      const listQuery = {
        page: 1,
        pageSize: 20,
        sort: '+id'
      }
      if (query) {
        query.page = +query.page
        query.pageSize = +query.pageSize
        sort = query.sort
      }
      const sortSymbol = sort[0]
      const sortColumn = sort.slice(1, sort.length)
      this.defaultSort = {
        prop: sortColumn,
        order: sortSymbol === '+' ? 'ascending' : 'descending'
      }
      this.listQuery = { ...listQuery, ...query }
    },
    wrapperKeyword(key, value) {
      function hightLight(value) {
        return `<span style="color:#1890ff">${value}</span>`
      }
      if (!this.listQuery[key]) {
        return value
      } else {
        // 正则表达式  ig为不区分大小写和全局搜索
        return value.replace(new RegExp(this.listQuery[key], 'ig'), value => hightLight(value))
      }
    }
  },
}
</script>

<style scoped>

</style>
