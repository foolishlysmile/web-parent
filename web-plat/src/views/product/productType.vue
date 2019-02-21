<template>
    <el-container>
        <el-aside width="300px">
            <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
        </el-aside>
        <el-main>
            <el-form ref="form" :model="form" label-width="80px">
                <el-form-item label="类型名称">
                    <el-input v-model="form.name"></el-input>
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
                    <el-input v-model="form.sortIndex"></el-input>
                </el-form-item>
                <el-form-item label="上级类型">
                    <el-cascader
                            :options="data"
                            :props="defaultProps"
                            expand-trigger="hover"
                            :change-on-select="true"
                            v-model="selectTypes"
                            :show-all-levels="false"
                            @change="handleChange"
                    ></el-cascader>
                </el-form-item>
                <el-form-item label="描述">
                    <el-input v-model="form.description" type="textarea"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="onSubmit">保存</el-button>
                    <el-button @click="onCancel">取消</el-button>
                </el-form-item>
            </el-form>
        </el-main>
    </el-container>
</template>

<script>
    export default {
        data() {
            return {
                form: {
                    name: '',
                    logo: '',
                    sortIndex: '',
                    pid: '',
                    description: ''
                },
                limit:1,
                fileList2:[],
                selectTypes: [],
                data: [],
                defaultProps: {
                    children: 'children',
                    label: 'name',
                    value: 'id'
                }
            };
        },
        methods: {
            handleNodeClick(data) {
                this.form = Object.assign({}, data);
                let str = data.path;
                str = str.substring(1, str.length-2);
                str = str.substring(0, str.lastIndexOf("."));
                let arr = str.split(".");
                for (let i = 0; i < arr.length; i++) {
                    this.selectTypes.push(Number.parseInt(arr[i]));
                }
            },
            onSubmit() {
                console.log('submit!');
            },
            getTreeData() {
                this.$http.post("/product/productType/treeData")
                    .then(
                        res => {
                            this.data = res.data;
                        }
                    )
            },
            handleChange(value) {
                this.selectTypes = value;
            },
            onCancel(){
                this.form={
                    name: '',
                    logo: '',
                    sortIndex: '',
                    pid: '',
                    description: ''
                }
            },
            handleSuccess(response, file, fileList) {
                this.form.logo = response.retObj;
                this.$message({
                    message: response.message,
                    type: 'success'
                });
            },
            handleRemove(file, fileList) {
                this.$http.delete("/common/fastdfs?fileName="+ this.form.logo)
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
            }
        },
        mounted() {
            this.getTreeData();
        }
    }
</script>

<style scoped>
    .el-aside {
        color: #333;
        text-align: center;
        border-right: 1px solid #20a0ff;
    }

    .el-main {
        background-color: #E9EEF3;
        color: #333;
        text-align: left;
        line-height: 160px;
    }

    .el-container {
        border: 1px solid #20a0ff;
        margin-bottom: 40px;
        position: absolute;
        top: 50px;
        left: 245px;
        right: 30px;
        bottom: 50px;
    }
</style>