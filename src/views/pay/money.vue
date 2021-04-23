<template>
    <div>
        <!-- gutter每一栏之间的距离 -->
        <el-row :gutter="20" >
            <el-col :span="6" :offset="0">
                <el-card shadow="never" style="margin:20px">
                    <div slot="header">
                        <span>可用余额(元)</span>
                        <el-button @click="applyMoney" style="float:right" type="primary" size="small">申请提现</el-button>
                    </div>
                    <div style="font-weight:600;font-size:28px">
                        <span>20000000.00</span>
                    </div>
                </el-card>
            </el-col>

            <el-col :span="6" :offset="0">
                <el-card shadow="never" style="margin:20px">
                    <div slot="header">
                        <span>累计收入(元)</span>
                    </div>
                    <div style="font-weight:600;font-size:28px">
                        <span>20000000.00</span>
                    </div>
                </el-card>
            </el-col>

            <el-col :span="6" :offset="0">
                <el-card shadow="never" style="margin:20px">
                    <div slot="header">
                        <span>待结算金额(元)</span>
                    </div>
                    <div style="font-weight:600;font-size:28px">
                        <span>00.00</span>
                    </div>
                </el-card>
            </el-col>

            <el-col :span="6" :offset="0">
                <el-card shadow="never" style="margin:20px">
                    <div slot="header">
                        <span>冻结金额(元)</span>
                    </div>
                    <div style="font-weight:600;font-size:28px">
                        <span>00.00</span>
                    </div>
                </el-card>
            </el-col>
        </el-row>

        <el-table algin="center" :data="tableData" ref="multipleTable" border @selection-change="sortChange">
            <el-table-column align="center" label="交易时间" width="180">
                <template slot-scope="scope">
                    <p>{{ scope.row.created_time }}</p>
                </template>
            </el-table-column>

            <el-table-column  align="center" label="提款账号">
                <template slot-scope="scope">
                    <p>{{ scope.row.account }}</p>
                </template>
            </el-table-column>
            
            <el-table-column  align="center" label="开户人" width="80">
                <template slot-scope="scope">
                    <p>{{ scope.row.name }}</p>
                </template>
            </el-table-column>

            <el-table-column  align="center" label="提现金额" width="380">
                <template slot-scope="scope">
                    <p>￥{{ scope.row.price }}</p>
                </template>
            </el-table-column>
            
            <el-table-column  align="center" label="状态" width="80">
                <template slot-scope="scope">
                    <el-tag :type="scope.row.status == 'success' ? 'success' : 'danger'" size="small">
                        {{ scope.row.status | statusFilters }}
                    </el-tag>
                </template>
            </el-table-column>

        </el-table>

        <pagination :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />
    
        <el-dialog title="提现" :visible.sync="dialogFormVisible">
            <el-form ref="dataForm" :model="temp">
                <el-form-item style="margin-left:10px" label="提现金额">
                    <el-input-number v-model="temp.price" :min="10" :max="10000" ></el-input-number>
                    <p style="color:red;margin:-2px 0 -10px 68px">提现金额最低为10，最高为10000</p>
                </el-form-item>
                <el-form-item  label="提现账户" prop="payment_id">
                    <el-select v-model="temp.payment_id" style="width:280px"  placeholder="请选择">
                        <el-option  v-for="(item,index) in payments" :key="index" :label="item.bank" :value="item.id">
                            <span>{{item.bank}}</span>
                            <span style="margin-left:20px">{{item.account}}</span>
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item style="margin-left:78px">
                    <el-button @click="dialogFormVisible = false">取消</el-button>
                    <el-button type="primary" @click="createOK">确定</el-button>
                </el-form-item>
            </el-form>
        </el-dialog>
    </div>
</template>

<script>
import { fetchAssets,fetchPayments,Cashout } from '../../api/pay'
import Pagination from '@/components/Pagination' 
let statusOptions = {
    0:"审核中",
    1:"提现成功",
    2:"提现失败",
}
export default {
    name:"Money",
    components:{
        Pagination
    },
    data(){
        return{
            tableData:[],
            total:0,
            listQuery: {
                page: 1,
                limit: 20,
            },
            // 弹窗绑定的数据
            temp:{
                price:0,
                payment_id:null
            },
            dialogFormVisible:false,
           
            payments:[]
        }
    },  
    filters:{
        statusFilters(val){
            return statusOptions[val]
        }
    },
    created(){
        this.getList()
        this.getPayList()
    },
    methods:{
        // 获取列表
        getList(){
            fetchAssets(this.listQuery).then(res=>{
                // console.log(res);
                this.tableData = res.data.items
            })
        },
        // 获取支付方式
        getPayList(){
            fetchPayments().then(res=>{
                // console.log(res);
                this.payments = res.data.items
            })
        },
        sortChange(){

        },
        resetTemp(){
            this.temp = {
                price:0,
                payment_id:null
            }
        },
        // 申请提现
        applyMoney(){
            this.resetTemp()
            this.dialogFormVisible = true
            this.$nextTick(()=>{
                this.$refs.dataForm.clearValidate()
            })
        },
        // 确认提现
        createOK(){
            this.$refs.dataForm.validate((valid)=>{
                if(valid){
                    Cashout(this.temp).then(res=>{
                        // console.log(res);
                        this.tableData.unshift(res.data)
                        this.dialogFormVisible = false
                        this.$notify({
                            title:"success",
                            message:"申请成功，等待审核",
                            type:"success",
                            duration:2000
                        })
                    })
                }
            })
        }
    }
}
</script>

<style scoped>
.el-table {
    width: 96%;
    text-align: center;
    margin: 20px auto;
}
</style>