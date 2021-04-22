<template>
    <div>
        <div class="select-title-btn">
            <el-dropdown>
                <el-button style="width:120px;margin-left:40px" type="danger">黑名单设置<i class="el-icon-arrow-down el-icon--right"></i></el-button>
                <el-dropdown-menu slot="dropdown">
                    <el-dropdown-item @click.native="denyComment(true)">禁止评论</el-dropdown-item>
                    <el-dropdown-item @click.native="denyAsk(true)">禁止访问</el-dropdown-item>
                </el-dropdown-menu>
            </el-dropdown>
            <el-form :inline="true" class="right-btn">
                

                <el-form-item>
                    <el-input v-model="listQuery.title" placeholder="用户名" @keyup.enter.native="handleSearch" />
                </el-form-item>

                <el-form-item>
                    <el-button type="primary" @click="handleSearch">查询</el-button>
                </el-form-item>
            </el-form>
        </div>
        <!-- @sort-change 排序条件发生变化时触发 -->
        <!-- sortable="custom" 是否可以排序 custom表示可以-->
        <!-- class-name 列的类名 -->
        <!-- @sort-change="sortChange" -->
        <el-table :data="tableData" ref="multipleTable" border class="el-table" @selection-change="sortChange">
            <el-table-column type="selection" width="55"> </el-table-column>

            <el-table-column label="用户">
                <div slot-scope="scope" style="display: flex;flex-wrap:nowrap">
                    <div>
                        <el-image class="user-img" :src="scope.row.user.avatar" />
                    </div>
                    <div style="line-height:70px">
                        {{ scope.row.user.username }}
                    </div>
                </div>
            </el-table-column>
            
            <el-table-column algin="center" label="消费总额" width="80">
                <template slot-scope="scope">
                    {{ scope.row.total_consume }}
                </template>
            </el-table-column>
            
            <el-table-column  algin="center" label="创建时间" width="160">
                <template slot-scope="scope">
                    {{ scope.row.created_time }}
                </template>
            </el-table-column>
            <el-table-column algin="center" label="操作" width="300">
                
                <template slot-scope="{row}">
                    <el-button type="primary" size="small" @click="openinfo(row.user.id)" >详情</el-button>
                    <el-button type="success" size="small" >联系用户</el-button>
                    <el-dropdown>
                        <el-button style="margin-left:20px" size="small" type="danger">黑名单设置<i class="el-icon-arrow-down el-icon--right"></i></el-button>
                        <el-dropdown-menu slot="dropdown">
                            <el-dropdown-item @click.native="denyComment(row.user.id)">禁止评论</el-dropdown-item>
                            <el-dropdown-item @click.native="denyAsk(row.user.id)">禁止访问</el-dropdown-item>
                        </el-dropdown-menu>
                    </el-dropdown>
                </template> 
               
               
            </el-table-column>
        </el-table>

        <!-- <el-pagination
            style="margin-top:30px"
            background
            layout="total, sizes, prev, pager, next, jumper"
            :total="total"
            :page.sync="listQuery.page"
            :page-sizes="[5,10,15]"
            :limit.sync="listQuery.limit"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
        >
        </el-pagination> -->
        <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />
        <info ref="info"></info>
    </div>
</template>

<script>
import { changeAccessStatus, changeCommentStatus, fetchList } from '../../api/user'
import Pagination from '@/components/Pagination' 
import info from "./info"

export default {
    components:{
        info,Pagination
    },
    data() {
        return {

            total:0,
            tableData:null,

            listQuery: {
                page: 1,
                limit: 5,
                title: undefined,
            },
            multipleSelection:[],   // 存放当前这一行的数据
        }
        
    },
    created() {
        this.getList()    
    },
    methods: {
        // 打开详情
        openinfo(id){
            this.$refs.info.open(id)
        },
        // 获取列表
        async getList(){
            await fetchList(this.listQuery).then((res) => {
                console.log(res)
                if (res.code === 20000) {
                    this.tableData = res.data.items,
                    this.total = res.data.total
                }
            })
            .catch((err) => {
                console.log('错误')
            })
        },
        // 禁止评论
        denyComment(ids){
            if(typeof ids == 'boolean' && !this.multipleSelection.length){
                return this.$message({
                    message:"请先选中需要操作的用户",
                    type:"error"
                })
            }
            let id = typeof ids == 'boolean' ? this.multipleSelection.map(item=>item.id) : ids
            let allText = typeof ids == 'boolean' ? '选中' : '当前'
            this.$confirm("是否要禁止" + allText + "用户评论", '提示',{
                confirmButtonText:'确定',
                cancelButtonText:'取消',
                type:'warning',
            }).then(action => {
                changeCommentStatus({
                    id,status:0
                }).then(res => {
                    this.$message({
                        message:"操作成功",
                        type:"success"
                    })
                    this.$refs.multipleTable.clearSelection()
                })
            })
        },
        // 禁止访问
        denyAsk(ids){
            if(typeof ids == 'boolean' && !this.multipleSelection.length){
                return this.$message({
                    message:"请先选中需要操作的用户",
                    type:"error"
                })
            }
            let id = typeof ids == 'boolean' ? this.multipleSelection.map(item=>item.id) : ids
            let allText = typeof ids == 'boolean' ? '选中' : '当前'
            this.$confirm("是否要禁止" + allText + "用户访问", '提示',{
                confirmButtonText:'确定',
                cancelButtonText:'取消',
                type:'warning',
            }).then(action => {
                changeAccessStatus({
                    id,status:0
                }).then(res => {
                    this.$message({
                        message:"操作成功",
                        type:"success"
                    })
                    this.$refs.multipleTable.clearSelection()
                })
            })
        },
        // 表单发生改变
        sortChange(data){
            // console.log(data);      // 当前这一行的数据
            this.multipleSelection = data
        },

        // 查询
        handleSearch(){
            this.listQuery.page = 1
            this.getList()
        },
        // 分页
        // 改变每页显示的条数
        handleSizeChange(val) {
            this.listQuery.page = 1
            this.listQuery.limit = val
            // 重新加载数据
            this.getList()
        },
        handleCurrentChange(val) {
            this.listQuery.page = val
            // 重新加载数据
            this.getList()
        }
    }
}
</script>

<style scoped>
/*  style=" "  */
.user-img{
    width: 70px;
    height: 70px;
    margin-right:10px;
    border-radius: 50%;
}
.select-title-btn {
  display: flex;
  justify-content: space-between;
  width: 98%;
  height: 40px;
  margin: 20px 20px 0 0;
}

.el-table {
  width: 96%;
  text-align: center;
  margin: 20px auto;
}

</style>>
