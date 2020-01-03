<template>
    <div>
        <el-button size="small" type="primary" @click="toAddHandler">添加</el-button>
        <el-button size="small" type="danger">批量删除</el-button>

        <el-table :data="employees">
            <el-table-column  prop="id" fixed="left" label="编号"></el-table-column>
            <el-table-column  prop="username" label="用户名"></el-table-column>
            <el-table-column  prop="realname" fixed="left" label="姓名"></el-table-column>
            <el-table-column  prop="gender" label="性别"></el-table-column>
            <el-table-column  prop="telephone" width="120" label="手机号"></el-table-column>
            <el-table-column  prop="idCard" width="200" label="身份证号"></el-table-column>
            <el-table-column  prop="bankCard" width="200" label="银行卡号"></el-table-column>
            <el-table-column  fixed="right" label="操作">
                <template v-slot="slot">
                    <a href="" @click.prevent="toUpdataHandler(slot.row)">修改</a>
                    <a href="" @click.prevent="toDeleteHandler(slot.row.id)">删除</a>
                </template>
            </el-table-column>

        </el-table>
    <!-- 模态框 -->
        <el-dialog
            title="录入员工信息"
            :visible.sync="visible"
            width="60%">
            <el-form ref="form" :model="form" label-width="80px">
                <el-form-item label="用户名">
                    <el-input v-model="form.username"></el-input>
                </el-form-item>
                <el-form-item label="密码">
                    <el-input type="password" v-model="form.password"></el-input>
                </el-form-item>
                 <el-form-item label="姓名">
                    <el-input v-model="form.realname"></el-input>
                </el-form-item>
                <el-form-item label="性别">
                    <el-radio-group v-model="form.grnder">
                        <el-radio label="男">男</el-radio>
                        <el-radio label="女">女</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="手机号">
                    <el-input v-model="form.telephone"></el-input>
                </el-form-item>
                <el-form-item label="身份证号">
                    <el-input v-model="form.idCard"></el-input>
                </el-form-item>
                <el-form-item label="银行卡号">
                    <el-input v-model="form.bankCard"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
            <el-button @click="closeModelHandler" size="small">取 消</el-button>
            <el-button type="primary" @click="submitHandler" size="small">确 定</el-button>
            </span>
        </el-dialog>

    <!-- 模态框 -->
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
            employees:[],
            form:{
                type:"waiter"
            }
        }
    },
    methods:{
        loadData(){
            let url = "http://localhost:6677/waiter/findAll"
            request.get(url).then((response)=>{
            // 将查询结果设置到customers中，this指向外部函数的this
            //箭头函数的this指向外部函数的this
                this.employees = response.data;
            })
        },
        toDeleteHandler(id) {
            this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {
                //alert(id);
                let url="http://localhost:6677/waiter/deleteById?id="+id;
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
                 types:"employee"
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
            let url = "http://localhost:6677/waiter/saveOrUpdate";
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