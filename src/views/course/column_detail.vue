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

        <el-dialog
            title="选择课程"
            :visible.sync="dialogVisible"
            width="70%"
            top="5vh"
            ref="chooseCourse"
            :before-close="handleClose">
                <el-container style="height: 71vh; margin-top: -30px; margin-bottom: -30px">
                    <el-header>
                        <el-form :inline="true" style="margin-left:720px">
                            <el-form-item>
                                <el-input v-model="form.title" placeholder="标题"></el-input>
                            </el-form-item>
                            <el-form-item>
                                <el-button type="primary" size="small" @click="handleInnerSearch">搜索</el-button>
                            </el-form-item>
                        </el-form>
                    </el-header>

                    <el-container>
                        <el-aside width="200px">
                            <el-menu @select="onAsideSelect" :default-active="key" background-color="#eef1f6">
                                <el-menu-item v-for="(item,index) in menus" :index="item.index" :key="index">
                                    <span slot="title">{{item.name}}</span>
                                </el-menu-item>
                            </el-menu>
                        </el-aside>

                        <el-container>
                            <el-main style="margin-top:-30px;height:60vh">
                                <el-table 
                                    :data="list" :key="tableKey" border fit 
                                    highlight-current-row 
                                    style="width:100%"
                                    ref="multipleTable"
                                   
                                    @selection-change="handleSelectionChange"
                                >
                                    <!-- 全选 -->
                                    <el-table-column type="selection" width="55"> </el-table-column>
                                    <el-table-column label="内容" min-width="180px">
                                        <template slot-scope="{ row }">
                                            <div style="display: flex">
                                            <img :src="row.cover" style="width: 100px; height: 50px; margin-right: 10px" />
                                            <div style=" display: flex; flex-direction: column;justify-content: space-between;" >
                                                <span>{{ row.title }}</span>
                                                <span style="color: red">￥{{ row.price }}</span>
                                            </div>
                                            </div>
                                        </template>
                                    </el-table-column>
                                </el-table>
                            </el-main>
                            <el-footer>
                                <el-pagination
                                    style="margin-top:30px"
                                    background
                                    layout="total, sizes, prev, pager, next, jumper"
                                    :total="total"
                                    :page.sync="listQuery.page"
                                    :page-sizes="[5,10,15]"
                                    @pagination="getData"
                                    :limit.sync="listQuery.limit"
                                    @size-change="handleSizeChange"
                                    @current-change="handleCurrentChange"
                                >
                                </el-pagination>
                            </el-footer>
                        </el-container>

                    </el-container>
                </el-container>

                <span slot="footer" class="dialog-footer">
                    <el-button @click="dialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
                </span>
        </el-dialog>

    </div>
</template>

<script>
import { fetchDetail,fetchDetailCourse } from "../../api/column"

let M = {}      // 用来放三个相同的接口
// 封装导入接口的函数
function importAll(r){
    // keys()从数组中创建一个数组迭代对象 如果对象是数组返回true
    r.keys().forEach((key)=> (M[key] = r(key)) )
}
// require.context用来引入多个组件 
// 第一个参数 目录  第二个参数 是否检索子文件夹 第三个参数 文件名
importAll(require.context("@/api/",true,/\.js$/))

const statusOptions = {
    0:"已下架",
    1:'已上架'
}

export default {
    
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
            input:"",
            total:"",
            tableKey:"",
            menus: [
                {
                name: "图文",
                index: "media",
                },
                {
                name: "音频",
                index: "audio",
                },
                {
                name: "视频",
                index: "video",
                },
            ],
            list: [],
            total:0,
            dialogVisible:false,
            callback: null,
            limit: -1,
            multipleSelection:[],   // 选中的条数
        }
        
    },
    created() {
        // 得到上面的
        this.getList()    
        // 得到下面的表格
        this.getNextList()
        // 获取弹窗中的数据
        this.getData()
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
        // 图片
        handlePictureCardPreview() {},
        handleUploadRemove() {},
        handleUploadSuccess() {},
       
        // 点击添加
        openAdd() {
            this.$refs.chooseCourse.open((val)=>{
                console.log('...');
                this.tableData = [...this.tableData,...val]
            },50)
            this.dialogVisible = true
        },
        open(callback,limit = -1){
            this.dialogVisible = true
            this.limit = limit
            this.callback = callback
            this.dialogVisible = true
            this.getData()
        },
        // 新增专栏时 点击确定
        confirm(){
            if(this.multipleSelection.length === 0){
                return this.$message.error("至少选中一个")
            }
            if(this.limit != -1 && this.multipleSelection.length > this.limit ){
                return this.$message.error("最多可以选中"+this.limit+"个")
            }
            this.callback(this.multipleSelection)
            this.close()
            // 清空选中
            this.$refs.multipleSelection.clearSelection()
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
        // 里面的查询
        handleInnerSearch(){
            this.form.page = 1
            this.getData()
        },
        
        // 处理上架下架
        handleUpDown(){
            this.detail.status = this.detail.status ? 0 : 1
        },
        // 处理完结连载
        changeIsend(){
            this.detail.isend = this.detail.isend ? 0 : 1
        },
        handleClose(done) {
            this.$confirm('确认关闭？')
            .then(_ => {
                done();
            }).catch(_ => {});
        },
        // 获取数据
        getData(){
            let fetchList = M[`./${this.key}.js`].fetchList
            fetchList(this.form).then((res)=>{
                // console.log(res);
                this.list = res.data.items
                this.total = res.data.total
            })
        },
        // 主题内容改变
        handleSelectionChange(val){
            this.multipleSelection = val
        },
        // 分页
        // 改变每页显示的条数
        handleSizeChange(val) {
            this.form.page = 1
            this.form.limit = val
            // 重新加载数据
            this.getData()
        },
        handleCurrentChange(val) {
            this.form.page = val
            // 重新加载数据
            this.getData()
        },
        // 弹窗侧边选择
        onAsideSelect(e){
            // console.log(e);     // media,vedio,audio
            this.key = e
            this.form.page = 1
            this.getData()
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
