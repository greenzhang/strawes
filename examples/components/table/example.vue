<template>
  <mn-scroller>
    <mw-container>
      <mn-section>
        <mw-crumb></mw-crumb>
      </mn-section>

      <mn-section>
        <table-search></table-search>

        <mw-alert-bar theme="info" hideIcon>
          符合条件的结果共 “{{ totalItems }}” 项
        </mw-alert-bar>

        <mw-table-group>
          <mw-table
            :items="calcTableItems(tableItems)"
            :columns="tableColumns"
            :selections.sync="selections">
            <template scope="scope" slot="cover">
              <img :src="scope.item.cover" alt="scope.cover.title" height="80" style="display: block;">
            </template>
            <template scope="scope" slot="tags">
              <mn-tag bg="#ddd" v-for="(tag, key) in scope.item.tags" :key="key">{{ tag }}</mn-tag>
            </template>
            <template scope="scope" slot="actions">
              <mw-link-bar :actions="scope.item.actions"></mw-link-bar>
            </template>
          </mw-table>

          <template slot="action">
            <mn-btn theme="primary" size="sm">新建影片</mn-btn>
            <mn-btn theme="secondary-link" size="sm">导出 EXCEL</mn-btn>
            <mn-btn theme="secondary-link" size="sm">批量导入</mn-btn>
          </template>

          <template slot="view">
            <mw-limit :limit="models.limit" :limitOptions="[10, 20, 50]"></mw-limit>
          </template>

          <template slot="paginate">
            <mw-paginate
              :totalPages="totalPages"
              :currentPage="models.page"></mw-paginate>
          </template>
        </mw-table-group>
      </mn-section>
    </mw-container>
  </mn-scroller>
</template>

<script>
  import tableSearch from './_search'
  import tag from 'vue-human/suites/tag'
  import tableColumns from './tableColumns'
  import calcTableItem from './calcTableItem'
  import Q from 'vue-human/utils/Query'
  import { listMovies } from '../../axios/movies'

  const defaultModels = {
    page: 1,
    limit: 10
  }

  export default {
    components: {
      ...tag.map(),
      tableSearch
    },
    data () {
      return {
        // 列
        tableColumns,
        // 数据
        tableItems: undefined,
        // 多选存储
        selections: [],
        // 总条数
        totalItems: 0,
        // 总页数
        totalPages: 1,
        // 请求关键词
        models: Q.merge({}, defaultModels)
      }
    },
    methods: {
      // 设置 undefined，显示为加载状态
      // 请求服务器数据
      // 合并表格数据
      // 合并总条数
      async listMovies () {
        // 设置为加载状态
        this.tableItems = undefined

        // 合并 url query 至 data
        this.models = Q.merge(defaultModels, Q.parse(this.$route.query))

        // 请求数据
        const response = await listMovies(this.models)
        this.tableItems = response.data.subjects
        this.totalItems = response.data.page.totalItems
        this.totalPages = response.data.page.totalPages
      },
      // 计算 tableItems，使其符合 tableColumn 列的要求
      calcTableItems (items) {
        if (Array.isArray(items)) return items.map(calcTableItem)
      }
    },
    watch: {
      '$route.query' () {
        this.listMovies()
      }
    },
    // 初始化数据
    created () {
      this.listMovies()
    }
  }
</script>
