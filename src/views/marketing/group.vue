<template>
    <div>
        <div class="select-title-btn">
            <el-button class="add-btn" type="primary" @click="openAdd"><svg-icon icon-class="edit" />创建拼团</el-button>
        </div>
        <!-- @sort-change 排序条件发生变化时触发 -->
        <!-- sortable="custom" 是否可以排序 custom表示可以-->
        <!-- class-name 列的类名 -->
        <el-table :data="tableData"  border class="el-table">
            

            <el-table-column label="拼团内容">
                <div slot-scope="scope" style="display: flex;flex-wrap:nowrap">
                    <div>
                        <el-image style="width: 100px; height: 50px;margin-right:10px" :src="scope.row.value.cover" />
                    </div>
                    <div >
                        <div style="displsy:flex;flex-wrap:wrap">
                            <div>
                                {{ scope.row.value.title }}
                            </div>
                            <div style="font-size:12px">
                                <div>
                                    原始价格：
                                    <span style="color:red;margin-top:10px">￥{{ scope.row.value.price }}</span>
                                </div>
                                <div>
                                    拼团价格：
                                    <span style="color:red;margin-top:10px">￥{{ scope.row.price }}</span>
                                </div>
                            </div>
                            
                        </div>
                    </div>
                </div>
            </el-table-column>
            
            <el-table-column align="center" label="成团人数" width="120">
                <template slot-scope="scope">
                    {{ scope.row.p_num }}
                </template>
            </el-table-column>
            <el-table-column  align="center" label="拼团时限(小时)" width="160">
                <template slot-scope="scope">
                    {{ scope.row.expire }}
                </template>
            </el-table-column>
            <el-table-column align="center" label="状态" width="120">
                <template slot-scope="scope">
                    <p :style="scope.row.status == 1 ? 'color:red' : 'color:#bbb' ">
                        {{ scope.row.status | statusFormat }}
                    </p>
                </template>
            </el-table-column>
            
            <el-table-column align="center" label="操作" width="280">
                <template slot-scope="{row}">
                    <el-button type="primary" size="small" @click="handleEdit(row)">编辑</el-button>
                    <el-button type="danger" size="small" :disabled="row.status == 0" @click="handleChangeStatus(row)">下架</el-button>
                </template> 
            </el-table-column>
        </el-table>

        <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />

        <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
            <el-form ref="dataForm" :model="rulesForm" :rules="rules" style="width: 600px; margin-left: 50px" >
                <el-form-item>新增</el-form-item>
                <el-form-item label="类型" prop="title" label-width="96px">
                    <el-select v-model="rulesForm.type" style="width: 300px; ">
                        <el-option label="课程" value="course"></el-option>
                        <el-option label="专栏" value="column"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label-width="96px" :label="rulesForm.type == 'course' ? '关联课程' : '关联专栏'" prop="value">
                    <el-button size="small" type="primary" @click="connectValue">关联</el-button>
                    <div v-if="rulesForm.value" style="display:flex">
                        <el-card shadow="always">
                            <div>
                                <img :src="rulesForm.value.cover" alt="">
                            </div>
                            <div>{{rulesForm.value.title}}</div>
                            <div style="color:red">￥{{rulesForm.value.price}}</div>
                        </el-card>
                    </div>
                </el-form-item>

                <el-form-item label="拼团价" prop="price" label-width="96px">
                    <el-input-number v-model="rulesForm.price" :min="1" :step="1" :controls="true"></el-input-number>
                </el-form-item>

                <el-form-item label="拼团人数" prop="p_num" label-width="96px">
                    <el-input-number v-model="rulesForm.p_num" :min="1" :step="1" :controls="true"></el-input-number>
                </el-form-item>

                <el-form-item label="是否开启拼团">
                    <el-radio-group v-model="rulesForm.auto">
                        <el-radio :label="1">是</el-radio>
                        <el-radio :label="0">否</el-radio>
                    </el-radio-group>
                </el-form-item>

                <el-form-item label="拼团时限" label-width="96px" prop="expire">
                    <el-radio-group v-model="rulesForm.expire">
                        <el-radio :label='24'>24小时</el-radio>
                        <el-radio :label='48'>48小时</el-radio>
                    </el-radio-group>
                </el-form-item>

                <el-form-item label="拼团活动时间">
                     <el-date-picker v-model="timerange" type="daterange" start-placeholder="开始日期" end-placeholder="结束日期" ></el-date-picker>
                </el-form-item>
                
                <el-form-item style="margin:20px 0 0 150px">
                    <el-button @click="quitAdd">取消</el-button>
                    <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">提交</el-button>
                </el-form-item>
            </el-form>
        </el-dialog>

        <chooseCourse ref="chooseCourse"></chooseCourse>
    </div>
