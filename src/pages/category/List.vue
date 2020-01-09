<template>
    <div>
        <el-button size="small" type="success" @click="toAddHandler">添加</el-button>
        <el-button size="small" type="danger" >批量删除</el-button>
         <!--表格-->
        <el-table :data="categories"> 
            <el-table-column prop="id" label="编号"></el-table-column>
            <el-table-column prop="name" label="栏目名称"></el-table-column>
            <el-table-column prop="num" label="栏目数量"></el-table-column>
            <el-table-column prop="parentId" label="父栏目"></el-table-column>
            <el-table-column label="操作">
                <template v-slot="slot">
                    <!--阻止默认跳转-->
                    <a href="" @click.prevent="toUpdataHandler(slot.row)">修改</a>
                    <a href="" @click.prevent="toDeleteHandler(slot.row.id)">删除</a>
                </template>
            </el-table-column>
        </el-table>
        <!--/表格-->
        <!--分页-->
        <el-pagination
            layout="prev, pager, next"
            :total="50">
        </el-pagination>
        <!--/分页-->
 <!--模态框-->
        <el-dialog
            title="录入栏目信息"
            :visible.sync="visible"
            width="60%">
            {{form}}
            <el-form ref="form" :model="form" label-width="80px">
                <el-form-item label="栏目名称">
                    <el-input v-model="form.name"></el-input>
                </el-form-item>
                <el-form-item label="栏目数量">
                    <el-input v-model="form.num"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
            <el-button @click="closeModelHandler" size="small">取 消</el-button>
            <el-button type="primary" @click="submitHandler" size="small">确 定</el-button>
            </span>
        </el-dialog>
        <!--/模态框-->
    </div>
</template>

<script>
import querystring, { stringify } from 'querystring'
import request from '@/utils/request'
import { types } from 'util'
export default {
    //用于存放要向网页中存放的数据
    data(){
        return{
            visible:false,
            categories:[],
            form:{
                type:"category"
                // 
            }
        }
    },
    methods:{
        loadData(){
            let url = "http://localhost:6677/category/findAll"
            request.get(url).then((response)=>{
            // 将查询结果设置到customers中，this指向外部函数的this
            //箭头函数的this指向外部函数的this
                this.categories = response.data;
            })
        },
        toDeleteHandler(id) {
            this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {
                //alert(id);
                let url="http://localhost:6677/category/deleteById?id="+id;
                request.get(url).then((response)=>{
                    this.loadData();
                    this.$message({
                        type: 'success',
                        message: response.message
                    });  
                })        
            });
        },
        toAddHandler(){
            this.visible=true;
             this.form={
                 types:"category"
            };
        },
        closeModelHandler(){
            
            this.visible=false;
        },
        toUpdataHandler(row){
             this.visible=true;
            //模态框表单显示当前行的详细
             this.form=row;
        },
        submitHandler(){
            let url = "http://localhost:6677/category/saveOrUpdate";
            //前端向后台发送请求，完成数据的保存
            request({
                url,
                method:"POST",
                //告诉后台即将发送字符串，查询字符串
                headers:{
                "Content-Type":"application/x-www-form-urlencoded"
                },
                //将this.form转换为查询字符串转发给后台
                data:querystring.stringify(this.form)
            }).then((response)=>{
                // 模态框关闭
                this.closeModelHandler();
                // 刷新
                this.loadData();
                // 提示消息
                this.$message({
                    type:"success",
                    message:response.message
                })
            })
        } 
    },
    created(){
        this.loadData();
    }
}
</script>
<style scoped>

</style>