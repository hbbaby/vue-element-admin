<template>
    <div>
        <div>
            <el-card shadow="hover" style="width:96%;margin:20px auto">
                <div style="display:flex">
                    <div>
                        <img :src="detail.cover" alt="">
                    </div>
                    <div style="margin-left:20px">
                        <div style="margin-top:-20px">
                            <h4>{{detail.title}}</h4>
                            <p style="font-size:12px;color:grey">{{detail.content}}</p>
                            <p style="color:red;margin-top:-5px">￥{{detail.price}}</p>
                        </div>
                        <div style="margin-top:-5px">
                            <el-button size="small" type="warning" @click="handleUpDown">{{detail.status ? '下架' : '上架' }}</el-button>
                            <el-button type="default" size="small" @click="changeIsend">设为{{detail.isend ? '连载中':'已完结'}}</el-button>
                        </div>
                    </div>
                </div>
            </el-card>
        </div>
        <div class="select-title-btn">
            <el-button class="add-btn" type="primary" @click="openAdd">
                <svg-icon icon-class="edit" />新增目录</el-button>
            <el-form :inline="true">
                <el-form-item>
                    <!-- clearable 是否显示清除按钮 -->
                    <el-select v-model="listQuery.status" placeholder="商品状态" clearable>
                        <el-option v-for="(item,index) in statusOptions" :key="index" :label="item" :value="index" />
                    </el-select>
                </el-form-item>

                <el-form-item>
                    <el-input v-model="listQuery.title" placeholder="标题" @keyup.enter.native="handleSearch" />
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
        <el-table ref="dragTable" :data="tableData" border class="el-table" @sort-change="sortChange">
            <el-table-column align="center" label="ID" sortable="custom" :class-name="getSortClass('id')" width="80">
                <template slot-scope="scope">
                    {{ scope.row.id }}
                </template>
            </el-table-column>

            <el-table-column label="单品内容">
                <div slot-scope="scope" style="display: flex;flex-wrap:nowrap">
                    <div>
                        <el-image style="width: 100px; height: 50px;margin-right:10px" :src="scope.row.cover" />
                    </div>
                    <div >
                        <div style="displsy:flex;flex-wrap:wrap">
                            <div>
                                {{ scope.row.title }}
                            </div>
                            <div style="color:red;margin-top:10px">
                                ${{ scope.row.price }}
                            </div>
                            
                        </div>
                    </div>
                </div>
            </el-table-column>
            
            <el-table-column align="center" label="订阅量" width="120">
                <template slot-scope="scope">
                    {{ scope.row.sub_count }}
                </template>
            </el-table-column>
            <el-table-column align="center" label="状态" width="120">
                <template slot-scope="scope">
                    <el-tag :type="scope.row.status === 1 ? 'success' : 'danger'" size="small">
                        {{ scope.row.status | statusFormat }}
                    </el-tag>
                </template>
            </el-table-column>
            <el-table-column align="center" label="创建时间" width="160">
                <template slot-scope="scope">
                    {{ scope.row.created_time }}
                </template>
            </el-table-column>
            <el-table-column align="center" label="操作" width="180">
                <template slot-scope="{row,$index}">
                    <el-button type="primary" size="small" @click="handleEdit(row)">编辑</el-button>
                    <el-button type="danger" size="small" @click="deleteItem(row,$index)">删除</el-button>
                </template> 
            </el-table-column>
            <el-table-column align="center" label="Drag" width="100">
                <svg-icon 
                    draggable icon-class="drag" 
                    style="font-size:26px;cursor:pointer;"
                />
            </el-table-column>
            
        </el-table>

        <chooseCourse ref="chooseCourse"></chooseCourse>

    </div>
</template>

<script>
import { fetchDetail,fetchDetailCourse } from "../../api/column"

import chooseCourse from "@/components/chooseCourse/index"

const statusOptions = {
    0:"已下架",
    1:'已上架'
}

export default {
    components:{
        chooseCourse
    },
    filters: {
        statusFormat(status) {
            return statusOptions[status]
        }
    },
    data() {
        return {
            tableData: [],      // 表格内容
            detail: {
                content: "",
                cover: "",
                created_time: "",
                id: 0,
                isend: 0,
                price: 0,
                status: 1,
                sub_count: 0,
                title: "",
                try: "",
                updated_time: ""
            },
            listQuery: {
                status: undefined,
                title:"",
                page:1,
                limit:10,
                sort:"+id"
            },
            statusOptions,
            form: {
                title: "",
                page: 1,
                limit: 10,
            },
            key:"media",
            total:"",
            list: [],
            total:0,
            dialogVisible:false,
            limit: -1,
            multipleSelection:[],   // 选中的条数
        }
    },
    created() {
        // 得到上面的
        this.getList()    
        // 得到下面的表格
        this.getNextList()
    },
    methods: {
        async getNextList(){
            await fetchDetailCourse(this.listQuery).then((res)=>{
                // console.log(res);
                this.tableData = res.data.items
                this.total = res.data.total
            })
            // 
        },
        // 拖拽函数
        // sortSet(){

        // },
        // 获取列表
        async getList(){
            let id = this.$route.params.id
            await fetchDetail({id}).then((res) => {
                // console.log(res)
                if (res.code === 20000) {
                    this.detail = res.data
                }
            }).catch((err) => {
                console.log('错误')
            })
        },
       
        // 点击添加
        openAdd() {
            this.$refs.chooseCourse.open((val)=>{
                console.log('...');
                this.tableData = [...this.tableData,...val]
            },50)
            this.dialogVisible = true
        },
        // 点击编辑
        handleEdit(row) {
            
        },
        
        // 删除一行
        deleteItem(row,index){
            this.$notify({
                title:'提示',
                message:"删除成功",
                type:'success',
                duration:2000
            })
            this.tableData.splice(index,1)
        },
        
        // 表单发生改变
        sortChange(data){
            // console.log(data);   // 没有执行
            const { prop,order } = data
            if(prop === 'id'){
                this.sortById(order)
            }
        },
        // 列明
        getSortClass(key){
            const sort = this.listQuery.sort
            return sort === `+${key}` ? `ascending` : `descending`
        },
        // 通过id分类
        sortById(order){
            if(order === 'ascending'){
                this.listQuery.sort = "+id"
            }
            if(order === 'descending'){
                this.listQuery.sort = "-id"
            }
            this.handleSearch()
            this.handleInnerSearch()
        },
        // 查询
        handleSearch(){
            this.listQuery.page = 1
            this.getNextList()
        },
        // 处理上架下架
        handleUpDown(){
            this.detail.status = this.detail.status ? 0 : 1
        },
        // 处理完结连载
        changeIsend(){
            this.detail.isend = this.detail.isend ? 0 : 1
        },
    }
}
</script>

<style scoped>
.add-btn {
  margin-left: 40px;
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
