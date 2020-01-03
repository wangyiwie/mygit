<template>
    <div>
        <el-button size="small" type="success" @click="toAddHandler">添加</el-button>
        <el-button size="small" type="danger" >批量删除</el-button>
        <!--表格-->
        <el-table :data="products"> 
            <el-table-column prop="id" label="编号"></el-table-column>
            <el-table-column prop="name" label="产品名称"></el-table-column>
            <el-table-column prop="price" label="价格"></el-table-column>
            <el-table-column prop="description" label="描述"></el-table-column>
            <el-table-column prop="categoryId" label="所属产品"></el-table-column>
            <el-table-column label="操作">
                <template v-slot="slot">
                    <!--阻止默认跳转-->
                    <a href="" @click.prevent="toUpdataHandler(slot.row)">修改</a>
                    <a href="" @click.prevent="toDeleteHandler(slot.row.id)">删除</a>
                    <a href="">详情</a>
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
            title="录入产品信息"
            :visible.sync="visible"
            width="60%">
            {{form}}
            <el-form ref="form" :model="form" label-width="80px">
                <el-form-item label="产品名称">
                    <el-input v-model="form.name"></el-input>
                </el-form-item>
                <el-form-item label="产品价格">
                    <el-input v-model="form.price"></el-input>
                </el-form-item>
                 <el-form-item label="产品描述">
                    <el-input v-model="form.description"></el-input>
                </el-form-item>
                <el-form-item label="所属栏目">
                    <el-select v-model="form.parentId" placeholder="请选择">
                        <el-option
                        v-for="item in options"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-upload
                    class="upload-demo"
                    action="https://jsonplaceholder.typicode.com/posts/"
                    :on-preview="handlePreview"
                    :on-remove="handleRemove"
                    :file-list="fileList"
                    list-type="picture">
                    <el-button size="small" type="primary">点击上传</el-button>
                    <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                </el-upload>
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
import request from '@/utils/request'
import querystring, { stringify } from 'querystring'
export default {
    methods:{
        loadData(){
            let url = "http://localhost:6677/product/findAll"
            request.get(url).then((response)=>{
            // 将查询结果设置到customers中，this指向外部函数的this
                this.products = response.data;
            })
        },
        toAddHandler(){
            this.visible=true;
        },
        closeModelHandler(){
            this.visible=false;
        },
        toUpdataHandler(){
            this.visible=true;
        },
        toDeleteHandler(id) {
            this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {
                //alert(id);
                let url="http://localhost:6677/product/deleteById?id="+id;
                request.get(url).then((response)=>{
                    this.loadData();
                    this.$message({
                        type: 'success',
                        message: response.message
                    });  
                })        
            });
        },
        submitHandler(){
            //this.form 对象 ---字符串--> 后台 {type:'customer',age:12}
            // json字符串 '{"type":"customer","age":12}'
            // request.post(url,this.form)
            // 查询字符串 type=customer&age=12
            // 通过request与后台进行交互，并且要携带参数
            let url = "http://localhost:6677/product/saveOrUpdate";
            request({
                url,
                method:"POST",
                headers:{
                "Content-Type":"application/x-www-form-urlencoded"
                },
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
        },
        handleRemove(file, fileList) {
            console.log(file, fileList);
        },
        handlePreview(file) {
            console.log(file);
        }
    },
    data(){
        return {
            visible:false,
            products:[],
            fileList: [],
            form:{
                type:"product"
            },
            options: [],
            value: ''
            
        }
    },
    created(){
        this.loadData();
    }
}
</script>

<style scoped>

</style>