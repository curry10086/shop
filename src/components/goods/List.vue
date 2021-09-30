<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图 -->
    <el-card>
      <!-- 搜索栏区 -->
      <el-row :gutter="20">
        <el-col :span = "8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span = "4">
          <el-button type="primary" @click="goAddPage">添加商品</el-button>
        </el-col>
      </el-row>

      <!-- 表格区域 -->
      <el-table :data="goodsList" stripe style="width: 100%" border>
        <el-table-column type="index" width="50"> </el-table-column>
        <el-table-column prop="goods_name" label="商品名称"></el-table-column>
        <el-table-column prop="goods_price" label="商品价格(元)" width="100px"></el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="80px"> </el-table-column>
        <el-table-column prop="add_time" label="创建时间" width="150px"> 
            <template slot-scope="scope">
                {{scope.row.add_time | dateFormat}}
            </template>
        </el-table-column>
        <el-table-column label="操作" width="140px"> 
           <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit"></el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeGoodsList(scope.row.goods_id)"></el-button>
           </template>
         
        </el-table-column>
      </el-table>


      <!-- 页码部分、 -->
      <div class="block">
        <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page.sync="queryInfo.pagenum"
            :page-sizes="[5, 10, 15,20]"
            :page-size="queryInfo.pagesize"
            layout="total, sizes,prev, pager, next, jumper"
            :total="total"
            background>
        </el-pagination>
    </div>
    </el-card>
  </div>
</template>

<script>
export default {
  name: "List",
  data() {
    return {
        goodsList:[],
        queryInfo:{
            query:'',
            // 当前页数
            pagenum:1,
            // 每页显示条数
            pagesize:10

        },
        total:0
        

    };
  },
  created() {
      this.getGoodsList()
  },
  methods: {
    //   根据分页回去对应的商品列表
      async getGoodsList(){
          const {data:res} = await this.$http.get('goods',{params:this.queryInfo})
          if(res.meta.status !== 200){
              return this.$message.error("获取商品列表失败1")
          }
          this.total = res.data.total
          this.goodsList = res.data.goods
      },
      handleSizeChange(newSize){
          this.queryInfo.pagesize = newSize
          this.getGoodsList()
      },
      handleCurrentChange(newPage){
          this.queryInfo.pagenum = newPage
          this.getGoodsList()
      },

      async removeGoodsList(id){
                const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                    }).catch(err => err)
                // 用户取消操作
                if(confirmResult !== 'confirm'){
                    return this.$message.info('已经取消删除!')
                    
                }
                // 删除逻辑
                const {data:res} = await this.$http.delete(`goods/${id}`)
                if(res.meta.status !== 200){
                    return this.$message.error('删除失败!')
                }

                this.$message.success('删除成功!')
                this.getGoodsList()
      },

      goAddPage(){
          this.$router.push('/goods/add')
      }
  },
};
</script>

<style scoped>

</style>> 