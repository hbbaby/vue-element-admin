<template>
    <div>
        <div class="select-title-btn">
            <el-button class="add-btn" type="primary" @click="openAdd"><svg-icon icon-class="edit" />新增专栏</el-button>

            <el-form :inline="true" class="right-btn">
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
        <el-table :data="tableData" border class="el-table" @sort-change="sortChange">
            <el-table-column algin="center" label="ID" sortable="custom" :class-name="getSortClass('id')" width="80">
                <template slot-scope="scope">
                    {{ scope.row.id }}
                </template>
            </el-table-column>

            <el-table-column algin="center" label="专栏内容">
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
            <el-table-column algin="center" label="更新状态" width="100">
                <template slot-scope="scope"  >
                    <!-- {{ scope.row.isend }} -->
                    <p style="color:red" v-if="scope.row.isend === 0">连载中</p>
                    <p  v-if="scope.row.isend === 1">已完结</p>
                </template>
            </el-table-column>
            <el-table-column algin="center" label="订阅量" width="100">
                <template slot-scope="scope">
                    {{ scope.row.sub_count }}
                </template>
            </el-table-column>
            <el-table-column algin="center" label="状态" width="120">
                <template slot-scope="scope">
                    <el-tag :type="scope.row.status === 1 ? 'success' : 'danger'" size="small">
                        {{ scope.row.status | statusFormat }}
                    </el-tag>
                </template>
            </el-table-column>
            <el-table-column algin="center" label="创建时间" width="160">
                <template slot-scope="scope">
                    {{ scope.row.created_time }}
                </template>
            </el-table-column>
            <el-table-column algin="center" label="操作" width="280">
                
                <template slot-scope="{row,$index}">
                    <el-button type="warning" size="small" @click="goDetail(row)">目录</el-button>
                    <el-button type="primary" size="small" @click="handleEdit(row)">编辑</el-button>
                    <!-- <el-buttom :type="row.status ? '':'success'" @click="handleChangeStatus(row,0)">
                        {{row.status? '下架':'上架'}}
                    </el-buttom> -->
                    <el-button v-if="row.status === 0" type="success" size="small" @click="handleChangeStatus(row,1)">上架</el-button>
                    <el-button v-if="row.status === 1" size="small" @click="handleChangeStatus(row,0)">下架</el-button>
                    <el-button type="danger" size="small" @click="deleteItem(row,$index)">删除</el-button>
                </template> 
               
               
            </el-table-column>
        </el-table>

        <el-pagination
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
        </el-pagination>

        <el-dialog :title="textMap[dialogStatus]" fullscreen :visible.sync="dialogFormVisible">
            <el-form ref="dataForm" :model="rulesForm" :rules="rules" style="width: 600px; margin-left: 50px" >
                <el-form-item>新增</el-form-item>
                <el-form-item label="标题" prop="title">
                    <el-input v-model="rulesForm.title" style="width: 500px; margin-left: 50px" />
                </el-form-item>
                <el-form-item label="封面">
                    <el-upload
                        style="margin-left: 100px"
                        action="https://jsonplaceholder.typicode.com/posts/"
                        list-type="picture-card"
                        :on-preview="handlePictureCardPreview"
                        :on-remove="handleUploadRemove"
                        :on-success="handleUploadSuccess"
                    >
                        <i class="el-icon-plus" />
                    </el-upload>
                <el-dialog :visible.sync="dialogVisible">
                    <img width="100%" :src="dialogImageUrl" alt="">
                </el-dialog>
                </el-form-item>
                    <el-form-item label="试看内容" prop="try">
                    <Tinymce v-model="rulesForm.try" style="width: 600px; height: 300px; margin-left: 100px" />
                </el-form-item>
                <el-form-item label="课程内容" prop="content" style="margin-top: 220px">
                    <Tinymce v-model="rulesForm.content" style="width: 600px; height: 300px; margin-left: 100px" />
                </el-form-item>
                <el-form-item label="课程价格" style="margin: 220px 0 0 30px">
                    <el-input-number v-model="rulesForm.price" :min="1" :max="10"  label="描述文字" />
                </el-form-item>
                <el-form-item label="状态" style="margin: 30px 0 0 30px" >
                    <el-radio-group v-model="rulesForm.status">
                        <el-radio :label="0">下架</el-radio>
                        <el-radio :label="1">上架</el-radio>
                    </el-radio-group>
                    
                </el-form-item>
                <el-form-item style="margin:20px 0 0 150px">
                    <el-button @click="quitAdd">取消</el-button>
                    <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">提交</el-button>
                </el-form-item>
            </el-form>
        </el-dialog>
    </div>
</template>

<script>
import { fetchList,createColumn,deleteColumn,updateColumn,fetchDetail,fetchDetailCourse } from '../../api/column'
import Tinymce from '../../components/Tinymce'

