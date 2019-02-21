<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.keyword" placeholder="关键字"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getProducts">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">新增</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleViewProperties">显示属性</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleSkuProperties">sku属性</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleOnSale">上架</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleOffSale">下架</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表数据-->
        <el-table
                ref="singleTable"
                :data="products"
                highlight-current-row
                @selection-change="handleSelectionChange"
                style="width: 100%">
            <el-table-column type="selection" width="50"></el-table-column>
            <el-table-column
                    type="index"
                    width="50">
            </el-table-column>
            <el-table-column
                    property="name"
                    label="商品名称"
                    min-width="120">
            </el-table-column>
            <el-table-column
                    property="subName"
                    label="副名称"
                    min-width="120">
            </el-table-column>
            <el-table-column
                    property="productType.name"
                    label="类型"
                    min-width="120">
            </el-table-column>
            <el-table-column
                    property="brand.name"
                    label="品牌"
                    min-width="120">
            </el-table-column>
            <el-table-column
                    property="onSaleTime"
                    label="上架时间"
                    min-width="120">
            </el-table-column>
            <el-table-column
                    property="offSaleTime"
                    label="下架时间"
                    min-width="120">
            </el-table-column>
            <el-table-column
                    property="state"
                    label="状态"
                    min-width="120">
            </el-table-column>
            <el-table-column label="操作" width="150">
                <template scope="scope">
                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <!--分页条-->
        <el-pagination
                @current-change="handlePageChange"
                :page-size="rows"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total"
                background
                style="float:right;"
        >
        </el-pagination>
        <el-dialog :title="title" :visible.sync="dialogTableVisible" center>
            <el-form :model="form" :rules="rules" ref="form" label-width="100px" class="demo-ruleForm">
                <el-form-item label="名称" prop="name">
                    <el-input v-model="form.name"></el-input>
                </el-form-item>
                <el-form-item label="副名称">
                    <el-input v-model="form.subName"></el-input>
                </el-form-item>
                <el-form-item label="类型">
                    <!--<el-input v-model="form.productTypeId"></el-input>-->
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
                <el-form-item label="品牌">
                    <!--<el-input v-model="form.brandId"></el-input>-->
                    <el-select v-model="form.brandId" placeholder="请选择">
                        <el-option
                                v-for="item in brands"
                                :key="item.id"
                                :label="item.name"
                                :value="item.id">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="medias">
                    <!--<el-input v-model="form.medias"></el-input>-->
                    <el-upload
                            class="upload-demo"
                            action="http://127.0.0.1:8023/services/common/fastdfs"
                            :on-preview="handlePreview"
                            :on-remove="handleRemove"
                            :on-success="handleSuccess"
                            :file-list="mediasList"
                            list-type="picture">
                        <el-button size="small" type="primary">点击上传</el-button>
                        <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                    </el-upload>
                </el-form-item>
                <el-form-item label="描述">
                    <el-input v-model="form.productExt.description"></el-input>
                </el-form-item>
                <el-form-item label="详情">
                    <el-input v-model="form.productExt.richContent"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="dialogTableVisible = false">取消</el-button>
                <el-button type="primary" @click.native="editSubmit">提交</el-button>
            </div>
        </el-dialog>
        <el-dialog :title="title" :visible.sync="dialogViewPropertiesVisible" center>
            <el-form :model="ViewForm" ref="ViewForm" label-width="100px" class="demo-ruleForm">
                <el-form-item v-for="specification in specifications" :label="specification.name">
                    <el-input v-model="specification.value"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="dialogViewPropertiesVisible = false">取消</el-button>
                <el-button type="primary" @click.native="SubmitViewProperties">提交</el-button>
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
                selectedRows:[],
                dialogTableVisible:false,
                title:'',
                mediasList:[],
                selectMediasList:[],
                brands:[],
                productTypes:[],
                selectTypes:[],
                props: {
                    value: "id",
                    label: "name",
                    children: "children"
                },
                editLoading:false,
                products: [],
                rows: 10,
                total: 40,
                page: 1,
                form: {
                    id: '',
                    name: '',
                    subName: '',
                    productTypeId: '',
                    brandId: '',
                    medias: '',
                    productExt: {
                        description: '',
                        richContent: ''
                    }
                },
                rules: {
                    name: [
                        {required: true, message: '请输入活动名称', trigger: 'blur'},
                    ]
                },
                dialogViewPropertiesVisible:false,
                ViewForm:{
                    name:''
                },
                specifications:[]
            }
        },
        methods: {
            handlePreview(file) {
            },
            handleRemove(file, fileList) {
                this.$http.delete("/common/fastdfs?fileName="+ file.response.retObj)
                    .then(res => {
                        let data = res.data;
                        if (data.success) {
                            console.debug(this.selectMediasList)
                            let i = this.selectMediasList.indexOf(file.response.retObj);
                            this.selectMediasList.splice(i, 1);
                            console.debug(this.selectMediasList)
                            this.$message({
                                message: data.message,
                                type: 'success'
                            });
                        } else {
                            this.$message.error(data.message);
                        }
                    })
            },
            handleSuccess(response, file, fileList) {
                this.selectMediasList.push(response.retObj);
                console.debug(this.selectMediasList)
                this.$message({
                    message: response.message,
                    type: 'success'
                });
                // let path ={name:file.name,url:"http://192.168.43.142"+response.retObj};
            },
            handleChange(value) {
                this.selectTypes = value;
            },
            editSubmit(){
                this.$refs.form.validate((valid) => {
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            this.editLoading = true;
                            this.form.productTypeId = this.selectTypes[this.selectTypes.length - 1];
                            this.form.medias = this.selectMediasList.toString();
                            //NProgress.start();
                            let para = Object.assign({}, this.form);
                            this.$http.post("/product/product/", para).then((res) => {
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
                                this.$refs['form'].resetFields();
                                this.dialogTableVisible = false;
                                this.getBrands();
                            });
                        });
                    }
                });
            },
            //获取商品数据
            getProducts() {
                let para = {
                    page: this.page,
                    rows: this.rows,
                    keyword: this.filters.keyword
                }
                this.$http.post("/product/product/json", para)
                    .then(res => {
                        this.products = res.data.rows;
                        this.total = res.data.total;
                    })
            },
            //显示新增
            handleAdd() {
                this.title = "新增商品";
                this.mediasList = [];
                this.form= {
                    id: '',
                        name: '',
                        subName: '',
                        productTypeId: '',
                        brandId: '',
                        medias: '',
                        productExt: {
                        description: '',
                            richContent: ''
                    }
                }
                this.dialogTableVisible = true;
            },
            handleCurrentChange(val) {
                this.currentRow = val;
            },
            handlePageChange(val) {
                this.page = val;
                this.getProducts();
            },
            //显示编辑
            handleEdit(index, row) {
                this.title = "编辑商品";
                this.selectTypes = [];
                let str = row.productType.path;
                str = str.substring(1, str.length - 1);
                let arr = str.split(".");
                for (let i = 0; i < arr.length; i++) {
                    this.selectTypes.push(Number.parseInt(arr[i]));
                }
                this.mediasList = [];
                if(row.medias){
                    let arr = row.medias.split(",");
                    for(let i=0;i<arr.length;i++){
                        this.mediasList.push({name:"temp"+i,url:"http://192.168.43.142"+arr[i]})
                    }
                }
                this.dialogTableVisible = true;
                this.form = Object.assign({}, row);
            },
            handleDel(index, row) {
                this.$confirm('确认删除该记录吗?', '提示', {
                    type: 'warning'
                }).then(() => {
                    this.listLoading = true;
                    this.$http.delete("/product/product/" + row.id).then((res) => {
                        this.listLoading = false;
                        let data = res.data;
                        if (!data.success) {
                            this.$message.error(data.message);
                        }
                        this.$message({
                            message: res.data.message,
                            type: 'success'
                        });
                        this.getProducts();
                    });
                    this.editForm.logo = row.logo;
                    this.handleRemove();
                }).catch(() => {
                });
            },
            resetForm(formName) {
                this.$refs[formName].resetFields();
            },
            handleViewProperties(){
                //获取选中行
                if(this.selectedRows.length!=1){
                    this.$message({
                        message:'请选中一行进行操作！',
                        type:'warning'
                    })
                    return;
                }
                this.dialogViewPropertiesVisible = true;
                //查询类型属性给specifications赋值
                this.$http.get("/product/specification/productType/"+this.selectedRows[0].productTypeId)
                    .then((res)=>{
                        console.debug(res.data);
                        this.specifications = res.data
                    })
            },
            handleSkuProperties(){

            },
            handleOnSale(){

            },
            handleOffSale(){

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
            deleteChildren(data) {
                for (let i = 0; i < data.length; i++) {
                    if (!data[i].children.length) {
                        delete data[i].children;
                    } else {
                        this.deleteChildren(data[i].children)
                    }
                }
            },
            getBrands() {
                this.$http.get("/product/brand/list")
                    .then(res => {
                        this.brands = res.data;
                    })
            },
            SubmitViewProperties(){

            },
            handleSelectionChange(selection){
                this.selectedRows = selection;
            }
        },
        mounted() {
            this.getProducts();
            this.getProductType();
            this.getBrands();
        }
    }

</script>

<style scoped>

</style>