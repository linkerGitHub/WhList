<template>
  <el-row
      v-loading="listStatus.loading"
      style="position:relative;"
  >
    <slot name="listScope" :response="lastResponse" />
    <div v-if="paginationTotalPage === 0 && showEmptySlot === true" class="empty-status">
      <slot name="emptyNotification">
        <div class="default-empty">
          <div>暂无数据</div>
        </div>
      </slot>
    </div>
    <el-pagination
        v-if="!(paginationHideOnEmpty && paginationTotalPage === 0) && !paginationHideOnAnytime"
        background
        :hide-on-single-page="hideOnSinglePage"
        :current-page="pageParam.page"
        :page-sizes="[10, 20, 50, 100, 200]"
        :page-size.sync="pageParam.per_page"
        :layout="paginationLayout"
        :total="paginationTotalPage"
        class="wh-list-pagination"
        @size-change="pageSizeChangeHandle"
        @current-change="currentPageChangeHandle"
    />
  </el-row>
</template>

<script>
import Vue from 'vue'
import { Loading, Row, Pagination } from 'element-ui'
import Axios from 'axios'
Vue.use(Loading)
export default {
  name: 'WhList',
  components: {
    ElPagination: Pagination,
    ElRow: Row
  },
  props: {
    /**
     * 是否显示空提示插槽
     **/
    showEmptySlot: {
      type: Boolean,
      default() {
        return true
      }
    },
    /**
     * 总是隐藏翻页器
     ***/
    paginationHideOnAnytime: {
      type: Boolean,
      default() {
        return false
      }
    },
    /**
     * 当没有数据时隐藏翻页器
     **/
    paginationHideOnEmpty: {
      type: Boolean,
      default() {
        return true
      }
    },
    /**
     * 当数据只有一页时隐藏翻页器
     **/
    hideOnSinglePage: {
      type: Boolean,
      default() {
        return false
      }
    },
    /**
     * 翻页器layout配置
     **/
    paginationLayout: {
      type: String,
      default() {
        return 'total, prev, pager, next'
      }
    },
    /**
     * 是否使用条目模板
     **/
    useItemTemplate: {
      type: Boolean,
      default() {
        return true
      }
    },
    /**
     * 从数据源中取得总页数
     */
    getTotalPageFromResponse: {
      type: Function,
      default: (res) => {
        if (res.data && res.data.data) {
          return res.data.data.total
        }
        return 0
      }
    },
    getListData: {
      type: Function,
      default: (res) => {
        return res.data.data || res.data.data.data || res.data.data.datas || []
      }
    },
    /**
     * 数据源地址，可以是地址字符串或者axios的请求实例。如果是字符串，将会使用默认的请求实例
     **/
    dataSrc: {
      type: [String, Function],
      required: true
    },
    /**
     * 键值的键名
     **/
    keyKey: {
      type: String,
      default() {
        return 'id'
      }
    },
    /**
     * 默认翻页设置
     */
    pageSet: {
      type: Object,
      default() {
        return {
          per_page: 10,
          page: 1
        }
      }
    },
    /**
     * 是否启用加载下一页
     **/
    loadMoreEnable: {
      type: Boolean,
      default() {
        return true
      }
    },
    /**
     * 额外的参数，可以用做筛选项，参数会加入到get请求的params中
     **/
    extraParams: {
      type: Object,
      default() {
        return {}
      }
    }
  },
  data() {
    return {
      extraParamsInnerTmp: {},
      pageParam: {
        per_page: 10,
        page: 1,
        ...this.pageSet
      },
      listRequester: null,
      listStatus: {
        loading: false
      },
      lastResponse: {}
    }
  },
  computed: {
    paginationTotalPage: function() {
      return this.getTotalPageFromResponse(this.lastResponse) || 0
    }
  },
  watch: {
    dataSrc: {
      handler: function(val, oldVal) {
        if (oldVal !== val) {
          if (this.dataSrc.constructor === String) {
            if (this.listRequester === null) {
              this.listRequester = Axios.create({
                url: this.dataSrc
              })
            } else {
              this.listRequester.defaults.url = this.dataSrc
            }
          } else {
            this.listRequester = this.dataSrc
          }
          this.pageRestartRequest()
        }
      },
      immediate: true
    },
    extraParams: {
      handler: function(val) {
        const oldVal = this.extraParamsInnerTmp
        if (JSON.stringify(val) !== JSON.stringify(oldVal)) {
          this.extraParamsInnerTmp = JSON.parse(JSON.stringify(val))
          this.pageRestartRequest()
        }
      },
      deep: true
    }
  },
  beforeMount() {
    this.extraParamsInnerTmp = JSON.parse(JSON.stringify(this.extraParams))
  },
  methods: {
    pageRestartRequest() {
      this.pageParam.page = 1
      this.listDataRequest()
    },
    /**
     * 以当前参数（翻页参数，附加请求参数等）,请求表格数据
     */
    listDataRequest() {
      this.listStatus.loading = true
      this.listRequester.request({
        method: 'get',
        params: {
          ...this.pageParam,
          ...this.extraParams
        }
      }).then(res => {
        this.lastResponse = res.data
      }).finally(() => {
        this.listStatus.loading = false
      })
    },
    // 翻页大小改变的处理
    pageSizeChangeHandle(val) {
      this.pageRestartRequest()
    },
    // 翻页事件处理
    currentPageChangeHandle(val) {
      this.pageParam.page = val
      this.listDataRequest()
    }
  }
}
</script>

<style scoped>
.default-empty{
  text-align: center;
  color: #c7c7c7;
}
.wh-list-pagination{
  padding: 20px;
}
</style>
