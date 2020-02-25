<template>
	<div>
		<!-- 面包屑导航区域 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>商品管理</el-breadcrumb-item>
			<el-breadcrumb-item>商品分类</el-breadcrumb-item>
		</el-breadcrumb>

		<!-- 卡片视图区域 -->
		<el-card>
			<el-row>
				<el-col>
					<el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
				</el-col>
			</el-row>

			<!-- 表格 -->
			<tree-table class="treeTable" :data="catelist" :columns="columns" :selection-type="false" :expand-type="false"
			 show-index index-text="" border>
				<!-- 是否有效 -->
				<template slot="isok" slot-scope="scope">
					<i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen"></i>
					<i class="el-icon-error" v-else style="color: red;"></i>
				</template>
				<!-- 排序 -->
				<template slot="order" slot-scope="scope">
					<el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
					<el-tag size="mini" type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
					<el-tag size="mini" type="warning" v-else>三级</el-tag>
				</template>
				<!-- 操作 -->
				<template slot="option" slot-scope="scope">
					<el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.cat_id)">编辑</el-button>
					<el-button size="mini" type="danger" icon="el-icon-delete" @click="removeCateById(scope.row.cat_id)">删除</el-button>
				</template>
			</tree-table>

			<!-- 分页区域 -->
			<el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum"
			 :page-sizes="[3, 5, 10, 15]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper"
			 :total="total">
			</el-pagination>
		</el-card>

		<!-- 添加分类的对话框 -->
		<el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
			<!-- 添加分类的表单 -->
			<el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
				<el-form-item label="分类名称：" prop="cat_name">
					<el-input v-model="addCateForm.cat_name"></el-input>
				</el-form-item>
				<el-form-item label="父级分类：">
					<!-- options 用来指定数据源 -->
					<!-- props 用于指定配置对象 -->
					<el-cascader v-model="selectedKeys" :options="parentCateList" :props="cascaderProps" @change="parentCateChanged"
					 clearable change-on-select>

					</el-cascader>
				</el-form-item>
			</el-form>
			<!-- 底部按钮区域 -->
			<span slot="footer" class="dialog-footer">
				<el-button @click="addCateDialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="addCate">确 定</el-button>
			</span>
		</el-dialog>

		<!-- 修改分类的对话框 -->
		<el-dialog title="修改分类" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
			<!-- 修改分类的表单 -->
			<el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
			  <el-form-item label="分类名称：" prop="cat_name">
			    <el-input v-model="editForm.cat_name"></el-input>
			  </el-form-item>
			</el-form>
			<!-- 底部按钮区域 -->
			<span slot="footer" class="dialog-footer">
				<el-button @click="editDialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="editCateInfo">确 定</el-button>
			</span>
		</el-dialog>
	</div>
</template>

