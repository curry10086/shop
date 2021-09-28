<template>
  <div>
      <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片试图 -->
    <el-card>
        <!-- 头部警告 -->
        <el-alert
            title="注意:只允许为第三级分类设置相关参数!"
            type="warning"
            :closable="false"
             show-icon>
        </el-alert>

        <!-- 选择商品分类区域 -->
        <el-row class="el-row">
            <el-col>
                <span>选择商品分类：</span>
                <!-- 选择商品分类的级联选择框 -->
                <el-cascader v-model="selectedCateKeys" :options="cateList" :props="cateProps" @change="handleChange"></el-cascader>
            </el-col>
        </el-row>

         <el-tabs v-model="activeName" @tab-click="handleTabClick">
            <el-tab-pane label="动态参数" name="many">
                <el-button type="primary" size="mini"
                :disabled="isBtnDisabled"
                @click="addDialogVisible=true">添加参数
                </el-button>
                <!-- 动态参数列表 -->
                <el-table :data="manyTableData" border stripe>
                    <!-- 展开行 -->
                    <el-table-column type="expand">
                        <template slot-scope="scope">
                            <!-- 循环渲染Tag标签 -->
                            <el-tag 
                                v-for="(item,i) in scope.row.attr_vals" 
                                :key="i" closable
                                @close="handleClose(i,scope.row)">
                                {{item}}
                            </el-tag>
                            <!-- 输入文本框 -->
                            <el-input
                                class="input-new-tag"
                                v-if="scope.row.inputVisible"
                                v-model="scope.row.inputValue"
                                ref="saveTagInput"
                                size="small"
                                @keyup.enter.native="handleInputConfirm(scope.row)"
                                @blur="handleInputConfirm(scope.row)">
                            </el-input>
                            <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                        </template>
                    </el-table-column>
                    <!-- 索引列 -->
                    <el-table-column type="index"></el-table-column>
                    <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-button type="primary" size="minni" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                            <el-button type="danger" size="minni" icon="el-icon-delete" @click="removeParame(scope.row.attr_id)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-tab-pane>
            <el-tab-pane label="静态属性" name="only">
                <el-button type="primary" size="mini"
                :disabled="isBtnDisabled"
                @click="addDialogVisible=true">添加属性
                </el-button>
                <!-- 静态属性参数列表 -->
                <el-table :data="onlyTableData" border stripe>
                    <!-- 展开行 -->
                    <el-table-column type="expand">
                        <template slot-scope="scope">
                            <!-- 循环渲染Tag标签 -->
                            <el-tag 
                                v-for="(item,i) in scope.row.attr_vals" 
                                :key="i" closable
                                @close="handleClose(i,scope.row)">
                                {{item}}
                            </el-tag>
                            <!-- 输入文本框 -->
                            <el-input
                                class="input-new-tag"
                                v-if="scope.row.inputVisible"
                                v-model="scope.row.inputValue"
                                ref="saveTagInput"
                                size="small"
                                @keyup.enter.native="handleInputConfirm(scope.row)"
                                @blur="handleInputConfirm(scope.row)">
                            </el-input>
                            <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                        </template>
                    </el-table-column>
                    <!-- 索引列 -->
                    <el-table-column type="index"></el-table-column>
                    <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-button type="primary" size="minni" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                            <el-button type="danger" size="minni" icon="el-icon-delete" @click="removeParame(scope.row.attr_id)">删除</el-button>
                        </template>
                    </el-table-column>
            </el-table>
            </el-tab-pane>
        </el-tabs>
    </el-card>

    <!-- 添加参数对话框 -->
    <el-dialog
        :title="'添加'+ titleText"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClosed">
        <!-- 对话框主体区域 -->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
            <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="addForm.attr_name"></el-input>
            </el-form-item>
        </el-form>
        <!-- 对话框底部区域 -->
        <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addParams">确 定</el-button>
        </span>
    </el-dialog>  

    <!-- 修改对话框 -->
    <el-dialog
        :title="'修改'+ titleText"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClosed">
        <!-- 对话框主体区域 -->
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
            <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="editForm.attr_name"></el-input>
            </el-form-item>
        </el-form>
        <!-- 对话框底部区域 -->
        <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editParams">确 定</el-button>
        </span>
    </el-dialog>    

  </div>
</template>

