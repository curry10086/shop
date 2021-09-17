<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card class="box-card">
        <!-- 添加角色 -->
        <el-row>
            <el-col>            
                <el-button type="primary" >添加角色</el-button>
            </el-col>
        </el-row>
        <!-- 角色列表 -->
        <el-table :data="rolesList" border style="width: 100%" stripe>
            <!-- 展开列 -->
            <el-table-column type="expand">
                <template slot-scope="scope">
                    <el-row :class="['bdbottom', i1===0?'bdtop':'','vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                        <!-- 渲染一级权限 -->
                        <el-col :span="5">
                            <el-tag closable>{{item1.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>
                        </el-col>
                        <!-- 渲染二级权限 -->
                        <el-col :span="19">
                            <!-- 通过for循环，嵌套渲染二级权限 -->
                            <el-row v-for="(item2,i2) in item1.children" :key="item2.id" :class="[i2!==0?'bdtop':'','vcenter']">
                                <el-col :span="6">
                                    <el-tag type="success" closable>{{item2.authName}}</el-tag>
                                    <i class="el-icon-caret-right"></i>
                                </el-col>
                                <!-- 渲染三级权限 -->
                                <el-col :span="18">
                                   <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row,item3.id)">{{item3.authName}}</el-tag>  
                                </el-col>
                            </el-row>
                        </el-col>
                    </el-row>

                </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="roleName" label="角色名称" width="180"></el-table-column>
            <el-table-column prop="roleDesc" label="角色描述" width="180"></el-table-column>
            <el-table-column label="操作" >
                <template slot-scope="scope">
                    <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
                    <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                    <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-card>
    <!-- 分配权限的对话框 -->
    <el-dialog
        title="分配权限"
        :visible.sync="setRightDialogVisble"
        width="50%"
        @close="setRightDialogClosd">
        <!-- 树形控件 -->
        <el-tree show-checkbox node-key="id" 
         default-expand-all
         :data="rightsList" 
         :props="treeProps"
         :default-checked-keys="defKeys"
         ref="treeRef"></el-tree>
        <span slot="footer" class="dialog-footer">
            <el-button @click="setRightDialogVisble = false">取 消</el-button>
            <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
    </el-dialog>
  </div>
</template>

<script>
    export default {
        name:'Roles',
        data() {
            return {
                // 所有角色列表数据
                rolesList:[],
                // 控制分配权限对话框的显示与隐藏
                setRightDialogVisble:false,
                // 所有权限的数据
                rightsList:[],
                treeProps:{
                    children:'children',
                    label:'authName'
                },
                defKeys:[],
                // 当前即将分配权限的id
                roleId:''
            }
        },
        created() {
            this.getRolesList()
        },
        methods: {
            // 获取角色列表
            async getRolesList(){
                const {data:res} = await this.$http.get('roles')
                if(res.meta.status !== 200){
                    return this.$message.error('获取权限错误')
                }
                this.$message.success('获取成功!')
                this.rolesList = res.data
                // console.log(res)
            },
            // 根据id删除对应的权限
            async removeRightById(role,rightId){
                console.log(role)
                // 弹框提示用户是否要删除
                const cofirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                    }).catch(err => err)
                 // 删除成功,返回confirm,失败返回cancel
                if(cofirmResult !== 'confirm'){
                    return this.$message.info('已取消删除')
                }

                const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
                console.log(res)
                 if(res.meta.status !== 200){
                    return this.$message.error("删除失败")
                }
                role.children = res.data

            },
            // 展示分配权限的对话框
            async showSetRightDialog(role){
                this.roleId = role.id
                // 获取所有的权限数据
                const {data:res} = await this.$http.get('rights/tree')
                if(res.meta.status !== 200){
                    this.$message.error("请求数据失败")
                }
                this.rightsList = res.data
                console.log(this.rightsList)
                // 获取三级节点的id
                this.getLeafKeys(role,this.defKeys)
                this.setRightDialogVisble = true

            },
            // 通过递归形式，获取角色所有三级权限的id，并保存到defKeys数组
            getLeafKeys(node,arr){
                if(!node.children){
                    return arr.push(node.id)
                }

                node.children.forEach(item =>
                    this.getLeafKeys(item,arr))
            },
            // 监听分配权限对话框的关闭
            setRightDialogClosd(){
                this.defKeys = []
            },
            // 点击为角色分配权限
            async allotRights(){
                const keys = [
                    ...this.$refs.treeRef.getCheckedKeys(),
                    ...this.$refs.treeRef.getHalfCheckedKeys()]
                
                const idStr = keys.join(',')
                const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
                if(res.meta.status !== 200){
                    return this.$message.err("分配权限失败")
                }

                this.$message.success("分配权限成功")
                this.getRolesList()
                this.setRightDialogVisble = false

            }
        },

    }
</script>

<style scoped>
    .el-tag{
        margin: 10px;
    }
    .bdtop{
        border-top: 1px solid #eee;
    }
    .bdbottom{
        border-bottom: 1px solid #eee;
    }
    .vcenter{
        display: flex;
        align-items: center;
    }
</style>