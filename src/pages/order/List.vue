<template>
    <div>
        <!--按钮-->
        <el-button size="small" type="success" @click="toAddHandler">添加</el-button>
        <el-button size="small" type="danger">删除</el-button>
        <!--/按钮-->
        <!--表格-->
        <el-table :data="Orders.list"> 
            <el-table-column prop="id" label="编号"></el-table-column>
            <el-table-column prop="orderTime" label="下单时间"></el-table-column>
            <el-table-column prop="total" label="总价"></el-table-column>
            <el-table-column prop="status" label="状态"></el-table-column>
            <el-table-column prop="customerId" label="顾客编号"></el-table-column>
            <el-table-column prop="waiterrId" label="员工编号"></el-table-column>
            <el-table-column prop="addressId" label="地址编号"></el-table-column>

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
            :total="Orders.total"
            @current-change="pageChangeHandler">
        </el-pagination>
        <!--/分页-->
        <!--模态框-->
        <el-dialog
            title="录入顾客信息"
            :visible.sync="visible"
            width="60%">
            <el-form ref="form" :model="form" label-width="80px">
                <el-form-item label="用户名">
                    <el-input v-model="form.username"></el-input>
                </el-form-item>
                <el-form-item label="密码">
                    <el-input type="password" v-model="form.password"></el-input>
                </el-form-item>
                 <el-form-item label="真实姓名">
                    <el-input v-model="form.realname"></el-input>
                </el-form-item>
                 <el-form-item label="手机号">
                    <el-input v-model="form.telephone"></el-input>
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
import request from '@/utils/request'
import querystring, { stringify } from 'querystring'
export default {
    //用于存放要向网页中存放的方法
    methods:{
        loadData(){
            let url = "http://localhost:6677/order/queryPage"
            request({
                url,
                method:"POST",
                headers:{
                "Content-Type":"application/x-www-form-urlencoded"
              },
                data:querystring.stringify(this.params)
            }).then((response)=>{
              this.Orders=response.data;
            })
        },
        toAddHandler(){
            this.visible=true;
             this.form={
                 types:"costomer"
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
        toDeleteHandler(id) {
            this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {
                //alert(id);
                let url="http://localhost:6677/order/deleteById?id="+id;
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
            //this.form 对象 ---字符串--> 后台 {type:'Order',age:12}
            // json字符串 '{"type":"Order","age":12}'
            // request.post(url,this.form)
            // 查询字符串 type=Order&age=12
            // 通过request与后台进行交互，并且要携带参数
            let url = "http://localhost:6677/order/saveOrUpdate";
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
        //当分页当前页改变是执行
        pageChangeHandler(page){
          //params中当前页改为插件中当前页
          this.params.page=page-1;
          //加载
          this.loadData();
        }
    },
    //用于存放要向网页中存放的数据
    data(){
        return {
            visible:false,
            Orders:[],
            form:{
                type:"Order"
            },
            params:{
              page:0,
              pageSize:10
            }
        }
    },
    created(){
        this.loadData();
    }
}
</script>

<style scoped>

</style>