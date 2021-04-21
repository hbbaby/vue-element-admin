<template>
    <el-dialog
            title="选择课程"
            :visible.sync="dialogVisible"
            width="60%"
            top="5vh"
            :before-close="handleClose">
                <el-container style="height: 71vh; margin-top: -30px; margin-bottom: -30px">
                    <el-header>
                        <el-form :inline="true" style="margin-left:620px">
                            <el-form-item>
                                <el-input v-model="form.title" placeholder="标题"></el-input>
                            </el-form-item>
                            <el-form-item>
                                <el-button type="primary" size="small">搜索</el-button>
                            </el-form-item>
                        </el-form>
                    </el-header>

                    <el-container>
                        <el-aside width="200px">
                            <el-menu :default-active="key" background-color="#eef1f6">
                                <el-menu-item v-for="(item,index) in menus" :index="item.index" :key="index">
                                    <span slot="title">{{item.name}}</span>
                                </el-menu-item>
                            </el-menu>
                        </el-aside>

                        <el-container>
                            <el-main>
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
</template>

<script>
// import { fetchList } from '../../api/media'
// import { fetchList } from '../../api/vedio'
// import { fetchList } from '../../api/audio'
let M = {}      // 用来放三个相同的接口
// 封装导入接口的函数
function importAll(r){
    // keys()从数组中创建一个数组迭代对象 如果对象是数组返回true
    r.keys().forEach((key)=> (M[key] = r(key)) )
}
// require.context用来引入多个组件 
// 第一个参数 目录  第二个参数 是否检索子文件夹 第三个参数 文件名
importAll(require.context("@/api/",true,/\.js$/))

export default {
    data(){
        return{
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
        }
    },
    methods : {
        // open(callback,limit = -1){
        //     this.dialogVisible = true
        //     // this.getData()
        //     this.limit = limit
        //     this.callback = callback
        // },
        handleClose(done) {
            this.$confirm('确认关闭？')
            .then(_ => {
                done();
            }).catch(_ => {});
        },
        // 获取数据
        // getData(){
        //     let fetchList = M[`./${this.key}.js`].fetchList
        //     fetchList(this.form).then((res)=>{
        //         console.log(res);
        //     })
        // },
        // 主题内容改变
        handleSelectionChange(){

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
        }
    }
}
</script>

<style scoped>

</style>