</template>

<script>
import { fetchGroup,createGroup,updateGroup } from '../../api/marketing'
import Pagination from '@/components/Pagination'  
import chooseCourse from "@/components/chooseCourse/index"

const statusOptions = {
    1:"拼团中",
    0:'已下架'
}

export default {
    components:{
        Pagination,chooseCourse
    },
    filters: {
        statusFormat(status) {
            return statusOptions[status]
        }
    },
    computed:{
        timerange:{
            get(){
                return [this.rulesForm.start_time,this.rulesForm.end_time]
            },
            // 重新赋值时 执行set  默认不执行
            set(val){
                this.rulesForm.start_time = val[0]
                this.rulesForm.end_time = val[1]
            }
        }
    },
    data() {
        return {
            tableData: [],      // 表格内容
            total:0,
            dialogFormVisible: false,       // 增加的对话框是否显示
            dialogVisible:false,
            dialogImageUrl:"",
            statusOptions,
            page:1,
            sort:'+id',
            num: 1, // 添加时的基础价格
            rulesForm: {    // 填写数据列表
                id: undefined,
                type: 'course',
                value: {
                    id:"",
                    title:"",
                    cover:"",
                    price:""
                },
                price: 0.00,
                p_num: 2,
                auto: 1,        // 是否启用拼团
                expire: 24,
                start_time: '',
                end_time: '',
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
                type: [{
                    required: true,
                    message: '类型不能为空',
                    trigger: 'change'
                }],
                value: [{
                    required: true,
                    message: '关联课程/专栏不能为空',
                    trigger: 'blur'
                }],
                price: [{
                    required: true,
                    message: '拼团价不能为空',
                    trigger: 'blur'
                }],
                p_num: [{
                    required: true,
                    message: '拼团人数不能为空',
                    trigger: 'blur'
                }],
                expire: [{
                    required: true,
                    message: '拼团时限不能为空',
                    trigger: 'blur'
                }],
            },
        }
        
    },
    created() {
        this.getList()    
    },
    methods: {
        // 获取列表
        async getList(){
            await fetchGroup(this.listQuery).then((res) => {
                // console.log(res)
                if (res.code === 20000) {
                    this.tableData = res.data.items,
                    this.total = res.data.total
                }
            })
            .catch((err) => {
                console.log('错误')
            })
        },
        // 关联
        connectValue(){
            this.$refs.chooseCourse.open(val=>{
                // console.log("...");
                console.log(val);
                let item = val[0]
                this.rulesForm.value = {
                    id:item.id,
                    title:item.title,
                    cover:item.cover,
                    price:item.price
                }
                this.$refs.dataForm.clearValidate()
            })
        },
        // 取消添加
        quitAdd(){
            this.dialogFormVisible = false
        },
        // 清空表单
        resetForm(){
            this.rulesForm = {
                id: undefined,
                type: 'course',
                value: null,
                price: 0.00,
                p_num: 2,
                auto: 1,        // 是否启用拼团
                expire: 24,
                start_time: '',
                end_time: '',
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
                    createGroup(this.rulesForm).then(() => {
                        this.dialogFormVisible = false
                        this.$notify({
                            title: 'Success',
                            message: 'Created Successfully',
                            type: 'success',
                            duration: 2000
                        })
                        // this.getList()
                        this.tableData.unshift(this.rulesForm)
                    })
                }
            })
        },
        // 点击编辑
        handleEdit(row) {
            this.rulesForm = Object.assign({}, row) 
            this.dialogStatus = 'update'
            this.dialogFormVisible = true
            this.$nextTick(() => {
                this.$refs.dataForm.clearValidate()
            })
        },
        // 提交编辑
        updateData() {
            this.$refs.dataForm.validate((valid) => {
                if (valid) {
                    const tempData = Object.assign({}, this.rulesForm)
                    updateGroup(tempData).then(() => {
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
            deleteMedia(row).then((res)=>{
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
        handleChangeStatus(row){
            this.$confirm("是否确定下架",'提示',{
                confirmButtonText:"下架",
                cancelButtonText:"取消",
                type:"warning"
            }).then(action=>{
                row.status = 0
                this.$message({
                    message:"下架成功",
                    type:"success"
                })
            })
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
