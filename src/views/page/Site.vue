<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">新增课程</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="site" highlight-current-row  @selection-change="selectChange"  style="width: 100%;">

            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column type="index" width="60">
            </el-table-column>
            <el-table-column prop="name" label="站点名称" width="150" sortable>
            </el-table-column>
            <el-table-column prop="url" label="站点地址" width="150" sortable>
            </el-table-column>
            <el-table-column prop="description" label="描述" width="300" sortable>
            </el-table-column>
            <el-table-column prop="sn" label="标识" width="120" sortable>
            </el-table-column>

            <el-table-column label="操作" width="150">
                <template scope="scope">
                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>

        <!--工具条-->
        <el-col :span="24" class="toolbar">
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="pageSize" :total="total" style="float:right;">
            </el-pagination>
        </el-col>
<!------------------------------------------------------->

        <!--新增界面-->
        <el-dialog title="新增课程" :visible.sync="addSiteVisible" :close-on-click-modal="false" style="width: 900px">
            <el-form :model="addSite" label-width="80px" :rules="addSiteRules" ref="addSite">
                <el-form-item label="名称" prop="name" style="width: 350px">
                    <el-input v-model="addSite.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="地址" prop="url" style="width: 350px">
                    <el-input v-model="addSite.url" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="描述" prop="description" style="width: 350px">
                    <el-input v-model="addSite.description" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="标识" prop="sn" style="width: 350px">
                    <el-input v-model="addSite.sn" auto-complete="off"></el-input>
                </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addSiteVisible = false">取消</el-button>
                <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>
<!----------------------------------------------------------------------------------->
        <!--编辑界面-->
        <el-dialog title="新增课程" :visible.sync="editFormVisible" :close-on-click-modal="false" style="width: 900px">
            <el-form :model="editForm" label-width="80px" :rules="addSiteRules" ref="editForm">
                <el-form-item label="名称" prop="name" style="width: 350px">
                    <el-input v-model="editForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="地址" prop="url" style="width: 350px">
                    <el-input v-model="editForm.url" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="描述" prop="description" style="width: 350px">
                    <el-input v-model="editForm.description" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="标识" prop="sn" style="width: 350px">
                    <el-input v-model="editForm.sn" auto-complete="off"></el-input>
                </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="editFormVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
    export default {
        name: "course",
        data(){
            return {
                options:[
                    {
                        value:0,
                        label:"未上架"
                    },
                    {
                        value:1,
                        label:"上架"
                    }
                ],
                filters:{
                    keyword:"",
                    status:null
                },
                site:[],
                sels:[],//多选框选中的行
                pageNum:1,
                pageSize:10,
                total:0,
                addSiteVisible:false,
                editFormVisible:false,
                courseTypes:[],
                addLoading: false,
                addSiteRules:{},
                addSite:{
                    name:"",
                    url:"",
                    description:"",
                    sn:""
                },
                editForm:{
                    name:"",
                    url:"",
                    description:"",
                    sn:""
                }
            }
        },
        methods:{
            //获取课程
            getSite(){
                let param = {};
                param.page = this.pageNum;
                param.rows = this.pageSize;
                this.$http.post("/page/site/page",param)
                    .then(res=>{
                        let {total,rows} = res.data;
                        this.total = total;
                        this.site = rows;
                    })
            },
            selectChange(selection){
                this.sels = selection;
            },
            handleAdd(){
                this.addSiteVisible = true;
                //清空表单
                this.addSite=[];
            },
            handleEdit(index, row) {
                this.editForm = Object.assign({}, row);
               this.editFormVisible = true;
            },
            handleDel(index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.$http.delete("/page/site/"+row.id).then(res=>{
                        let {success,msg} = res.data;
                        if(success){
                            this.$message({
                                message: msg,
                                type: 'success'
                            });
                            this.getSite();
                        }else{
                            this.$message({
                                message: msg,
                                type: 'error'
                            });
                        }
                    })

                }).catch(() => {

                });
            },
            handleCurrentChange(val){
                this.pageNum = val;
                this.getSite();
            },
            addSubmit(){
                let param={};
                param.name = this.addSite.name;
                param.url = this.addSite.url
                param.description = this.addSite.description
                param.sn = this.addSite.sn;
                this.$confirm('确认提交吗？', '提示', {}).then(() => {
                    this.$http.post("/page/site/save",param)
                        .then(res=>{
                            let {success,message} = res.data;
                            if (success){
                                this.$message({
                                    message:message ,
                                    type: 'success'
                                });
                                this.getSite();
                            }else{
                                this.$message({
                                    message: message,
                                    type: 'error'
                                });
                            }
                        })
                });
                this.addSiteVisible = false;

            },
            editSubmit(){
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            let para = Object.assign({}, this.editForm);
                            console.log(para)
                            this.$http.post("/page/site/save/",para).then(res=>{
                                let {success,message} = res.data;
                                if(success){
                                    this.$message({
                                        message: message,
                                        type: 'success'
                                    });
                                    this.editFormVisible = false;
                                    this.getSite();
                                }else{
                                    this.$message({
                                        message: message,
                                        type: 'error'
                                    });
                                }

                            })
                        });
                    }
                });
            }
        },
        mounted(){
            this.getSite();
        }
    }
</script>

<style scoped>

</style>