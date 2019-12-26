<template id="courseType">
    <el-row style="height: 100%;border: 1px solid #DCDFE6;margin-top: 10px">
        <el-col :span="6" style="border-right: 1px solid #DCDFE6; min-height:500px;">
            <div class="grid-content bg-purple">
                <el-tree style="border: none" :data="courseTypes" :props="defaultProps" accordion  @node-click="handleNodeClick">
                </el-tree>
            </div>
        </el-col>
        <el-col :span="17" style="margin-left: 10px;padding-top: 10px">
            <div class="grid-content bg-purple">
                <template>
                    <el-table
                            :data="tableData"
                            style="width: 100%">
                        <el-table-column
                                label="日期"
                                width="180">
                            <template slot-scope="scope">
                                <i class="el-icon-time"></i>
                                <span style="margin-left: 10px">{{ scope.row.date }}</span>
                            </template>
                        </el-table-column>
                        <el-table-column
                                label="姓名"
                                width="180">
                            <template slot-scope="scope">
                                <el-popover trigger="hover" placement="top">
                                    <p>姓名: {{ scope.row.name }}</p>
                                    <p>住址: {{ scope.row.address }}</p>
                                    <div slot="reference" class="name-wrapper">
                                        <el-tag size="medium">{{ scope.row.name }}</el-tag>
                                    </div>
                                </el-popover>
                            </template>
                        </el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button
                                        size="mini"
                                        @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                                <el-button
                                        size="mini"
                                        type="danger"
                                        @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
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
                }
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
            handleNodeClick(){

            }
        },
        mounted(){
            //对courseTypes数据赋值
           this.getTreeData();
        }
    };
</script>