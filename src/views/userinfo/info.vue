<template>
    <div>
        <el-dialog title="用户详情" :visible.sync="dialogVisible" width="80%" top="5vh">
            <el-container>
                <el-header style="height:12vh">
                    <!-- 指定分栏间隔 -->
                    <el-row :gutter="20">
                        <el-col :span="6"><p>ID：{{id}}</p></el-col>
                        <el-col :span="6"><p>用户名：{{user.username}}</p></el-col>
                        <el-col :span="6"><p>昵称：{{user.nickname}}</p></el-col>
                        <el-col :span="6"><p>手机号：{{user.phone}}</p></el-col>
                        <el-col :span="6"><p>锁定：{{user.status ? '否' : '是'}}</p></el-col>
                        <el-col :span="6"><p>会员：{{user.user_level}}</p></el-col>
                        <el-col :span="6"><p>会员到期时间：{{user.user_level_expire}}</p></el-col>
                        <el-col :span="6"><p>注册时间：{{user.created_time}}</p></el-col>
                    </el-row>
                </el-header>
                <el-main style="height:55vh;padding-bottom:0;">
                    <el-tabs v-model="activeName" @tab-click="handleClick">
                        <el-tab-pane :name="item.name" :label="item.label" v-for="(item,index) in tabs" :key="index">
                            <el-table :data="list" height="280" border style="width:100%">
                                <el-table-column :label="item1.label" :prop="item1.prop" v-for="(item1,index1) in item.ths" :key="index1">

                                </el-table-column>
                            </el-table>

                        </el-tab-pane>
                        
                    </el-tabs>
                </el-main>
                <el-footer>
                    <Pagination :total="listQuery.total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" ></Pagination>
                </el-footer>
            </el-container>
        </el-dialog>
    </div>
</template>

<script>
import { fetchUserDetail,fetchUserCourse,fetchUserColumn,fetchUserOrder,fetchUserHistory,fetchUserComment } from "../../api/user"
import Pagination from '@/components/Pagination' 

export default {
    name:"info",
    components:{
        Pagination
    },
    data(){
        return{
            dialogVisible:false,
            id:0,
            user:{
                avatar: "",         // 用户头像
                created_time: "",   // 创建时间
                id: 0,      
                nickname: "",       // 
                phone: "",
                status: 0,
                user_level: "",     // 年度会员
                user_level_expire: "",  // 创建时间
                username: "",
            },
            activeName:"course",
            listQuery:{
                page:1,
                limit:10,
                total:100
            },
            list:[],
            tabs:[{
                label:"课程订阅",
                name:"course",
                ths:[{
                    prop:"title",
                    label:"课程标题"
                },{
                    prop:"price",
                    label:"购买价格"
                },{
                    prop:"created_time",
                    label:"购买时间"
                }],
                request:fetchUserCourse
            },{
                label:"专栏订阅",
                name:"column",
                ths:[{
                    prop:"title",
                    label:"专栏标题"
                },{
                    prop:"price",
                    label:"购买价格"
                },{
                    prop:"created_time",
                    label:"购买时间"
                }],
                request:fetchUserColumn
            },{
                label:"订单记录",
                name:"order",
                ths:[{
                    prop:"id",
                    label:"ID"
                },
                // {
                //     prop:"no",
                //     label:"订单号"
                // },
                {
                    prop:"price",
                    label:"购买价格"
                },
                // {
                //     prop:"status",
                //     label:"状态"
                // },
                {
                    prop:"title",
                    label:"商品"
                },{
                    prop:"created_time",
                    label:"购买时间"
                }],
                request:fetchUserOrder
            },{
                label:"观看历史",
                name:"history",
                ths:[{
                    prop:"title",
                    label:"课程标题"
                },
                // {
                //     prop:"type",
                //     label:"课程类型"
                // },{
                //     prop:"total_time",
                //     label:"学习时长"
                // }
                ],
                request:fetchUserHistory
            },{
                label:"用户评论",
                name:"comment",
                ths:[
                //     {
                //     prop:"content",
                //     label:"评论内容"
                // },
                {
                    prop:"created_time",
                    label:"评论时间"
                },{
                    prop:"title",
                    label:"课程标题"
                },
                // {
                //     prop:"type",
                //     label:"类型"
                // }
                ],
                request:fetchUserComment
            }],
        }
    },
    computed:{
        currentTable(){
            // 找到名字和当前名字一样的
            // this.activeName是五个 course column order history comment
            return this.tabs.find(item => item.name == this.activeName)
        }
    },
    created(){
        this.getList()
    },
    methods:{
        open(id){
            this.dialogVisible = true
            this.id = id
            fetchUserDetail({id:this.id}).then(res=>{
                // console.log(res);
                this.user = res.data.user
            })
        },
        getList(){
            this.currentTable.request(this.listQuery).then(res=>{
                // console.log(res);
                // 一条一条的数据
                this.list = res.data.items
                this.listQuery.total = res.data.total
            })  
        },
        // 切换tab选项卡
        handleClick(tab){

        }
    }
}
</script>

<style scoped>

</style>>