const statusOptions = {
    0:"已下架",
    1:'已上架'
}

export default {
    components: {
        Tinymce
    },
    filters: {
        statusFormat(status) {
            return statusOptions[status]
        }
    },
    data() {
        return {
            tableData: [],
            total:0,
            list:null,
            dialogFormVisible: false,       // 增加的对话框是否显示
            dialogVisible:false,
            dialogImageUrl:"",
            statusOptions,
            page:1,
            sort:'+id',
            num: 1, // 添加时的基础价格
            rulesForm: {    // 得到的数据列表
                id: '',
                title: '',
                status: 1,
                price: 0,
                try: '', // 试看内容
                content: '',
                cover: ''
            },
            listQuery: {
                page: 1,
                limit: 5,
                status: undefined,
                title: undefined,
                sort: '+id'
            },
            dialogStatus:"",
            textMap: {
                update: '修改',
                create: '新增'
            },
            rules: {
                title: [
                    {
                        required: true,
                        message: '标题不能为空',
                        trigger: 'blur'
                    }
                ],
                try: [
                    {
                        required: true,
                        message: '试看内容不能为空',
                        trigger: 'blur'
                    }
                ],
                content: [
                    {
                        required: true,
                        message: '课程内容不能为空',
                        trigger: 'blur'
                    }
                ]
            },
        }
        
    },
    created() {
        this.getList()    
    },
    methods: {
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
        // 图片
        handlePictureCardPreview() {},
        handleUploadRemove() {},
        handleUploadSuccess() {},
        
        // 取消添加
        quitAdd(){
            this.dialogFormVisible = false
        },
        // 清空表单
        resetForm(){
            this.rulesForm = {
                id: '',
                title: '',
                status: 1,
                price: 0,
                try: '', // 试看内容
                content: '',
                cover: ''
            }
        },
        // 点击添加
        openAdd() {
            this.resetForm()
            this.dialogStatus = 'create'
            this.dialogFormVisible = true
            this.$nextTick(() => {
                this.$refs['dataForm'].clearValidate()
            })
        },
        // 提交添加
        createData() {
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.rulesForm.id = parseInt(Math.random() * 100) + 1024 // mock a id
                    this.rulesForm.author = 'vue-element-admin'
                    createColumn(this.rulesForm).then(() => {
                        this.tableData.unshift(this.rulesForm)
                        this.dialogFormVisible = false
                        this.$notify({
                            title: 'Success',
                            message: 'Created Successfully',
                            type: 'success',
                            duration: 2000
                        })
                    })
                }
            })
        },
        // 点击编辑
        handleEdit(row) {
                this.rulesForm = Object.assign({}, row) // copy obj
                // this.rulesForm.timestamp = new Date(this.rulesForm.timestamp)
                this.dialogStatus = 'update'
                this.dialogFormVisible = true
                this.$nextTick(() => {
                    this.$refs['dataForm'].clearValidate()
                })
            },
        // 提交编辑
        updateData() {
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    const tempData = Object.assign({}, this.rulesForm)
                    // tempData.timestamp = +new Date(tempData.timestamp) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
                    updateColumn(tempData).then(() => {
                        const index = this.tableData.findIndex(v => v.id === this.rulesForm.id)
                        this.tableData.splice(index, 1, this.rulesForm)
                        this.dialogFormVisible = false
                        this.$notify({
                            title: 'Success',
                            message: 'Update Successfully',
                            type: 'success',
                            duration: 2000
                        })
                    })
                }
            })
        },
        // 删除一行
        deleteItem(row,index){
            deleteColumn(row).then((res)=>{
                // console.log(res);
                if(res.code === 20000){
                    this.$notify({
                        title:'提示',
                        message:"删除成功",
                        type:'success',
                        duration:2000
                    })
                    this.tableData.splice(index,1)
                }
            })
        },
        // 改变状态
        handleChangeStatus(row,status){
            this.$message({
                message:"修改成功",
                type:'success'
            })
            row.status = status
        },
        
        // 表单发生改变调此函数 然后调用id分类
        sortChange(data){
            const { prop,order } = data
            console.log(data);
            if(prop === 'id'){
                this.sortById(order)
            }
        },
        // 列明
        getSortClass(key){
            const sort = this.listQuery.sort
            return sort === `+${key}` ? `ascending` : `descending`
        },
        // 通过id分类 调用查询函数
        sortById(order){
            if(order === 'ascending'){
                this.listQuery.sort = "+id"
            }
            if(order === 'descending'){
                this.listQuery.sort = "-id"
            }
            this.handleSearch()
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
        },
        // 去详情页面
        goDetail(row){
            console.log(row.id);     // id
            let id = row.id
            this.$router.push({
                path:"/course/column-detail/"+id,
            })
        }
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
