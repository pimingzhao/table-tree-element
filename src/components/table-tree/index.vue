<!--
 * @Author: pimzh
 * @Date: 2021-08-19 19:17:39
 * @LastEditTime: 2021-08-19 22:01:27
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

    onIconClick(node) {
      if (node.expand) {
        node.expand = false
        this.loopTree(node.children, (item, i, p) => {
          if (p === node || item.__expand) {
            !this.hiddenRows.includes(item.__id) && this.hiddenRows.push(item.__id)
          }
        }, node)
      } else {
        this.$set(node, 'expand', true)
        this.loopTree(node.children, (item, i, p) => {
          item.__expand = p.expand
          if (p === node || item.__expand) {
            this.hiddenRows.splice(this.hiddenRows.indexOf(item.__id), 1)
          }
        }, node)
      }
    },

    getRowCls(data) {
      return (
        this.hiddenRows.includes(data.row.__id) ? 'row-hidden ' : ''
      ) + (this.rowClassName(data) || '')
    },
    getCellCls(data) {
      return (
        data.columnIndex === this.treeIndex ? 'no-overflow ' : ''
      ) + (this.cellClassName(data) || '')
    },

    loopTree(data, cb, p) {
      data.forEach((item, i) => {
        cb(item, i, p)
        if (item.children?.length) {
          this.loopTree(item.children, cb, item)
        }
      })
    },

    isObj(data) {
      return Object.prototype.toString.call(data) === '[object Object]'
    },
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

    formatData(data) {
      const arr = []
      this.loopTree(this.deepClone(data), (item, i, p) => {
        // 节点层级
        item.__level = (p ? p.__level : -1) + 1
        // 节点唯一标识
        item.__id = (p?.__id || 'x') + '_' + i
        item.expand = item.expand || false
        if (p) {
          // 初始化节点不展开
          !item.expand && this.hiddenRows.push(item.__id)
          item.__expand = p.expand
          p.children[p.children.length - 1].__isLast = true
        }
        arr.push(item)
      })
      arr[arr.length - 1].__isLast = true
      return arr
    }
  },
  render(h) {
    return (<el-table row-class-name={this.getRowCls} cell-class-name={this.getCellCls} data={this.tableData} props={this.$attrs} on={this.$listeners}>
      { this.$slots.default.map((item, i) => {
        if (!item.tag) {
          return ''
        }
        if (item.componentOptions.propsData?.type === 'tree') {
          return this.createTree(h, item.componentOptions.propsData, i)
        }
        return item
      }) }
    </el-table>)
  }
}
</script>

<style lang="scss" scoped>
::v-deep.el-table {
  .row-hidden {
    display: none;
  }
  .no-overflow {
    padding: 0;
    .cell {
      padding: 0;
      height: 50px;
      display: flex;
      overflow: visible;
    }
  }
}
</style>
