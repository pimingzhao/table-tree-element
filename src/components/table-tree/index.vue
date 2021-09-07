<!--
 * @Author: pimzh
 * @Date: 2021-08-19 19:17:39
 * @LastEditTime: 2021-09-07 18:36:39
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
    },
    childrenKey: {
      type: String,
      default: 'children'
    }
  },
  data() {
    return {
      tableData: [],
      hiddenRows: [],
      treeIndex: -1,
      childrenMap: Object.create(null) // parent 与 chlid 之间的映射关系
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
    createTree(h, props, i) {
      this.treeIndex = i
      const { onIconClick, childrenKey } = this
      return h('el-table-column', {
        props,
        scopedSlots: {
          default: ({ row }) => h('render-tree', {
            props: { data: row, onIconClick, childrenKey }
          })
        }
      })
    },
    // 展开关闭树节点时触发
    onIconClick(node) {
      const childrenMap = this.childrenMap
      const tableData = this.tableData
      const cids = [...childrenMap[node.__id]]
      // 关闭节点
      if (node.expand) {
        while (cids.length) {
          const child = tableData[cids.pop()]
          this.hiddenRows.push(child.__id)
          if (child.expand && child.__id in childrenMap) {
            cids.push(...childrenMap[child.__id])
          }
        }
        node.expand = false
      } else {
        // 展开节点
        while (cids.length) {
          const child = tableData[cids.pop()]
          this.hiddenRows.splice(
            this.hiddenRows.indexOf(child.__id),
            1
          )
          if (child.expand && child.__id in childrenMap) {
            cids.push(...childrenMap[child.__id])
          }
        }
        node.expand = true
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
    // 数据转换
    formatData(data) {
      const tableData = []
      const childrenMap = this.childrenMap
      const childrenKey = this.childrenKey
      let count = 0

      const loopTree = (arr, p) => {
        arr.forEach((item, i) => {
          // 节点层级
          item.__level = (p ? p.__level : -1) + 1
          // 节点唯一标识
          item.__id = (p?.__id || 'x') + '_' + i
          if (p) {
            !p.expand && this.hiddenRows.push(item.__id)
            p[childrenKey][p[childrenKey].length - 1].__isLast = true
            childrenMap[p.__id].push(count)
          }
          count++
          tableData.push(item)
          if (item[childrenKey]?.length) {
            item.expand = item.expand || false
            childrenMap[item.__id] = []
            // loop children
            loopTree(item[childrenKey], item)
          } else {
            // 是否应该是 item.expand = false
            delete item.expand
          }
        })
      }
      // 通过深拷贝不污染原数据集
      loopTree(this.deepClone(data))
      tableData.length && (tableData[tableData.length - 1].__isLast = true)
      this.childrenMap = childrenMap
      return tableData
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
