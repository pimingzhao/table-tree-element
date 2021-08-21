<!--
 * @Author: pimzh
 * @Date: 2021-08-21 13:41:10
 * @LastEditTime: 2021-08-21 14:07:51
 * @LastEditors: pimzh
 * @Description: 
-->
<template>
  <div>
    <el-button @click="handleTree">tree</el-button>
    <el-button @click="handleTable">table</el-button>
    <table-tree :data="data" style="width: 100%" border @row-click="handleRow">
      <el-table-column type="index" width="50" fixed="left" />
      <el-table-column v-if="isTree" type="tree" width="120" fixed="left" />
      <el-table-column prop="name" label="名称" />
      <el-table-column prop="date" label="日期" width="580" />
      <el-table-column prop="date" label="日期" width="180" />
      <el-table-column prop="date" label="日期" width="180" />
      <el-table-column prop="date" label="日期" width="180" />
      <el-table-column prop="address" label="地址" width="580" fixed="right" />
    </table-tree>
  </div>
</template>

<script>
import TableTree from './components/table-tree'
export default {
  components: {
    TableTree
  },
  data() {
    return {
      data: [],
      isTree: false
    }
  },
  created() {
    this.handleTree()
  },
  methods: {
    loopTree(data, cb, p) {
      data.forEach((item, i) => {
        cb(item, i, p)
        if (item.children?.length) {
          this.loopTree(item.children, cb, item)
        }
      })
    },
    handleTable() {
      this.isTree = false
      this.data = this.getData()
    },
    handleTree() {
      this.isTree = true
      const data = this.getCommonData()
      this.loopTree(data, (item) => {
        // 默认展开所有节点
        item.expand = true
      })
      this.data = data
    },
    getCommonData() {
      return [
        {
          date: '2016-05-02',
          name: '1',
          address: '上海市普陀区金沙江路 1518 弄'
        },
        {
          date: '2016-05-04',
          name: '2',
          address: '上海市普陀区金沙江路 1517 弄',
          children: [
            {
              date: '2016-05-04',
              name: '2-1',
              address: '上海市普陀区金沙江路 1517 弄',
              children: [
                {
                  date: '2016-05-04',
                  name: '2-1-1',
                  address: '上海市普陀区金沙江路 1517 弄'
                },
                {
                  date: '2016-05-04',
                  name: '2-1-2',
                  address: '上海市普陀区金沙江路 1517 弄'
                },
                {
                  date: '2016-05-04',
                  name: '2-1-3',
                  address: '上海市普陀区金沙江路 1517 弄'
                }
              ]
            },
            {
              date: '2016-05-04',
              name: '2-2',
              address: '上海市普陀区金沙江路 1517 弄',
              children: [
                {
                  date: '2016-05-04',
                  name: '2-2-1',
                  address: '上海市普陀区金沙江路 1517 弄'
                },
                {
                  date: '2016-05-04',
                  name: '2-2-2',
                  address: '上海市普陀区金沙江路 1517 弄'
                },
                {
                  date: '2016-05-04',
                  name: '2-2-3',
                  address: '上海市普陀区金沙江路 1517 弄'
                }
              ]
            },
            {
              date: '2016-05-04',
              name: '2-3',
              address: '上海市普陀区金沙江路 1517 弄'
            }
          ]
        },
        {
          date: '2016-05-01',
          name: '3',
          address: '上海市普陀区金沙江路 1519 弄'
        },
        {
          date: '2016-05-03',
          name: '4',
          address: '上海市普陀区金沙江路 1516 弄'
        }
      ]
    },
    getData() {
      return [
        {
          date: '2016-05-02',
          name: '1',
          address: '上海市普陀区金沙江路 1518 弄'
        },
        {
          date: '2016-05-04',
          name: '2',
          address: '上海市普陀区金沙江路 1517 弄',
          children: []
        },
        {
          date: '2016-05-04',
          name: '2',
          address: '上海市普陀区金沙江路 1517 弄',
          children: []
        }
      ]
    },
    handleRow(row) {
      console.log(row)
    }
  }
}
</script>
