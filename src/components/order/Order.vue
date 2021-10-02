<template>
  <div>
       <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图 -->
    <el-card>
        <!-- 搜索框 -->
        <el-row>
            <el-col :span="8">
                <el-input placeholder="请输入内容" v-model="queryInfo.query">
                    <el-button slot="append" icon="el-icon-search"></el-button>
                </el-input>
            </el-col>
        </el-row>
        <!-- 表格 -->
        <el-table
            :data="ordersList"
            style="width: 100%"
            >
            <el-table-column type="index" width="50"></el-table-column>
            <el-table-column prop="order_number" label="订单编号"></el-table-column>
            <el-table-column prop="order_price" label="订单价格" width="180"></el-table-column>
            <el-table-column prop="pay_status" label="是否付款">
                <template slot-scope="scope">
                    <el-tag type="danger" v-if="scope.row.pay_status === '0'">未付款</el-tag>
                    <el-tag type="success" v-else>已付款</el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="is_send" label="是否发货"></el-table-column>
            <el-table-column prop="create_time" label="下单时间">
                <template slot-scope="scope">
                   {{scope.row.create_time | dateFormat}}
                </template>
            </el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <el-button type="primary" size="mini" icon="el-icon-edit" @click="showBox"></el-button>
                    <el-button type="success" size="mini" icon="el-icon-location" @click="showProgressBox"></el-button>
                </template>
            </el-table-column>
        </el-table>


        <!-- 分页 -->
        <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-sizes="[1, 5, 10, 15]"
            :page-size="queryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
            </el-pagination>
    </el-card>

    <!-- 修改地址对话框 -->
    <el-dialog
        title="修改地址"
        :visible.sync="addressDialogVisible"
        width="50%"
        @close="addressDialogClosed">
        <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef" label-width="100px">
            <el-form-item label="省市区/县" prop="address1">
                <el-cascader
                    v-model="addressForm.address1" 
                    :options="citydata"
                    :props="{ expandTrigger: 'hover' }"></el-cascader>
            </el-form-item>
            <el-form-item label="详细地址" prop="address2">
                <el-input v-model="addressForm.address2"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="addressDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addressDialogVisible = false">确 定</el-button>
        </span>
        </el-dialog>

        <!-- 物流进度对话框 -->
        <el-dialog
            title="物流进度"
            :visible.sync="progressDialogVisible"
            width="50%">
            <span>wuliu </span>
            <span slot="footer" class="dialog-footer">
                <el-button @click="progressDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="progressDialogVisible = false">确 定</el-button>
            </span>
            </el-dialog>

  </div>
</template>

<script>
import citydata from './citydata'
export default {
    name:'Order',
    data() {
        return {
            queryInfo:{
                query:'',
                // 当前页码
                pagenum:1,
                // 每页显示条数
                pagesize:10    

            },
            ordersList:[],
            total:0,
            addressDialogVisible:false,
            addressForm:{
                address1:[],
                address2:''
            },
            addressFormRules:{
                address1:[
                    {required:true,message:"请选择省市区/县",trigger:'blur'}
                ],
                address2:[
                    {required:true,message:"请输入详细地址",trigger:'blur'}
                ]
            },
            citydata,
            // 物流进度对话框
            progressDialogVisible:false

            
        }
    },
    created() {
        this.getOrderList()
        
    },
    methods: {
        async getOrderList(){
            const {data:res} = await this.$http.get('orders',{params:this.queryInfo})
            if(res.meta.status !== 200){
                return this.$message.error('获取订单列表失败！')
            }
            this.ordersList = res.data.goods
            this.total = res.data.total
            console.log(res.data)
        },
        // 每页条数改变改变
        handleSizeChange(pagesize){
            this.queryInfo.pagesize = pagesize
            this.getOrderList()
        },
        // 当前页改变
        handleCurrentChange(pagenum){
            this.queryInfo.pagenum = pagenum
            this.getOrderList()
        },
        // 展示修改地址对话框
        showBox(){
            this.addressDialogVisible = true
            
        },

        addressDialogClosed(){
            this.$refs.addressFormRef.resetFields()
        },
        // 物流进度
        async showProgressBox(){
            // this.progressDialogVisible = true
            // const {data:res} = await this.$http.get('/kuaidi/804909574412544580') 
            // if(res.meta.status !== 200){
            //     return this.$message.error("获取物流信息失败")
            // }
            // console.log(res.data)
        }
        
    },


}
</script>

<style scoped>
.el-cascader{
    width: 100%;
}

</style>>