<template>
    <div>
        <!-- 按钮 -->
        <el-button type="success" size = "small" @click="toAddHandler">添加</el-button>
        <el-button size="small" type="danger">批量删除</el-button>
        <!-- 表格 -->
        <el-table :data="products">
            <el-table-column prop="id" label="编号"> </el-table-column>
            <el-table-column prop="name" label="产品名称"></el-table-column>
            <el-table-column prop="price" label="价格"></el-table-column>
            <el-table-column prop="description" label="描述"></el-table-column>
            <el-table-column prop="categoryId" label="所属分类"></el-table-column>
            <el-table-column prop="photo" width="550px" label="照片"></el-table-column>

            <el-table-column prop="操作" label="操作">
             <template v-slot="slot">
                <a href="" @click.prevent="toDeleteHandler(slot.row.id)">删除</a>
                <a href="" @click.prevent="toUpdateHandler(slot.row)">修改</a>
            </template>
            </el-table-column>
        </el-table>

        <!-- 分页开始 -->
       <el-pagination
          layout="prev, pager, next"
          :total="50">
       </el-pagination>
    <!-- 分页结束 -->

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
                    <el-form-item label="所属分类">
                         <el-select v-model="value" placeholder="请选择">
                            <el-option v-for="item in options" 
                            :key="item.id"
                            :label="item.name" 
                            :value="item.id">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="产品描述">
                        <el-input type="textarea" v-model="form.description"></el-input>
                    </el-form-item>                   
                    <el-upload class="upload-demo"
                        action="http://134.175.154.93:6677/file/upload"
                        :on-preview="handlePreview" :on-remove="handleRemove"
                        :file-list="fileList" 
                        :on-success="uploadsuccessHandler">
                    <el-button size="small" type="primary">点击上传</el-button>
                    <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                    </el-upload>
                </el-form>
                <span slot="footer" class="dialog-footer">
                    <el-button size="small" @click="closeModalHandler">取 消</el-button>
                    <el-button size="small" type="primary" @click="submitHandler">确 定</el-button>
                </span>
            </el-dialog>
            <!--/模态框-->
    </div>
</template>


<script>
import request from '@/utils/request'
import querystring from 'querystring'   
export default {
    //用于存放网页中需要调用的方法
    methods:{
        loadCategory(){
            let url = "http://localhost:6677/product/findAll"
            request.get(url).then((response)=>{
                // 将查询结果设置到products中，this指向外部函数的this
                this.options = response.data;
            })
        },
        loadData(){
            let url = "http://localhost:6677/product/findAll"
            request.get(url).then((response)=>{
                // 将查询结果设置到products中，this指向外部函数的this
                this.products = response.data;
            })
        },
        handleRemove(file, fileList) {
        console.log(file, fileList);
        },
        handlePreview(file) {
            console.log(file);
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
                this.closeModalHandler();
                // 刷新
                this.loadData();
                // 提示消息
                this.$message({
                    type:"success",
                    message:response.message
                })
            })
        },
        toDeleteHandler(id) {
            this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {
                let url = "http://localhost:6677/product/deleteById?id="+id;
                request.get(url).then((response)=>{
                  //1.刷新数据
                  this.loadData();
                  //2.提示结果
                this.$message({
                    type: 'success',
                    message:response.message
                  });  

                })
                        
            });
        },
        toAddHandler(){
             this.form = {
            type:"product"
            }
            this.visible=true; 
        },
        closeModalHandler(){
            this.visible=false;
        },
        toUpdateHandler(row){
            this.fileList = [];
            this.form = row;
            this.visible=true;
        },
        uploadsuccessHandler(response){
            let photo = "http://134.175.154.93:8888/group1/"+response.data.id
            //将图片地址设置到form中，便于一起提交后台
            this.form.photo = photo;
           //console.log(response)
        }   
    },
    //用于存放要向网页中存放的数据
     data(){
        return{
            visible:false,
            products:[],
            form:{},
            fileList:[],
            options: [],
            value: ''
        }
    },
    created(){
        //vue实例创建完毕
        this.loadData()
        //加载栏目信息 用于下拉菜单
        this.loadCategory()
        }
    }

</script>


<style scoped>

</style>
