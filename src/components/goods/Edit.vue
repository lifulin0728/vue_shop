<template>
	<div>
		<!-- 面包屑导航区域 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
			<el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
			<el-breadcrumb-item>商品管理</el-breadcrumb-item>
			<el-breadcrumb-item>修改商品</el-breadcrumb-item>
		</el-breadcrumb>
		
		<!-- 卡片视图区域 -->
		<el-card>
			<!-- 提示区域 -->
			<el-alert title="修改商品信息" type="info" center show-icon :closable="false"></el-alert>
			<!-- 步骤条区域 -->
			<el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
			  <el-step title="基本信息"></el-step>
			  <el-step title="商品参数"></el-step>
			  <el-step title="商品属性"></el-step>
				<el-step title="商品图片"></el-step>
				<el-step title="商品内容"></el-step>
				<el-step title="完成"></el-step>
			</el-steps>
			
			<!-- tab栏区域 -->
			<el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" label-position="top">
				<el-tabs v-model="activeIndex" :tab-position="'left'">
					<el-tab-pane label="基本信息" name="0">
						<el-form-item label="商品名称" prop="goods_name">
							<el-input v-model="editForm.goods_name"></el-input>
						</el-form-item>
						<el-form-item label="商品价格" prop="goods_price">
							<el-input v-model="editForm.goods_price" type="number"></el-input>
						</el-form-item>
						<el-form-item label="商品重量" prop="goods_weight">
							<el-input v-model="editForm.goods_weight" type="number"></el-input>
						</el-form-item>
						<el-form-item label="商品数量" prop="goods_number">
							<el-input v-model="editForm.goods_number" type="number"></el-input>
						</el-form-item>
						<el-form-item label="商品分类" prop="goods_cat">
							<el-input v-model="categoryName" disabled></el-input>
						</el-form-item>
					</el-tab-pane>
					<el-tab-pane label="商品参数" name="1">
						<!-- 渲染表单的Item项 -->
						<el-form-item v-for="item in manyTableData" :key="item.attr_id" :label="item.attr_name">
							<!-- 复选框组 -->
							<el-checkbox-group v-model="item.attr_vals">
								<el-checkbox v-for="(value, index) in item.attr_vals" :label="value" :key="index" border></el-checkbox>
							</el-checkbox-group>
						</el-form-item>
					</el-tab-pane>
					<el-tab-pane label="商品属性" name="2">
						<el-form-item v-for="item in onlyTableData" :key="item.attr_id" :label="item.attr_name">
							<el-input v-model="item.attr_vals"></el-input>
						</el-form-item>
					</el-tab-pane>
					<el-tab-pane label="商品图片" name="3">
						<!-- 图片上传缩略图 -->
						<!-- action 表示图片要上传到的后台API地址 -->
						<el-upload :action="uploadURL" :on-preview="handlePreview" :on-remove="handleRemove" list-type="picture" :headers="headerObj" :on-success="handleSuccess" :file-list="fileList">
						  <el-button size="small" type="primary">点击上传</el-button>
						</el-upload>
					</el-tab-pane>
					<el-tab-pane label="商品内容" name="4">
						<!-- 富文本编辑器组件 -->
						<quill-editor v-model="editForm.goods_introduce"></quill-editor>
						<!-- 修改商品的按钮 -->
						<el-button type="primary" class="btnEdit" @click="edit">修改商品</el-button>
					</el-tab-pane>
				</el-tabs>
			</el-form>
		</el-card>
		
		<!-- 图片预览对话框 -->
		<el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
		  <img :src="previewPath" alt="" class="previewImg">
		</el-dialog>
	</div>
</template>

