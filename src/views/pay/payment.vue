<template>
    <div>
        <el-button @click="handleCreate" style="margin:20px 0 0 40px" type="primary" ><svg-icon icon-class="edit" />新增收款账号</el-button>
        
        <el-table :data="tableData" ref="multipleTable" border class="el-table" @selection-change="sortChange">

            <el-table-column align="center" label="账号">
                <template slot-scope="scope">
                    {{ scope.row.account }}
                </template>
            </el-table-column>

            
            <el-table-column align="center" label="开户人" width="120">
                <template slot-scope="scope">
                    {{ scope.row.name }}
                </template>
            </el-table-column>

            <el-table-column align="center" label="开户行" width="300">
                <template slot-scope="scope">
                    {{ scope.row.path }}
                </template>
            </el-table-column>

            <el-table-column align="center" label="操作" width="260">
                <template slot-scope="{row,$index}">
                    <el-button type="primary" size="small" @click="createUpdate(row)" >编辑</el-button>
                    <el-button type="danger" size="small" @click="deleteRow(row,$index)" >删除</el-button>
                </template> 
            </el-table-column>
        </el-table>
        <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />
    
        <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">

            <el-form ref="dataForm" :rules="rules" :model="temp" style="width: 400px; margin-left:100px;">
                <el-form-item label="账号" prop="account">
                    <el-input  v-model="temp.account" placeholder="请输入账号" />
                </el-form-item>

                <el-form-item label="省市区">
                     <VDistpicker 
                        :province="temp.province" 
                        :city="temp.city" 
                        :area="temp.area" 
                        @province="handleDistPicker($event,'province')"
                        @city="handleDistPicker($event,'city')"
                        @area="handleDistPicker($event,'area')"
                        style="width:280px"  
                        placeholder="请选择">
                    </VDistpicker>
                </el-form-item>
                
                <el-form-item label="详细地址" prop="path">
                    <el-input v-model="temp.path" placeholder="请输入账号" />
                </el-form-item>

                <el-form-item label="所属银行" prop="bank">
                    <el-select v-model="temp.bank" style="width:280px"  placeholder="请选择">
                        <el-option  v-for="(item,index) in banks" :key="index" :label="item.text" :value="item.text">
                        </el-option>
                    </el-select>
                </el-form-item>

                <el-form-item label="姓名" prop="name">
                    <el-input v-model="temp.name" />
                </el-form-item>

                <el-form-item style="margin:20px 0 0 100px">
                    <el-button @click="dialogFormVisible = false">取消</el-button>
                    <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">提交</el-button>
                </el-form-item>
            </el-form>
        </el-dialog>
    </div>
</template>

<script>
import { fetchPayments,deletePayment,createPayment,updatePayment } from '../../api/pay'
import Pagination from '@/components/Pagination'
import VDistpicker from "v-distpicker"  // 需要安装
import {getBank} from "@/utils/bank"

export default {
    name:"Payment",
    components:{
        Pagination,VDistpicker
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
            dialogFormVisible:false, 
            dialogStatus:"",        // 增加还是编辑   
            textMap:{
                update:"编辑",
                create:"新增"
            } ,
            temp:{
                account:"",
                province:"",
                city:"",
                area:"",
                path:"",
                bank:"",
                name:"",
                id:""
            },
            banks:[],
            rules: {
                account: [{
                    required: true,
                    message: '账户不能为空',
                    trigger: 'blur'
                }],
                bank: [{
                    required: true,
                    message: '所属银行不能为空',
                    trigger: 'change'
                }],
                name: [{
                    required: true,
                    message: '开户人不能为空',
                    trigger: 'blur'
                }],
                path: [{
                    required: true,
                    message: '详细地址不能为空',
                    trigger: 'blur'
                }],
            },
        }
        
    },
    created() {
        this.getList()   
        this.banks = getBank() 
    },
    methods:{
        // 获取列表
        async getList(){
            await fetchPayments(this.listQuery).then((res) => {
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
        sortChange(){

        },
        // 点击删除
        deleteRow(row,index){
            deletePayment(row).then(res=>{
                // console.log(res);
                this.tableData.splice(index,1)
                this.$confirm("是否确定删除",'提示',{
                    confirmButtonText:"确定",
                    cancelButtonText:"取消",
                    type:"warning"
                }).then(()=>{
                    this.$message({
                        message:res.data,
                        type:'success'
                    })
                }).catch(()=>{
                    this.$message({
                        type:'info',
                        message:'已取消删除'
                    })
                })
                
            })
        },
        // 置空
        resetForm(){
            this.temp = {
                account:"",
                province:"",
                city:"",
                area:"",
                path:"",
                bank:"",
                name:"",
            }
        },
        // 点击新增
        handleCreate(){
            this.dialogFormVisible = true   
            this.dialogStatus = 'create'
            this.resetForm()
            this.$nextTick(() => {
                this.$refs['dataForm'].clearValidate()
            })
        },
        createData(){
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    if(this.temp.province == '' || this.temp.city == '' || this.temp.area == ''){
                        return this.$message({
                            message:"省市区不能为空",
                            type:"error"
                        })
                    }
                    createPayment(this.temp).then(() => {
                        // 把增加的东西放到表格的第一个
                        this.tableData.unshift(this.temp)
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
        createUpdate(row){
            // 编辑的时候实现数据回显 把这一行的数据先添加到temp中
            this.temp = Object.assign({},row)
            this.dialogFormVisible = true   
            this.dialogStatus = 'update'
            this.$nextTick(() => {
                this.$refs['dataForm'].clearValidate()
            })
        },
        updateData(){
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    if(this.temp.province == '' || this.temp.city == '' || this.temp.area == ''){
                        return this.$message({
                            message:"省市区不能为空",
                            type:"error"
                        })
                    }
                    // 新的修改过的temp 是tempData
                    const tempData = Object.assign({}, this.temp)
                    updatePayment(tempData).then(() => {
                        const index = this.tableData.findIndex(v => v.id === this.temp.id)
                        this.tableData.splice(index, 1, this.temp)
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
        // 处理省市区
        handleDistPicker(e,k){
            this.temp[k] = e.value
        }
    },
}
</script>

<style scoped>
.el-table {
  width: 96%;
  text-align: center;
  margin: 20px auto;
}
</style>