<script>
    export default {
        name:'Params',
        data() {
            return {
                cateList:[],
                cateProps:{
                    expandTrigger: 'hover' ,
                    value:'cat_id',
                    label:'cat_name',
                    children:'children'
                },
                selectedCateKeys:[],

                activeName:'many',

                manyTableData:[],
                onlyTableData:[],
                addDialogVisible:false,
                // 添加参数表单的数据对象
                addForm:{
                    attr_name:''
                },
                // 添加表单的验证规则对象
                addFormRules:{
                    attr_name:[
                        { required: true, message: "请输入参数名称", trigger: "blur" },
                    ]
                },

                editDialogVisible:false,
                 // 修改表单的数据对象
                editForm:{
                    attr_name:''
                },
                // 修改表单的验证规则对象
                editFormRules:{
                    attr_name:[
                        { required: true, message: "请输入参数名称", trigger: "blur" },
                    ]
                },
                
            }
        },
        created() {
            this.getCateList()
            
        },
        methods: {
            // 获取所有商品的分类
            async getCateList(){
                const {data:res} = await this.$http.get('categories')

                if(res.meta.status !== 200){
                    return this.$message.error('获取父级分类数据失败！')
                }
                this.cateList = res.data
                console.log(this.cateList)
                
            },

            async handleChange(){
                this.getParamsData()

            },
            async getParamsData(){
                if(this.selectedCateKeys.length !== 3){
                    this.selectedCateKeys = []
                    this.manyTableData = []
                    this.onlyTableData = []
                    return
                }
                const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params:{
                    sel:this.activeName
                }})

                if(res.meta.status !== 200){
                    return this.$message.error("获取属性失败")
                }

                res.data.forEach(item => {
                    item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
                    // 控制文本框的显示与隐藏
                    item.inputVisible = false
                    // 文本框中输入的内容
                    item.inputValue = ''
                });

                console.log(res.data)
                if(this.activeName === 'many'){
                    this.manyTableData = res.data

                }else{
                    this.onlyTableData = res.data
                }
            },

            handleTabClick(){
                this.getParamsData()
            },

            addDialogClosed(){
                this.$refs.addFormRef.resetFields()
            },

            addParams(){
                this.$refs.addFormRef.validate(async valid => {
                    if(!valid) return
                    const {data:res} = await this.$http.post(`categories/${this.cateId}/attributes`,
                    {
                        attr_name:this.addForm.attr_name,
                        attr_sel:this.activeName
                    })
                    console.log(res)

                    if(res.meta.status !== 201){
                        return this.$message.error('添加分类失败')
                    }
                    this.$message.success('添加成功！')
                    this.getParamsData()
                    this.addDialogVisible = false
                })
            },
            // 点击按钮,展示修改对话框
            async showEditDialog(attr_id){
                // 查询当前参数的信息
               const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`,{
                    params:{
                        attr_sel:this.activeName
                    }
                })

                if(res.meta.status !== 200){
                    return this.$message.error('根据id查询失败!')
                }
                this.editForm= res.data
                this.editDialogVisible = true
            },
            editDialogClosed(){
                // 重置修改的表单
                this.$refs.editFormRef.resetFields()
            },
            // 点击确认,修改参数信息
            editParams(){
                this.$refs.editFormRef.validate( async valid =>{
                    if(!valid) return
                    const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,{
                        attr_name:this.editForm.attr_name,
                        attr_sel:this.activeName
                    })

                    if(res.meta.status !== 200){
                        return this.$message.error('更新失败')
                    }
                    this.getParamsData()
                    this.editDialogVisible = false
                })
            },
            // 根据id删除对应的参数项            
            async removeParame(attr_id){
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
                const {data:res} = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)
                if(res.meta.status !== 200){
                    return this.$message.error('删除失败!')
                }

                this.$message.success('删除成功!')
                this.getParamsData()
                
            },

            async handleInputConfirm(row){
                if(row.inputValue.trim().length === 0){
                    row.inputValue = ''
                    row.inputVisible = false
                    return
                }
                row.attr_vals.push(row.inputValue.trim())
                row.inputValue = ''
                row.inputVisible = false

                this.saveAttrVals(row)

            },
            // 将对attr_vals的操作保存到数据库
            async saveAttrVals(row){
                const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,{
                    attr_name:row.attr_name,
                    attr_sel:row.attr_sel,
                    attr_vals:row.attr_vals.join(' ')
                })

                if(res.meta.status !== 200){
                    return this.$message.error("更新新的参数失败!")
                }
                this.$message.success('更新成功!')
            },
            showInput(row){
                row.inputVisible = true
                this.$nextTick(_ => {
                    this.$refs.saveTagInput.$refs.input.focus();
                });
            },
            handleClose(i,row){
                row.attr_vals.splice(i,1)
                this.saveAttrVals(row)

            }
        },
        computed:{
            isBtnDisabled(){
                if(this.selectedCateKeys.length !== 3){
                    return true
                }
                return false
            },

            cateId(){
                if(this.selectedCateKeys.length === 3){
                    return this.selectedCateKeys[this.selectedCateKeys.length - 1]
                }
                return null
            },
            // 动态计算标题的文本
            titleText(){
                return this.activeName === 'many' ? '动态参数' : '静态属性'
            }
        }
    }
</script>

<style scoped>
    .el-row{
        margin:15px 0;
    }
    .el-tag{
        margin:0px 10px;
    }
    .input-new-tag{
        width: 150px;
    }

</style>>