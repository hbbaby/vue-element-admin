<template>
    <div>
        <div class="select-title-btn">

            <el-button @click="handleDownload" style="width:120px;margin-left:40px" type="primary">导出选中</el-button>
                
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

            <el-table-column align="center" label="订单号" width="80">
                <template slot-scope="scope">
                    {{ scope.row.no }}
                </template>
            </el-table-column>

            <el-table-column label="商品名称">
                <div slot-scope="scope" style="display: flex;flex-wrap:nowrap">
                    <div style="line-height:70px" v-for="(item,index) in scope.row.goods" :key="index">
                        {{ item.title }}
                    </div>
                </div>
            </el-table-column>
            
            <el-table-column align="center" label="订单类型" width="80">
                <template slot-scope="scope">
                    {{ scope.row.type | typeFilters}}
                </template>
            </el-table-column>

            <el-table-column align="center" label="订单状态" width="100">
                <template slot-scope="scope">
                    <el-tag :type="scope.row.status == 'success' ? 'success' : 'danger'" size="small">
                        {{ scope.row.status | statusFilters }}
                    </el-tag>
                </template>
            </el-table-column>


            <el-table-column align="center" label="原价/实付(元)" width="120">
                <template slot-scope="scope">
                    <span>{{ scope.row.total_price }}/</span>
                    <span style="color:red">{{scope.row.price}}</span>
                </template>
            </el-table-column>
            
            <el-table-column align="center" label="支付方式" width="80">
                <template slot-scope="scope">
                    {{ scope.row.pay_method | methodFilters }}
                </template>
            </el-table-column>
            
            <el-table-column  align="center" label="创建/支付时间" width="160">
                <template slot-scope="scope">
                    <p>{{ scope.row.created_time }}</p>
                    <p>{{ scope.row.updated_time }}</p>
                </template>
            </el-table-column>

            <el-table-column align="center" label="操作" width="100">
                <template slot-scope="{row,$index}">
                    <el-button type="danger" size="small" @click="deleteRow(row,$index)" >删除</el-button>
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
    </div>
</template>

<script>
import { fetchOrder,deleteOrder } from '../../api/pay'
import Pagination from '@/components/Pagination' 
import { parseTime } from '@/utils'
// import Export2Excel from "@/vendor/Export2Excel"

let typeOptions = {
    group:"拼团",
    default:"普通"
}
let statusOptions = {
    pendding:"待支付",
    success:"成功",
    fail:"失败"
}
let methodOptions = {
    wechat:"微信",
    apply:"支付宝"
}

export default {
    components:{
        Pagination
    },
    filters:{
        // 拼团还是普通
        typeFilters(val){
            return typeOptions[val]
        },
        // 成功还是失败
        statusFilters(val){
            return statusOptions[val]
        },
        // 微信还是支付宝
        methodFilters(val){
            return methodOptions[val]
        }
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
            downloadLoading:false
        }
        
    },
    created() {
        this.getList()    
    },
    methods: {
       
        // 获取列表
        async getList(){
            await fetchOrder(this.listQuery).then((res) => {
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
        // 删除当前行
        deleteRow(row,index){
            deleteOrder(row).then(res=>{
                // console.log(res);
                if(res.code === 20000){
                    this.$message({
                        showClose: true,
                        message:"删除成功",
                        type:"success"
                    })
                }
                this.tableData.splice(index,1)
            })
        },
        // 导出选中
        handleDownload(){
            this.downloadLoading = true
            // Export2Excel.then(excel=>{
            //     console.log(excel);
            // })
            import("@/vendor/Export2Excel").then(excel=>{
                // console.log(excel);
                const tHeader = [
                        '订单号', 
                        '商品名称', 
                        '订单类型', 
                        '订单状态', 
                        '原价（元）',
                        '实付（元）',
                        '支付方式',
                        '创建时间',
                        '支付时间',
                    ]
                    const filterVal = [
                        'no', 
                        'goods', 
                        'type', 
                        'status', 
                        'total_price',
                        'price',
                        'pay_method',
                        'created_time',
                        'pay_time',
                    ]
                    const list = this.multipleSelection.map(item=>{
                        console.log(item);      // 是一个对象 是当前行的信息
                        let obj = { ...item }   // 把行中的信息结构出来
                        obj.goods = item.goods.map(v=>v.title).join(",")
                        obj.type = typeOptions[item.type]
                        obj.pay_method = methodOptions[item.pay_method]
                        return obj
                    })
                    const data = this.formatJson(filterVal,this.tableData)
                    excel.export_json_to_excel({
                        header:tHeader,data:data
                    })
                    this.downloadLoading = false
            })
        },
        formatJson(filterVal,jsonData){
            return jsonData.map(v => filterVal.map(j => {
                if(j === 'timestamp'){
                    return parseTime(v[j])
                }else{
                    return v[j]
                }
            }))
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
