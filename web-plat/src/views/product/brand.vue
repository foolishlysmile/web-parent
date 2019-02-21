<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.keyword" placeholder="关键字"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getBrands">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">新增</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="brands" highlight-current-row v-loading="listLoading" @selection-change="selsChange"
                  style="width: 100%;">
            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column type="index" min-width="60">
            </el-table-column>
            <el-table-column prop="name" label="品牌名称" min-width="120" sortable>
            </el-table-column>
            <el-table-column prop="englishName" label="英文名称" min-width="100" sortable>
            </el-table-column>
            <el-table-column prop="productType.name" label="类型" min-width="120" sortable>
            </el-table-column>
            <el-table-column prop="description" label="描述" min-width="100" sortable>
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
            <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="rows"
                           :total="total" style="float:right;">
            </el-pagination>
        </el-col>

        <!--编辑界面-->
        <el-dialog :title="title" v-model="dialogVisible" :close-on-click-modal="false" :visible.sync="dialogVisible">
            <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
                <el-form-item label="品牌名称" prop="name">
                    <el-input v-model="editForm.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="英文名称">
                    <el-input v-model="editForm.englishName" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="logo">
                    <el-upload
                            class="upload-demo"
                            action="http://127.0.0.1:8023/services/common/fastdfs"
                            :on-preview="handlePreview"
                            :on-remove="handleRemove"
                            :on-success="handleSuccess"
                            :file-list="fileList2"
                            :limit="limit"
                            list-type="picture">
                        <el-button size="small" type="primary">点击上传</el-button>
                        <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                    </el-upload>
                </el-form-item>
                <el-form-item label="排序号">
                    <el-input v-model="editForm.sortIndex" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="类型">
                    <el-cascader
                            :options="productTypes"
                            :props="props"
                            expand-trigger="hover"
                            :change-on-select="true"
                            v-model="selectTypes"
                            :show-all-levels="false"
                            @change="handleChange"
                    ></el-cascader>
                </el-form-item>
                <el-form-item label="描述">
                    <el-input v-model="editForm.description" type="textarea"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="dialogVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
            </div>
        </el-dialog>
    </section>
</template>

<script>
    export default {
        data() {
            return {
                filters: {
                    keyword: ''
                },
                fileList2: [],
                brands: [],
                limit:1,
                total: 0,
                page: 1,
                rows: 10,
                listLoading: false,
                sels: [],//列表选中列
                dialogVisible: false,//编辑界面是否显示
                editLoading: false,
                editFormRules: {
                    name: [
                        {required: true, message: '请输入姓名', trigger: 'blur'}
                    ]
                },
                title: '',
                props: {
                    value: "id",
                    label: "name",
                    children: "children"
                },
                selectTypes: [],
                productTypes: [],
                //编辑界面数据
                editForm: {
                    id: '',
                    logo: '',
                    name: '',
                    sortIndex: '',
                    englishName: '',
                    productTypeId: '',
                    description: ''
                }
            }
        },
        methods: {
            handleSuccess(response, file, fileList) {
                this.editForm.logo = response.retObj;
                this.$message({
                    message: response.message,
                    type: 'success'
                });
            },
            handleCurrentChange(val) {
                this.page = val;
                this.getBrands();
            },
            handleRemove(file, fileList) {
                this.$http.delete("/common/fastdfs?fileName="+ this.editForm.logo)
                    .then(res => {
                        let data = res.data;
                        if (data.success) {
                            this.$message({
                                message: data.message,
                                type: 'success'
                            });
                        } else {
                            this.$message.error(data.message);
                        }
                    })
            },
            handlePreview(file) {
            },
            //获取用户列表
            getBrands() {
                let para = {
                    page: this.page,
                    rows: this.rows,
                    keyword: this.filters.keyword
                };
                this.listLoading = true;
                this.$http.post("/product/brand/json", para)
                    .then(
                        res => {
                            this.total = res.data.total;
                            this.brands = res.data.rows;
                            this.listLoading = false;
                        }
                    )
            },
            //删除
            handleDel: function (index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    this.$http.delete("/product/brand/" + row.id).then((res) => {
                        this.listLoading = false;
                        let data = res.data;
                        if (!data.success) {
                            this.$message.error(data.message);
                        }
                        this.$message({
                            message: res.data.message,
                            type: 'success'
                        });
                        this.getBrands();
                    });
                    this.editForm.logo = row.logo;
                    this.handleRemove();
                }).catch(() => {
                });
            },
            //显示编辑界面
            handleEdit: function (index, row) {
                this.title = "编辑品牌";
                this.selectTypes = [];
                let str = row.productType.path;
                str = str.substring(1, str.length - 1);
                let arr = str.split(".");
                for (let i = 0; i < arr.length; i++) {
                    this.selectTypes.push(Number.parseInt(arr[i]));
                }
                this.fileList2=[{name:"temp",url:"http://192.168.43.142"+row.logo}];
                this.dialogVisible = true;
                this.editForm = Object.assign({}, row);
            },
            //显示新增界面
            handleAdd: function () {
                this.title = "新增品牌";
                this.dialogVisible = true;
                this.selectTypes = [],
                this.fileList2 = [],
                this.editForm = {
                    id: '',
                    logo: '',
                    name: '',
                    sortIndex: '',
                    englishName: '',
                    productTypeId: '',
                    description: ''
                };
            },
            //编辑
            editSubmit: function () {
                this.$refs.editForm.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.editLoading = true;
                            this.editForm.productTypeId = this.selectTypes[this.selectTypes.length - 1];
                            //NProgress.start();
                            let para = Object.assign({}, this.editForm);
                            this.$http.post("/product/brand/", para).then((res) => {
                                this.editLoading = false;
                                let data = res.data;
                                if (!data.success) {
                                    this.$message({
                                        message: res.data.message,
                                        type: 'error'
                                    });
                                }
                                //NProgress.done();
                                this.$message({
                                    message: res.data.message,
                                    type: 'success'
                                });
                                this.$refs['editForm'].resetFields();
                                this.dialogVisible = false;
                                this.getBrands();
                            });
                        });
                    }
                });
            },
            selsChange: function (sels) {
                this.sels = sels;
            },
            //批量删除
            batchRemove: function () {
                var ids = this.sels.map(item => item.id).toString();
                var logs = this.sels.map(item => item.logo);
                this.$confirm('确认删除选中记录吗？', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    this.$http.delete("/product/brand/batchDelete/" + ids).then((res) => {
                        this.listLoading = false;
                        //NProgress.done();
                        this.$message({
                            message: '删除成功',
                            type: 'success'
                        });
                        this.getBrands();
                    });
                    for(let i=0;i<logs.length;i++){
                        this.editForm.logo = logs[i];
                        this.handleRemove();
                    }
                }).catch(() => {

                });
            },
            getProductType() {
                this.$http.post("/product/productType/treeData")
                    .then(
                        res => {
                            let data = res.data;
                            this.deleteChildren(data);
                            this.productTypes = data;
                        }
                    )
            },
            handleChange(value) {
                this.selectTypes = value;
            },
            deleteChildren(data) {
                for (let i = 0; i < data.length; i++) {
                    if (!data[i].children.length) {
                        delete data[i].children;
                    } else {
                        this.deleteChildren(data[i].children)
                    }
                }
            }
        },
        mounted() {
            this.getBrands();
            this.getProductType();
        }
    }

</script>

<style scoped>

</style>