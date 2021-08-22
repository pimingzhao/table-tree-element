<!--
 * @Author: pimzh
 * @Date: 2021-08-19 19:17:39
 * @LastEditTime: 2021-08-22 15:37:28
 * @LastEditors: pimzh
 * @Description:
-->
<script>
import RenderTree from './render-tree'

export default {
  name: 'TableTree',
  components: { RenderTree },
  props: {
    data: {
      type: Array,
      default: () => []
    },
    rowClassName: {
      type: Function,
      default: () => {}
    },
    cellClassName: {
      type: Function,
      default: () => {}
    }
  },
  data() {
    return {
      tableData: [],
      hiddenRows: [],
      treeIndex: -1
    }
  },
  watch: {
    data: {
      handler(val) {
        this.treeIndex = -1
        this.hiddenRows = []
        this.tableData = this.formatData(val)
      },
      immediate: true
    }
  },
  mounted() {
    // bind table methods to table tree
    const table = this.$refs.table
    const tableMethods = table.$options.methods
    Object.keys(tableMethods).forEach(key => {
      this[key] =  tableMethods[key].bind(table)
    })
  },
  methods: {
    createTree(h, props, i) {
      this.treeIndex = i
      const { onIconClick } = this
      return h('el-table-column', {
        props,
        scopedSlots: {
          default: ({ row }) => h('render-tree', {
            props: { data: row, onIconClick }
          })
        }
      })
    },
    // 展开关闭树节点时触发
    onIconClick(node) {
      if (node.expand) {
        node.expand = false
        this.loopTree(node.children, (item, i, p) => {
          item.__expand = p.expand
          if (p.expand || !item.__expand) {
            !this.hiddenRows.includes(item.__id) && this.hiddenRows.push(item.__id)
          }
        }, node)
      } else {
        node.expand = true
        this.loopTree(node.children, (item, i, p) => {
          item.__expand = p.expand
          if (item.__expand) {
            this.hiddenRows.splice(this.hiddenRows.indexOf(item.__id), 1)
          }
        }, node)
      }
    },
    // 合并 props row-class-name 与当前组件的 rowClassName
    getRowCls(data) {
      return (
        this.hiddenRows.includes(data.row.__id) ? 'row-hidden ' : ''
      ) + (this.rowClassName(data) || '')
    },
    // 同 rowClassName
    getCellCls(data) {
      return (
        data.columnIndex === this.treeIndex ? 'no-padding ' : ''
      ) + (this.cellClassName(data) || '')
    },
    // 先序遍历树
    loopTree(data, cb, p) {
      data.forEach((item, i) => {
        cb(item, i, p)
        if (item.children?.length) {
          this.loopTree(item.children, cb, item)
        }
      })
    },
    // 是否为一个Object
    isObj(data) {
      return Object.prototype.toString.call(data) === '[object Object]'
    },
    // 深拷贝数据
    deepClone(data) {
      if (Array.isArray(data)) {
        return data.map(item => this.deepClone(item))
      }
      if (this.isObj(data)) {
        const o = {}
        for (const key in data) {
          o[key] = this.deepClone(data[key])
        }
        return o
      }
      return data
    },
    // 数据转换
    formatData(data) {
      const arr = []
      // 通过深拷贝不污染原数据集
      this.loopTree(this.deepClone(data), (item, i, p) => {
        // 节点层级
        item.__level = (p ? p.__level : -1) + 1
        // 节点唯一标识
        item.__id = (p?.__id || 'x') + '_' + i
        item.children?.length && (item.expand = item.expand || false)
        if (p) {
          // 初始化节点不展开
          !p.expand && this.hiddenRows.push(item.__id)
          item.__expand = p.expand
          p.children[p.children.length - 1].__isLast = true
        }
        arr.push(item)
      })
      arr.forEach(item => item.children?.length && (item.expand = item.expand || false))
      arr.length && (arr[arr.length - 1].__isLast = true)
      return arr
    }
  },
  render(h) {
    return (
      <el-table
        ref="table"
        row-class-name={this.getRowCls}
        cell-class-name={this.getCellCls}
        data={this.tableData}
        props={this.$attrs}
        on={this.$listeners}
      >
        {this.$slots.default.map((item, i) => {
          if (!item.tag) {
            return ""
          }
          if (item.componentOptions.propsData?.type === "tree") {
            return this.createTree(h, item.componentOptions.propsData, i)
          }
          return item
        })}
      </el-table>
    )
  }
}
</script>

<style lang="scss" scoped>
::v-deep.el-table {
  .row-hidden {
    display: none;
  }
  .no-padding {
    padding: 0;
    .cell {
      padding: 0;
      height: 50px;
      display: flex;
    }
  }
}
</style>