<script>
	export default {
		name: "Cate",
		data() {
			return {
				// 查询条件
				queryInfo: {
					type: 3,
					pagenum: 1,
					pagesize: 5
				},
				// 商品分类的数据列表，默认为空
				catelist: [],
				// 总数据条数
				total: 0,
				// 为table指定列的定义
				columns: [{
						label: '分类名称',
						prop: 'cat_name'
					},
					{
						label: '是否有效',
						// 表示将当前列定义为模板列
						type: 'template',
						// 表示当前这一列使用模板名称
						template: 'isok'
					},
					{
						label: '排序',
						// 表示将当前列定义为模板列
						type: 'template',
						// 表示当前这一列使用模板名称
						template: 'order'
					},
					{
						label: '操作',
						// 表示将当前列定义为模板列
						type: 'template',
						// 表示当前这一列使用模板名称
						template: 'option'
					}
				],
				// 控制添加分类对话框的显示与隐藏
				addCateDialogVisible: false,
				// 添加分类的表单数据对象
				addCateForm: {
					// 将要添加的分类的名称
					cat_name: '',
					// 父级分类的Id
					cat_pid: 0,
					// 分类的等级，默认要添加的是一级分类
					cat_level: 0
				},
				// 添加分类表单的验证规则对象
				addCateFormRules: {
					cat_name: [{
						required: true,
						message: '请输入分类名称',
						trigger: 'blur'
					}, ]
				},
				// 父级分类的列表
				parentCateList: [],
				// 指定级联选择器的配置对象
				cascaderProps: {
					expandTrigger: 'hover',
					value: 'cat_id',
					label: 'cat_name',
					children: 'children'
				},
				// 选中的父级分类的Id数组
				selectedKeys: [],
				// 控制修改分类对话框的显示与隐藏
				editDialogVisible: false,
				// 查询到的分类数据对象
				editForm: {},
				// 修改表单的验证规则对象
				editFormRules: {
					cat_name: [
						{ required: true, message: '请输入分类名称', trigger: 'blur' }
					]
				}
			}
		},
		created() {
			this.getCateList()
		},
		methods: {
			// 获取商品分类数据
			async getCateList() {
				const {
					data: res
				} = await this.$http.get("categories", {
					params: this.queryInfo
				})

				if (res.meta.status !== 200) {
					return this.$message.error("获取商品分类数据失败")
				}

				// 把数据列表，赋值给 catelist
				this.catelist = res.data.result
				// 为总数据条数赋值
				this.total = res.data.total
				console.log(this.catelist)
			},
			// 监听 pagesize 改变
			handleSizeChange(newSize) {
				this.queryInfo.pagesize = newSize
				this.getCateList()
			},
			// 监听 pagenum 改变
			handleCurrentChange(newPage) {
				this.queryInfo.pagenum = newPage
				this.getCateList()
			},
			// 点击按钮，展示添加分类的对话框
			showAddCateDialog() {
				// 先获取父级分类的数据列表
				this.getParentCateList()
				// 在展示出对话框
				this.addCateDialogVisible = true
			},
			// 获取父级分类的数据列表
			async getParentCateList() {
				const {
					data: res
				} = await this.$http.get("categories", {
					params: {
						type: 2
					}
				})

				if (res.meta.status !== 200) {
					return this.$message.error("获取商品分类数据失败")
				}

				this.parentCateList = res.data
			},
			// 选中项发生变化触发这个函数
			parentCateChanged() {
				// 如果 selectedKeys 数组中 length 大于0，证明选中的父级分类
				// 反之，就说明没有选中如何父级分类
				if (this.selectedKeys.length > 0) {
					// 父级Id
					this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1];
					// 为当前分类的等级赋值
					this.addCateForm.cat_level = this.selectedKeys.length;
					return;
				} else {
					// 父级Id
					this.addCateForm.cat_pid = 0;
					// 为当前分类的等级赋值
					this.addCateForm.cat_level = 0;
				}
			},
			// 点击按钮，添加新的分类
			addCate() {
				// 对表单进行预校验
				this.$refs.addCateFormRef.validate(async valid => {
					if (!valid) return
					// 预校验通过发起添加分类的数据请求
					const {
						data: res
					} = await this.$http.post("categories", this.addCateForm)

					if (res.meta.status !== 201) {
						return this.$message.error("添加分类失败")
					}

					this.$message.success("添加分类成功")
					this.getCateList()
					this.addCateDialogVisible = false
				})
			},
			// 监听对话框的关闭事件，重置表单数据
			addCateDialogClosed() {
				this.$refs.addCateFormRef.resetFields()
				this.selectedKeys = []
				this.addCateForm.cat_level = 0
				this.addCateForm.cat_pid = 0
			},
			// 点击按钮，展示修改分类的对话框
			async showEditDialog(id) {
				const {data: res} = await this.$http.get('categories/' + id)
				
				if (res.meta.status !== 200) {
					return this.$message.error("查询分类数据失败！")
				}
				this.editForm = res.data
				this.editDialogVisible = true
			},
			// 点击按钮，修改分类数据
			editCateInfo() {
				// 对表单进行预校验
				this.$refs.editFormRef.validate(async valid => {
					if (!valid) return 
					// 预校验通过发起添加分类的数据请求
					const {data: res} = await this.$http.put("categories/" + this.editForm.cat_id, {
						cat_name: this.editForm.cat_name
					})
					
					if (res.meta.status !== 200) {
						return this.$message.error("修改分类数据失败！")
					}
					
					this.$message.success("修改分类数据成功！")
					this.getCateList()
					this.editDialogVisible = false
				})
			},
			// 监听修改分类对话框的显示与隐藏
			editDialogClosed() {
				this.$refs.editFormRef.resetFields()
			},
			// 根据id删除对应的分类数据
			async removeCateById(id) {
				// 弹框访问用户是否删除数据
				const confirmResult = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).catch(err => err)
				
				// 如果用户确定删除，则返回值为字符串 confirm
				// 如果用户取消删除，则返回值为字符串 cancel
				if (confirmResult !== 'confirm') {
					return this.$message.info("已取消删除")
				}
				
				const {data: res} = await this.$http.delete("categories/" + id)
				
				if (res.meta.status !== 200) {
					return this.$message.error("删除分类失败！")
				}
				
				this.$message.success("删除分类成功！")
				this.getCateList()
			}
		}
	}
</script>

<style lang="less" scoped>
	.treeTable {
		margin-top: 15px;
	}

	.el-cascader {
		width: 100%;
	}
</style>
