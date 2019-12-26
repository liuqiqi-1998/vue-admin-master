<template id="courseType">
    <el-row style="height: 100%;border: 1px solid #DCDFE6;margin-top: 10px">
        <el-col :span="6" style="border-right: 1px solid #DCDFE6; min-height:500px;">
            <div class="grid-content bg-purple">
                <el-tree style="border: none"  :expand-on-click-node="false" :data="courseTypes" :props="defaultProps" accordion  @node-click="handleNodeClick">
                </el-tree>
            </div>
        </el-col>
        <el-col :span="17" style="margin-left: 10px">
            <div class="grid-content bg-purple">
                <template>
                    <el-col  class="toolbar" style="padding-bottom: 0px;">
                        <el-form :inline="true" :model="filters" ref="docform" class="docform">
                            <el-form-item>
                                <el-button type="primary" size="medium" icon="el-icon-plus" v-on:click="getdocs">添加</el-button>
                            </el-form-item>
                        </el-form>
                    </el-col>
                    <el-table :data="courses" style="width: 100%">
                        <el-table-column type="selection" width="50">
                        </el-table-column>
                        <el-table-column type="index" width="60" label="#">
                        </el-table-column>
                        <el-table-column label="课程编号" prop="id" width="120" sortable>
                        </el-table-column>
                        <el-table-column label="课程名称" prop="name" width="120" sortable>
                        </el-table-column>
                        <el-table-column label="创建时间" prop="createTime" :formatter="dateFormat" width="120">
                        </el-table-column>
                        <el-table-column label="所属标题" prop="pid" width="120">
                        </el-table-column>
                        <el-table-column label="描述"  prop="description" width="120">
                        </el-table-column>
                        <el-table-column label="操作" min-width="150">
                            <template scope="scope">
                                <el-button type="primary" size="small"  round @click="handleDel(scope.$index, scope.row)" >编辑</el-button>
                                <el-button type="danger"  size="small"  round  @click="handleDel(scope.$index, scope.row)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                    <!--工具条-->
                    <el-col :span="24" class="toolbar">
                        <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="pageSize" :total="total" style="float:right;">
                        </el-pagination>
                    </el-col>
                </template>
            </div>

        </el-col>
    </el-row>
</template>
<style>
    .el-row {
        margin-bottom: 20px;
        height: 100%;
    }
    :last-child {
        margin-bottom: 0;
    }
    #courseType el-col {
        border: 1px solid red;
        border-radius: 4px;
    }
    .grid-content {
        border-radius: 4px;
        min-height: 36px;
    }
</style>

<script>
    export default {
        data() {
            return {
                courseTypes:[],
                defaultProps: {
                    children: 'children',
                    label: 'name'
                },
                courses:[],
            //分页
                total: 0,
                page: 1,
                pageSize:10,
                keyword:null
            }
        },
        methods:{
            getTreeData(){
                // 发送一个异步请求: get请求 /product/courseType/treeData
                this.$http.get("/course/courseType/treeData").then(res=>{
                    console.log(this);
                    this.courseTypes = res.data;
                });
            },
            getCourses(){
                let para = {
                    page: this.page,
                    rows: this.pageSize,
                    keyword:this.keyword
                };
                //加载数据
                this.$http.post("/course/courseType/page",para).then(res =>{
                    let {total,rows} = res.data;
                    this.total = total;
                    this.courses = rows;
                });
            },
            handleCurrentChange(val) {
                this.page = val;
                this.getCourses();
            },
            handleNodeClick(v,e){
               //发送异步请求，获取所有子节点
                //console.log(v.id); 获取当前id
                //console.log(e.parent.data.id);//获取父级标题id
                this.$http.get("/course/courseType/findChildById/"+v.id).then(
                    res =>{
                        this.courses = res.data;
                });
                this.getCourses();
            },
            dateFormat(row,column){
                var t=new Date(row.createTime);//row 表示一行数据, updateTime 表示要格式化的字段名称
                return t.getFullYear()+"-"+(t.getMonth()+1)+"-"+t.getDate();//+" "+t.getHours()+":"+t.getMinutes()+":"+t.getSeconds()+"."+t.getMilliseconds()
            },
            //删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.$http.delete("/course/courseType/"+row.id).then(res=>{
                        let {success,message} = res.data;
                        if(success){
                            this.$message({
                                message: message,
                                type: 'success'
                            });
                            this.getdocs();
                        }else{
                            this.$message({
                                message: message,
                                type: 'error'
                            });
                        }
                    })
                }).catch(() => {

                });
            },
        },
        mounted(){
            //对courseTypes数据赋值
           this.getTreeData();
           this.getCourses();
        }
    };
</script>