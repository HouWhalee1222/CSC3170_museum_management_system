<template>
	<div class="container-vertical-center container-horizontal-center">
		<el-form label-position="left" ref="formRef" :model="formModel" :rules="formRules">
			<el-form-item prop="warehouseId" :label="api.createCheck.attrMap.warehouseId.value">
				<el-select
					:loading="optionLoading"
					v-model="formModel.warehouseId"
					@visible-change="handlerVisibleChange"
				>
					<el-option
						v-for="value in warehouseIdList"
						:key="value.id"
						:value="value.id"
						:label="value.name"
					></el-option>
				</el-select>
			</el-form-item>
			<el-form-item>
				<el-button
					style="width: 100%;"
					type="primary"
					plain
					@click="showCreateWarehouseDialog = true"
				>
					新建仓库
				</el-button>
			</el-form-item>
			<el-form-item>
				<el-button style="width: 100%;" type="primary" @click="startCheck">
					开始盘点
				</el-button>
			</el-form-item>
		</el-form>
		<el-dialog @closed="handlerClosedDialog" title="新建仓库" :visible.sync="showCreateWarehouseDialog">
			<el-form
				label-position="top"
				ref="dialogFormRef"
				:model="dialogFormModel"
				:rules="dialogFormRules"
			>
				<el-form-item prop="name" :label="api.createWarehouse.attrMap.name.value">
					<el-input v-model="dialogFormModel.name"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer">
				<el-button @click="showCreateWarehouseDialog = false">取 消</el-button>
				<el-button type="primary" @click="createWarehouse">确 认</el-button>
			</div>
		</el-dialog>
	</div>
</template>

<script>
import { Message } from 'element-ui'
export default {
	data() {
		return {
			formModel: {
				warehouseId: ''
			},
			warehouseIdList: [],
			formRules: {
				warehouseId: [
					{
						required: true,
						message: '请选择仓库',
						trigger: 'blur'
					}
				]
			},
			optionLoading: false,
			validateState: true,
			showCreateWarehouseDialog: false,
			dialogFormModel: {
				name: ''
			},
			dialogFormRules: {
				name: [
					{
						required: true,
						message: '请输入仓库名称',
						trigger: 'blur'
					}
				]
			},
			dialogValidateState: true
		}
	},
	methods: {
		// 开始盘点
		async startCheck() {
			await this.touchValidate()
			if (!this.validateState) {
				return
			}
			const vm = this
			this.api.createCheck
				.func(this.formModel)
				.then(res => {
					console.log('start check')
					vm.$router.push('/check/' + res.data.data.id)
				})
				.catch(err => {
					console.log('start check fail', err)
				})
		},
		// 触发表单校验
		touchValidate() {
			const vm = this
			this.$refs.formRef.validate((result, object) => {
				vm.validateState = result
			})
		},
		// 新建仓库
		async createWarehouse() {
			await this.touchDialogValidate()
			if (!this.dialogValidateState) {
				return
			}
			const vm = this
			this.api.createWarehouse
				.func(this.dialogFormModel)
				.then(res => {
					console.log('create warehouse success')
					Message.success('新建成功')
					vm.showCreateWarehouseDialog = false
				})
				.catch(err => {
					console.log('create warehouse fail', err)
					vm.showCreateWarehouseDialog = false
				})
		},
		// 触发 dialog 表单校验
		touchDialogValidate() {
			const vm = this
			this.$refs.dialogFormRef.validate((result, object) => {
				vm.dialogValidateState = result
			})
		},
		// 监听select开关
		handlerVisibleChange(event) {
			// 打开的时候加载远程选项
			const vm = this
			if (event) {
				vm.optionLoading = true
				vm.api.createCheck.attrMap.warehouseId
					.remoteSelectApi()
					.then(res => {
						console.log('get options success')
						vm.warehouseIdList = res.data.data
						vm.optionLoading = false
					})
					.catch(err => {
						console.log('get options fail', err)
						vm.optionLoading = false
					})
			}
		},
		// 监听 dialog 关闭
		handlerClosedDialog() {
			this.$refs.dialogFormRef.resetFields()
		}
	}
}
</script>

<style></style>
