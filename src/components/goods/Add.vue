<template>
  <div class="add">
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图 -->
    <el-card>
        <!-- 提示区域 -->
      <el-alert
        center
        show-icon
        title="添加商品信息"
        type="info"
        :closable="false">
      </el-alert>
      <!-- 步骤条区域 -->
      <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position='top'>

        <el-tabs tab-position="left" v-model="activeIndex" :before-leave="beforeTabLeava" @tab-click="tabClicked">

            <el-tab-pane label="基本信息" name="0">
                <el-form-item label="商品名称" prop="goods_name">
                    <el-input v-model="addForm.goods_name"></el-input>
                </el-form-item>
                <el-form-item label="商品价格" prop="goods_price">
                    <el-input type="number" v-model="addForm.goods_price"></el-input>
                </el-form-item>
                <el-form-item label="商品重量" prop="goods_weight">
                    <el-input  type="number" v-model="addForm.goods_weight"></el-input>
                </el-form-item>
                <el-form-item label="商品数量" prop="goods_number">
                    <el-input type="number" v-model="addForm.goods_number"></el-input>
                </el-form-item>
                <el-form-item label="商品分类" prop="goods_cat">
                    <el-cascader
                        v-model="addForm.goods_cat"
                        :options="cateList"
                        :props="cateProps"
                        @change="handleChange">
                    </el-cascader>
                </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品参数" name="1">
                <!-- 渲染表单的item项 -->
                <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
                    <!-- 复选框组 -->
                    <el-checkbox-group v-model="item.attr_vals">
                        <el-checkbox :label="cb" v-for="(cb,i) in item.attr_vals" :key="i" border></el-checkbox>
                        
                    </el-checkbox-group>    
                </el-form-item>    
            </el-tab-pane>
            <el-tab-pane label="商品属性" name="2">
                <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
                    <el-input v-model="item.attr_vals"></el-input>
                </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品图片" name="3">
                <!-- action表示图片要上传到的后台api地址 -->
                <el-upload
                    action="http://127.0.0.1:8888/api/private/v1/upload"
                    :on-preview="handlePreview"
                    :on-remove="handleRemove"
                    list-type="picture"
                    :headers="headerObj"
                    :on-success = "handleSuccess">
                    <el-button size="small" type="primary">点击上传</el-button>
                </el-upload>
            </el-tab-pane>
            <el-tab-pane label="商品内容" name="4">
                <quill-editor v-model="addForm.goods_introduce"></quill-editor>
                <!-- 添加商品按钮 -->
                <el-button type="primary" @click="add">添加商品</el-button>
            </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <!-- 图片预览对话框 -->
    <el-dialog
        title="图片预览"
        :visible.sync="previewVisible"
        width="50%">
        <img :src="previewUrl" alt="图片加载失败" class="privewImg">
        
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  name: "Add",
  data() {
    return {
        activeIndex:'0',
        // 添加商品的表单数据对象
        addForm:{
            goods_name:'',
            goods_price:0,
            goods_number:0,
            goods_weight:0,
            // 商品所属的分类数组
            goods_cat:[],
            // 上传的图片临时路径（对象）
            pics:[],
            // 商品的详情介绍
            goods_introduce:'',
            attrs:[]

        },
        cateList:[],
        // 动态参数列表
        manyTableData:[],
         // 静态属性列表
        onlyTableData:[],
        // 图片上传组件的headers请求头对象
        headerObj:{
            Authorization:window.sessionStorage.getItem('token')
        },
        cateProps:{ 
            expandTrigger: 'hover',
            label:'cat_name',
            value:'cat_id',
            children:'children' 
            },
        addFormRules:{
            goods_name:[
                { required: true, message: '请输入商品名称', trigger: 'blur' },
            ],
            goods_price:[
                { required: true, message: '请输入商品价格', trigger: 'blur' },
            ],
            goods_number:[
                { required: true, message: '请输入商品数量', trigger: 'blur' },
            ],
            goods_weight:[
                { required: true, message: '请输入商品重量', trigger: 'blur' },
            ],
            goods_cat:[
                { required: true, message: '请选择商品分类', trigger: 'blur' },
            ],
        },

        previewUrl:'',
        // 图片预览对话框的显示与隐藏
        previewVisible:false

    };
  },
  created() {
      this.getCateList()
  },
  methods: {
      async getCateList(){
          const {data:res} = await this.$http.get('categories')
          if(res.meta.status !== 200){
              this.$message.error("获取商品分类数据列表失败！")
          }
          console.log(res.data)
          this.cateList = res.data
      },
    //   级联选择框发生变化，触发这个函数
      handleChange(){
          if(this.addForm.goods_cat.length !== 3){
              this.addForm.goods_cat = []
          }
          console.log(this.addForm.goods_cat)
      },
    //   切换tabs标签
      beforeTabLeava(activeName,oldActiveName){
        //   console.log('即将离开的页面：' + oldActiveName)
        //   console.log('新的的页面：' + activeName)
        if(oldActiveName === '0' && this.addForm.goods_cat.length !== 3){
            this.$message.error("请先选择商品分类！")
            return false
        }
      },

      async tabClicked(){
          console.log(this.activeIndex)
        //   证明访问的是商品参数面板
          if(this.activeIndex === '1'){
              
              const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{
                  params:{sel:'many'}
              })
              if(res.meta.status !== 200){
                  return this.$message.error("获取动态参数失败！")
              }
              res.data.forEach(item => {
                  item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
                  
              });
                  console.log(res.data)
              this.manyTableData = res.data
          }else if(this.activeIndex === '2'){
              const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{
                  params:{sel:'only'}
              })
              if(res.meta.status !== 200){
                  return this.$message.error("获取静态属性失败！")
              }
              console.log(res.data)
              this.onlyTableData = res.data
          }
      },

    //   处理图片预览效果
      handlePreview(file){
          console.log(file)
          this.previewUrl = file.response.data.url
          this.previewVisible = true


      },
    //   处理移除图片的操作
      handleRemove(file){
        //   1.获取将要删除的图片的临时路径
        console.log(file)
        const filePath =file.response.data.tmp_path
        // 2.从pics数组中,找到这个图片的对应的索引值
        const i = this.addForm.pics.findIndex(x => 
        x.pic === filePath)
        //3. 调用数组的splice方法,把图片信息对象从pics中移除
        this.addForm.pics.splice(i,1)
        console.log(this.addForm)
      },
    //   监听图片上传成功的事件
    handleSuccess(response){
        console.log(response)
        // 1.拼接得到一个图片信息对象
        const picInfo = {pic:response.data.tmp_path}
        // 2.将图片信息对象，push到pics数组中
        this.addForm.pics.push(picInfo)

        console.log(this.addForm)
    },
    // 添加商品
    add(){
        this.$refs.addFormRef.validate(async valid =>{
            if(!valid){
                return this.$message.error('请填写必要的表单项!')
            }

            // 引入深拷贝
            const form = _.cloneDeep(this.addForm)
            form.goods_cat = form.goods_cat.join(',')
            // 处理动态参数
            this.manyTableData.forEach(item =>{
                const newInfo = {
                    attr_id:item.attr_id,
                    attr_value:item.attr_vals.join(' ')
                }
                    this.addForm.attrs.push(newInfo)
            })
            // 处理静态属性
            this.onlyTableData.forEach(item =>{
                const newInfo = {
                    attr_id:item.attr_id,
                    attr_value:item.attr_vals
                }
                    this.addForm.attrs.push(newInfo)
            })
            form.attrs = this.addForm.attrs
            console.log(form)
            // 添加商品,
            const {data:res} =await this.$http.post('goods',form)
            if(res.meta.status !== 201){
                return this.$message.error("添加商品失败!")
            }

            this.$message.success("添加商品成功!")
            this.$router.push('/goods')

        })
    } 
  },
  computed:{
      cateId(){
          if(this.addForm.goods_cat.length === 3){
              return this.addForm.goods_cat[2]
          }
          return null
      }
  }
};
</script>

<style scoped>
    .el-steps{
        margin: 15px 0px;
    }
    .el-step__title{
        font-size: 13px;
    }
    .el-checkbox{
        margin: 0 10px 0 0;
    }
    .privewImg{
        width: 100%;
    }
    /* vue引用了第三方组件，需要在组件中局部修改第三方组件的样式，而又不想去除scoped属性造成组件之间的样式污染。此时只能通过>>>，穿透scoped。 */
    .add >>> .ql-editor { 
        min-height: 300px;
    }
    .el-button{
        margin-top: 10px;
    }
  
</style>>