<script>
	export default {
		name: "Edit",
		data() {
			return {
				// 获取要修改商品Id
				id: this.$route.params.id,
				// 查询到的商品信息对象
				editForm: {},
				// 控制步骤条的活跃状态
				activeIndex: '0',
				// 修改商品的验证规则对象
				editFormRules: {
					goods_name: [
						{ required: true, message: '请输入商品名称', trigger: 'blur' }
					],
					goods_price: [
						{ required: true, message: '请输入商品价格', trigger: 'blur' }
					],
					goods_weight: [
						{ required: true, message: '请输入商品重量', trigger: 'blur' }
					],
					goods_number: [
						{ required: true, message: '请输入商品数量', trigger: 'blur' }
					]
				},
				// 分类Id的数组
				goods_array: [],
				// 商品所属的分类名称
				categoryName: '',
				// 动态参数的列表数据
				manyTableData: [],
				// 静态属性的列表数据
				onlyTableData: [],
				// 上传图片的URL地址
				uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
				// 图片上传组件的headers请求对象
				headerObj: {
					Authorization: window.sessionStorage.getItem('token')
				},
				// 预览图片
				fileList: [],
				// 预览图片的路径
				previewPath: '',
				// 控制图片预览对话框的显示与隐藏
				previewVisible: false
			}
		},
		created() {
			this.getGoodsList()
		},
		methods: {
			// 根据商品Id查询商品信息
			async getGoodsList() {
				const {data: res} = await this.$http.get(`goods/${this.id}`)
				
				if (res.meta.status !== 200) {
					return this.$message.error("查询商品信息失败！")
				}
				
				this.editForm = res.data
				// 把商品对应的分类名称Id，通过split()方法分割成数组
				this.goods_array = res.data.goods_cat.split(',')
				
				// 循环分类Id的数组，获取分类Id对应名称
				for (let [index, item] of this.goods_array.entries()) {
					if (index !== this.goods_array.length - 1) {
						this.categoryName += await this.getCateList(item) + " / "
					} else {
						this.categoryName += await this.getCateList(item)
					}
				}
				
				// 处理动态参数，从attrs数组中分离动态参数
				this.manyTableData = this.editForm.attrs.filter(item => {
					return item.attr_sel == "many"
				})
				
				// 将处理过的动态参数属性中attr_vals字符串转化为数组
				this.manyTableData.forEach(item => {
					item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(" ")
				})
				
				// 处理静态参数
				this.onlyTableData = this.editForm.attrs.filter(item => {
					return item.attr_sel == "only"
				})
				
				// 处理图片
				this.fileList = this.editForm.pics.map(item => {
					let json = {}
					json.name = item.pics_id
					json.url = item.pics_sma_url
					return json
				})
				
				console.log(this.editForm)
			},
			// 根据Id查询对应分类的信息
			async getCateList(id) {
				const {data: res} = await this.$http.get(`categories/${id}`)
				
				if (res.meta.status !== 200) {
					return this.$message.error("查询分类信息失败！")
				}
				
				return res.data.cat_name
			},
			// 处理图片预览效果
			handlePreview(file) {
				if (file.response) {
					this.previewPath = file.response.data.url
				} else {
					this.previewPath = file.url
				}
				this.previewVisible = true
			},
			// 处理移除图片的操作
			handleRemove(file) {
				console.log(file)
				// 1.获取将要删除的图片url
				let fileUrl = ''
				if (file.response) {
					fileUrl = file.response.data.url
				} else {
					fileUrl = file.url
				}
				// console.log()
				// 2.从pics数组中，找到这个图片对应的索引值
				const index = this.editForm.pics.findIndex(item => item.pics_sma_url === fileUrl)
				// 3.调用数组的splic方法，把图片信息对象，从pics数组中移除
				this.editForm.pics.splice(index, 1)
			},
			// 监听图片上传成功的事件
			handleSuccess(response) {
				// 1.拼接得到一个图片信息对象
				const picInfo = { 
					pic: response.data.tmp_path,
					pics_sma_url: response.data.url
				}
				// 2.将图片信息对象，push到pics数组中
				this.editForm.pics.push(picInfo)
			},
			// 修改商品
			edit() {
				this.$refs.editFormRef.validate(async valid => {
					if (!valid) {
						return this.$message.error("请填写必要的表单项！")
					}
					
					// 执行修改的业务逻辑
					this.editForm.attrs = []
					// 处理动态参数
					this.manyTableData.forEach(item => {
						const newInfo = {
							attr_id: item.attr_id,
							attr_vals: item.attr_vals.join(' ')
						}
						this.editForm.attrs.push(newInfo)
					})
					// 处理静态参数
					this.onlyTableData.forEach(item => {
						const newInfo = {
							attr_id: item.attr_id,
							attr_vals: item.attr_vals
						}
						this.editForm.attrs.push(newInfo)
					})
					
					// 发起请求修改商品
					const {data: res} = await this.$http.put(`goods/${this.editForm.goods_id}`, {
						goods_name: this.editForm.goods_name,
						goods_cat: this.editForm.goods_cat,
						goods_price: this.editForm.goods_price,
						goods_number: this.editForm.goods_number,
						goods_weight: this.editForm.goods_weight,
						goods_introduce: this.editForm.goods_introduce,
						pics: this.editForm.pics,
						attrs: this.editForm.attrs
					})
					
					console.log(res)
					
					if (res.meta.status !== 200) {
						return this.$message.error("修改商品信息失败！")
					}
					
					this.$message.success("修改商品信息成功！")
					this.$router.push('/goods')
				})
			}
		}
	}
</script>

<style lang="less" scoped>
	.el-checkbox {
		margin: 0 8px 0 0 !important;
	}
	
	.btnEdit {
		margin-top: 15px;
	}
	
	.previewImg {
		width: 100%;
	}
</style>