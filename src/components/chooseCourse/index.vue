<template>
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
                            <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getData" />
                        </el-footer>
                    </el-container>
                    
                </el-container>
            </el-container>

            <div style="margin-top:30px">
                <span slot="footer">
                    <el-button @click="dialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="confirm">确 定</el-button>
                </span>
            </div>
            
    </el-dialog>
</template>

<script>

let M = {}      // 用来放三个相同的接口
// 封装导入接口的函数
function importAll(r){
    // keys()从数组中创建一个数组迭代对象 如果对象是数组返回true
    r.keys().forEach((key)=> (M[key] = r(key)) )
}
// require.context用来引入多个组件 
// 第一个参数 目录  第二个参数 是否检索子文件夹 第三个参数 文件名
importAll(require.context("@/api/",true,/\.js$/))
import Pagination from '@/components/Pagination'
export default {
    components:{
        Pagination
    },
    data() {
        return {
            tableData: [],      // 表格内容
            
            listQuery: {
                title:"",
                page:1,
                limit:10,
            },
            form: {
                title: "",
                page: 1,
                limit: 10,
            },
            key:"media",
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
        // 获取弹窗中的数据
        this.getData()
    },
    methods:{
        open(callback,limit = -1){
            this.dialogVisible = true
            this.limit = limit
            this.callback = callback
            this.dialogVisible = true
            this.getData()
        },
        // 里面的查询
        handleInnerSearch(){
            this.form.page = 1
            this.getData()
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
         // 弹窗侧边选择
        onAsideSelect(e){
            // console.log(e);     // media,vedio,audio
            this.key = e
            this.form.page = 1
            this.getData()
        },
        handleClose(done) {
            this.$confirm('确认关闭？')
            .then(_ => {
                done();
            }).catch(_ => {});
        },
        // 主题内容改变
        handleSelectionChange(val){
            this.multipleSelection = val
        },
        close(){
            this.dialogVisible = false
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
            this.$refs.multipleTable.clearSelection()
        },
    }
}
</script>

<style scoped>

</style>