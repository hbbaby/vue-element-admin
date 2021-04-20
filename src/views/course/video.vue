<template>
  <div>
    <div class="select-title-btn">
      <el-button class="add-btn" type="primary"><svg-icon icon-class="edit" />新增视频</el-button>

      <el-form :inline="true" class="right-btn">
        <el-form-item>
          <el-select v-model="value" placeholder="商品状态">
            <el-option label="已上架" value="1" />
            <el-option label="已下架" value="2" />
          </el-select>
        </el-form-item>

        <el-form-item>
          <el-input v-model="title" placeholder="标题" />
        </el-form-item>

        <el-form-item>
          <el-button type="primary">查询</el-button>
        </el-form-item>
      </el-form>
    </div>

    <el-table :data="tableData" border class="el-table">
      <el-table-column label="ID" width="80">
        <template slot-scope="scope">
          {{ scope.row.id }}
        </template>
      </el-table-column>
      <el-table-column label="视频内容">
        <template slot-scope="scope">
          {{ scope.row.content }}
        </template>
      </el-table-column>
      <el-table-column label="订阅量" width="120">
        <template slot-scope="scope">
          {{ scope.row.book }}
        </template>
      </el-table-column>
      <el-table-column label="状态" width="120">
        <template slot-scope="scope">
          <!-- {{scope.row.status}} -->
          <el-button type="success" size="small" plain>{{ scope.row.status }}</el-button>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" width="160">
        <template slot-scope="scope">
          {{ scope.row.time }}
        </template>
      </el-table-column>
      <el-table-column label="操作" width="280">
        <el-button type="primary" size="small">编辑</el-button>
        <el-button type="success" size="small">上架</el-button>
        <el-button type="danger" size="small">删除</el-button>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import { fetchList } from '../../api/course'
export default {
  data() {
    return {
      title: '', // 标题
      value: '', // 内容
      tableData: [
        {
          id: '1',
          content: '',
          book: '623',
          status: '已上架',
          time: '2021/4/20'
        }, {
          id: '2',
          content: '',
          book: '623',
          status: '已上架',
          time: '2021/4/20'
        }, {
          id: '3',
          content: '',
          book: '623',
          status: '已上架',
          time: '2021/4/20'
        }, {
          id: '4',
          content: '',
          book: '623',
          status: '已上架',
          time: '2021/4/20'
        }
      ]
    }
  },
  mounted() {
    // this.getfetchList()
  },
  created() {

  },
  methods: {
    async getfetchList() {
      const res = await fetchList()
      console.log(res)
    }
  }
}
</script>

<style scoped>
.add-btn{
    margin-left: 40px;
}
.select-title-btn{
    display: flex;
    justify-content: space-between;
    width: 98%;
    height: 40px;
    /* border: 1px solid red; */
    margin: 20px 20px 0 0;
}

.el-table{
    width: 96%;
    text-align: center;
    margin: 20px auto;
}
</style>>
