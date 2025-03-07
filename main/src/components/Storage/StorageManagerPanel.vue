<!--
 * @LastEditors: Jerryk jerry@icewhale.org
 * @LastEditTime: 2023-01-16 18:59:52
 * @FilePath: /CasaOS-UI/src/components/Storage/StorageManagerPanel.vue
  * @Description:
  *
  * Copyright (c) 2022 by IceWhale, All Rights Reserved.
  -->
<template>
	<div class="modal-card">

		<!-- Modal-Card Body Start -->
		<template v-if="!isCreating">
			<header class="pl-5 mt-4 pt-1 b-line">
				<h3 class="title is-3 mb-3">{{ title }}</h3>
				<div class="close-container">
					<button class="delete" type="button" @click="$emit('close')"/>
				</div>
			</header>
			<section :class="{ 'b-line': storageData.length > 0 && activeTab === 0}" class="pr-5 pl-5 mt-4 pb-2">
				<!-- Storage and Disk List Start -->
				<div v-if="!creatIsShow" class="is-flex-grow-1 is-relative">
					<div v-if="activeTab == 1" class="create-container">
						<popper :options="{ placement: 'bottom', modifiers: { offset: { offset: '0,4px' } } }"
								append-to-body
								trigger="hover">
							<div v-show="unDiskData.length == 0" class="popper  tooltip-content dark">
								{{ $t('Please insert a Drive to Create Storage') }}
							</div>
							<div slot="reference">
								<b-button :disabled="unDiskData.length == 0" :type="state_createstorage_operability"
										  rounded @click="showCreate">{{ $t('Create Storage') }}
								</b-button>
							</div>
						</popper>
					</div>
					<b-tabs v-model="activeTab" animateInitially animated class="region-box">
						<b-tab-item :disabled="!isShowMergeTab" :label="$t('Merge')"
									class="scrollbars-light-auto tab-item">
							<MergeStorages @update="()=> {getDiskList(); activeTab = 1;}"></MergeStorages>
						</b-tab-item>
						<b-tab-item :label="$t('Storage')" class="scrollbars-light-auto tab-item">
							<storage-combination :storageData="mergeConbinationsStorageData"
												 :type="state_mainstorage_operability"
												 @reload="getDiskList"></storage-combination>
							<template v-if="storageData.length">
								<storage-item v-for="(item, index) in storageData" :key="'storage' + index" :item="item"
											  @getDiskList="getDiskList"></storage-item>
							</template>
							<div v-if="showTipsMergeDisks" class="_background-tips" style="position: relative">
								<div class="is-flex is-align-items-center is-justify-content-center is-flex-grow-1">
									<div>
										<p class="has-text-title-06">Tips:</p>
										<p class="has-text-full-03">Use as ONE space, Try Merge</p>
									</div>
									<b-image :src="require('@/assets/img/learn/tips-mergeStorage.svg')"></b-image>
								</div>
								<b-icon class="cursor-pointer" icon="close-outline" pack="casa"
										size="is-small"
										style="position: absolute;top: 0.25rem;right: 0.25rem;"
										@click=""></b-icon>
							</div>
						</b-tab-item>
						<b-tab-item :label="$t('Drive')" class="scrollbars-light-auto tab-item">
							<drive-item v-for="(item, index) in diskData" :key="'disk' + index"
										:item="item"></drive-item>
							<div v-if="diskData.length === 1"
								 class="is-flex is-align-items-center is-justify-content-center _background-tips">
								<div>
									<p class="has-text-title-06">Tips:</p>
									<p class="has-text-full-03">Insert more hard drives and restart.</p>
								</div>
								<b-image :src="require('@/assets/img/learn/tips-insertMoreDrives.svg')"></b-image>
							</div>
						</b-tab-item>
					</b-tabs>

				</div>
				<!-- Storage and Disk List End -->

				<!-- Create Storage Start -->
				<div v-if="creatIsShow" class="is-flex-grow-1 is-relative">
					<ValidationObserver ref="ob1">
						<ValidationProvider v-slot="{ errors, valid }" name="StorageName" rules="required">
							<b-field :label="$t('Storage Name')" :message="$t(errors)"
									 :type="{ 'is-danger': errors[0], 'is-success': valid }">
								<b-input v-model="createStorageName"
										 @keyup.native="createStorageName = createStorageName.replace(/[^\w]/g, '')"
										 @paste.native="createStorageName = createStorageName.replace(/[^\w]/g, '')"></b-input>
							</b-field>
						</ValidationProvider>

						<b-field :label="$t('Choose Drive')">
							<b-select v-model="activeDisk" expanded @input="onDiskChoose">
								<option v-for="(option, index) in unDiskData" :key="option.path" :value="index">
									{{ option.name }} ({{ option.model }} - {{ renderSize(option.size) }})
								</option>
							</b-select>
						</b-field>

					</ValidationObserver>

					<article v-if="createStorageType == 'format'" class="message is-danger mt-5">
						<section class="message-body">
							<div class="media">
								<div class="media-left">
									<span class="icon is-large is-danger"><i class="mdi mdi-alert-circle mdi-48px"></i></span>
								</div>
								<div class="media-content">
									<h3 class="is-size-5">{{ $t('Warning') }}</h3>
									<p class="is-size-14px">
										{{ $t("The selected drive will be emptied.") }}<br>
										{{
											$t(`Please make sure again that there is no important data on the selected drive
										that needs to be
										backed up.`)
										}}
									</p>
								</div>
							</div>
						</section>
					</article>

					<article v-else class="message is-danger mt-5">
						<section class="message-body">
							<div class="media">
								<div class="media-left">
									<span class="icon is-large is-danger"><i class="mdi mdi-alert-circle mdi-48px"></i></span>
								</div>
								<div class="media-content">
									<h3 class="is-size-5">{{ $t('Attention') }}</h3>
									<p class="is-size-14px">
										{{ $t("The drive you select can be used directly as storage.") }}<br>
										{{
											$t(`You can also choose to create it after formatting. If formatted, the
										selected drive will be
										emptied.`)
										}}<br>
										{{
											$t(`Please make sure again that there is no important data on the selected drive
										that needs to be
										backed up.`)
										}}
									</p>
								</div>
							</div>
						</section>
					</article>

				</div>
				<!-- Create Storage End -->
				<b-loading v-model="isLoading" :can-cancel="false" :is-full-page="false"></b-loading>
			</section>
		</template>
		<section v-if="isCreating" class="modal-card-body ">
			<div class="installing-warpper mt-6 mb-6">
				<div class="is-flex is-align-items-center is-justify-content-center mb-5">
					<lottie-animation :animationData="require('@/assets/ani/creating.json')" :autoPlay="true"
									  :loop="true"
									  class="creating-animation"></lottie-animation>
				</div>
				<h3 class="title is-4 has-text-centered has-text-weight-light">{{ $t('Creation in progress') }}...</h3>
			</div>
		</section>

		<!-- Modal-Card Body End -->
		<!-- Modal-Card Footer Start-->
		<footer v-if="!isCreating && creatIsShow" class="modal-card-foot is-flex is-align-items-center">
			<div class="is-flex-grow-1"></div>
			<div>
				<b-button :label="$t('Cancel')" rounded @click="showDefault"/>
				<b-button :label="$t('Format and Create')" :loading="isValiding"
						  :type="createStorageType == 'format' ? 'is-primary' : ''" rounded
						  @click="createStorge(true)"/>
				<b-button v-if="createStorageType == 'mountable'" :label="$t('Create')" :loading="isValiding"
						  rounded
						  type="is-primary" @click="createStorge(false)"/>
			</div>
		</footer>
		<!-- Modal-Card Footer End -->

	</div>
</template>

<script>
import LottieAnimation                          from "lottie-web-vue";
import smoothReflow                             from 'vue-smooth-reflow'
import delay                                    from 'lodash/delay';
import max                                      from 'lodash/max';
import {ValidationObserver, ValidationProvider} from "vee-validate";
import {mixin}                                  from '@/mixins/mixin';
import DriveItem                                from './DriveItem.vue'
import StorageItem                              from './StorageItem.vue'
import Popper                                   from 'vue-popperjs';
import StorageCombination                       from "./StorageCombination.vue";
import cToolTip                                 from '@/components/basicComponents/tooltip/tooltip.vue';
import events                                   from '@/events/events';
import MergeStorages                            from '@/components/Storage/MergeStorages.vue';

export default {
	name: "storage-manager-panel",
	components: {
		LottieAnimation,
		ValidationObserver,
		ValidationProvider,
		DriveItem,
		StorageItem,
		Popper,
		StorageCombination,
		cToolTip: cToolTip,
		MergeStorages,
	},
	mixins: [smoothReflow, mixin],
	data() {
		return {
			isLoading: true,
			creatIsShow: false,
			isCreating: false,
			isValiding: false,
			activeTab: 2,
			activeDisk: "",
			createStorageName: "",
			createStoragePath: "",
			createStorageSeiral: "",
			createStorageType: "",
			diskData: [],
			unDiskData: [],
			storageData: [],
			mergeConbinationsStorageData: [],
			hasMergeState: false,
			mergeStorageList: [],
		}
	},

	computed: {
		title() {
			return this.creatIsShow ? this.$t('Create Storage') : this.$t('Storage Manager')
		},
		state_createstorage_operability() {
			if (this.unDiskData.length == 0) {
				return "is-link is-light"
			}
			return "is-link"
		},
		state_mainstorage_operability() {
			if (this.unDiskData.length == 0) {
				return "is-link"
			}
			return ""
		},
		isShowMergeTab() {
			return this.storageData.length > 1 || this.mergeConbinationsStorageData.length > 0
		},
		showTipsMergeDisks() {
			// TODO test localstorage. exit : true.
			return this.unDiskData === 0 && this.mergeConbinationsStorageData.length === 0 && this.storageData.length > 1
		},
	},

	watch: {
		'activeTab': {
			handler(val, oldVal) {
				if (val === oldVal) {
					return
				}
				switch (val) {
					case 0:
						this.$messageBus('storagemanager_storage');
						break;
					case 1:
						this.$messageBus('storagemanager_drive');
						break;
				}
			}
		}
	},
	mounted() {
		//Smooth
		this.$smoothReflow({
			el: '.modal-card',
			property: ['height', 'width'],
			transition: 'height .25s ease, width .75s ease-out'
		})

		//Get disk list
		let _this = this
		delay(async function () {
			await _this.getDiskList()
			if (_this.diskData.length === 1) {
				_this.activeTab = 2
			} else {
				_this.activeTab = 1
			}
		}, 150);

		this.$EventBus.$on(events.REFRESH_DISKLIST, () => {
			this.getDiskList()
		});
	},

	beforeDestroy() {
		this.$EventBus.$off(events.REFRESH_DISKLIST)
	},

	methods: {
		/**
		 * @description: Get disk list
		 * @param {}
		 * @return {void}
		 */
		async getDiskList(showDefault = false) {

			// get disk list
			try {
				const diskRes = await this.$api.disks.getDiskList()
				this.diskData = diskRes.data.data.disks
				this.unDiskData = diskRes.data.data.avail
			} catch (error) {
				console.log(error.response.message);
			}

			// get storage list
			// TODO: the part is repetition
			//  with APPs Installation Location requirement document
			// 获取merge信息
			let mergeStorageList
			try {
				mergeStorageList = await this.$api.local_storage.getMergerfsInfo().then((res) => res.data.data[0]['source_volume_uuids'])
			} catch (e) {
				mergeStorageList = []
				console.log(e)
			}

			try {
				// get storage list info
				const storageRes = await this.$api.storage.list({system: "show"}).then(v => v.data.data)
				let storageArray = []
				let mergeConbinations = []
				let testMergeMiss = mergeStorageList
				storageRes.forEach(item => {
					item.children.forEach(part => {
						part.disk = item.path
						part.diskName = item.disk_name
						// storageArray.push(part)
						if (mergeStorageList.includes(part.uuid) || (mergeStorageList.length > 0 && item.disk_name === 'System')) {
							mergeConbinations.push(part)
							testMergeMiss = testMergeMiss.filter(v => v !== part.uuid)
						} else {
							storageArray.push(part)
						}
					})
				})
				// sort Business: The System comes first.
				let storageArraySort = storageArray.sort((a, b) => {
					if (a.diskName === 'System') {
						return -1
					} else if (b.diskName === 'System') {
						return 1
					} else {
						return 0
					}
				})
				let mergeConbinationsSort = mergeConbinations.sort((a, b) => {
					if (a.diskName === 'System') {
						return -1
					} else if (b.diskName === 'System') {
						return 1
					} else {
						return 0
					}
				});
				// mergeConbinations.reverse();
				testMergeMiss.forEach(item => {
					mergeConbinationsSort.push({
						"uuid": "",
						"mount_point": "",
						"size": "",
						"avail": "",
						"type": "",
						"path": item,
						"drive_name": "",
						"label": "",
						"persisted_in": "",
						"disk": "",
						"diskName": ""
					})
				})

				this.storageData = storageArraySort.map((storage) => {
					return {
						uuid: storage.uuid,
						name: storage.label,
						isSystem: storage.diskName == "System",
						fsType: storage.type,
						size: storage.size,
						availSize: storage.avail,
						usePercent: 100 - Math.floor(storage.avail * 100 / storage.size),
						diskName: storage.drive_name,
						path: storage.path,
						mount_point: storage.mount_point,
						disk: storage.disk
					}
				})
				this.mergeConbinationsStorageData = mergeConbinationsSort.map((storage) => {
					return {
						uuid: storage.uuid,
						name: storage.label,
						isSystem: storage.diskName == "System",
						fsType: storage.type,
						size: storage.size,
						availSize: storage.avail,
						usePercent: 100 - Math.floor(storage.avail * 100 / storage.size),
						diskName: storage.drive_name,
						path: storage.path,
						mount_point: storage.mount_point,
						disk: storage.disk
					}
				})

				let diskNumArray = this.storageData.map(storage => {
					if (storage.name.includes("Storage")) {
						let diskNum = storage.name.replace("Storage", "")
						return (/^\d+$/.test(diskNum)) ? Number(diskNum) : 0
					} else {
						return 0
					}
				})
				let nextMaxNum = max(diskNumArray) + 1;
				if (this.unDiskData.length > 0) {
					this.createStoragePath = this.unDiskData[0].path
					this.createStorageSeiral = this.unDiskData[0].serial
					this.createStorageType = this.getDiskType(this.unDiskData[0])
					this.createStorageName = "Storage" + nextMaxNum
					this.activeDisk = 0
				}
				if (showDefault) {
					this.showDefault()
					this.isCreating = false
					this.createStorageName = ""
				}
			} catch (error) {
				console.log(error);
			}

			this.isLoading = false
		},

		disksSort(array) {
			array.sort((a, b) => {
				if (a.diskName == "System") {
					return -1
				} else if (b.diskName == "System") {
					return 1
				} else if (a.label > b.label) {
					return 1
				} else if (a.label < b.label) {
					return -1
				} else {
					return 0
				}
			})
		},

		/**
		 * @description: Disk choose handle
		 * @param {int} evt index of select
		 * @return {void}
		 */
		onDiskChoose(index) {
			this.createStoragePath = this.unDiskData[index].path
			this.createStorageSeiral = this.unDiskData[index].serial
			this.createStorageType = this.getDiskType(this.unDiskData[index])
		},
		showDefault() {
			this.creatIsShow = false
		},
		showCreate() {
			this.$messageBus('storagemanager_createstorage');
			this.creatIsShow = true
			let diskNumArray = this.storageData.map(disk => {
				if (disk.name.includes("Storage")) {
					let diskNum = disk.name.replace("Storage", "")
					return (/^\d+$/.test(diskNum)) ? Number(diskNum) : 0
				} else {
					return 0
				}
			})
			let nextMaxNum = max(diskNumArray) + 1;
			this.createStorageName = "Storage" + nextMaxNum
		},

		// show storage settings modal
		async showStorageSettingsModal() {
			this.$messageBus('storagemanager_mergestorage');
			// TODO: the part is repetition
			//  with APPs Installation Location requirement document
			// 获取merge信息
			let mergeStorageList
			try {
				mergeStorageList = await this.$api.local_storage.getMergerfsInfo().then((res) => res.data.data[0]['source_volume_uuids'])
			} catch (e) {
				mergeStorageList = []
				console.log(e)
			}

			this.$buefy.modal.open({
				parent: this,
				component: () => import('@/components/Storage/MergeStorages.vue'),
				hasModalCard: true,
				trapFocus: true,
				ariaModal: true,
				canCancel: ['escape'],
				onCancel: () => {
					this.$EventBus.$emit(events.REFRESH_DISKLIST);
				},
				events: {
					close: () => {
						this.$EventBus.$emit(events.REFRESH_DISKLIST);
					}
				},
				props: {
					mergeStorageList
				}
			})

		},

		/**
		 * @description: Validate form async
		 * @param {Object} ref ref of component
		 * @return {Boolean}
		 */
		async checkStep(ref) {
			let isValid = await ref.validate()
			return isValid
		},
		/**
		 * @description: Create a new storage
		 * @param {}
		 * @return {void}
		 */
		createStorge(needFormat) {
			this.isValiding = true
			this.checkStep(this.$refs.ob1).then(val => {
				this.isValiding = false
				if (val) {
					this.submitCreate(needFormat)
				}
			}).catch(err => {
				this.isValiding = false
				this.$buefy.toast.open({
					duration: 3000,
					message: err.response.data.message,
					type: 'is-danger'
				})
				console.error(err)
			})

		},
		submitCreate(format) {
			this.isCreating = true
			let data = {
				path: this.createStoragePath,
				name: this.createStorageName,
				format: format
			}
			this.$api.storage.create(data).then((res) => {

				if (res.data.success != 200) {
					this.isCreating = false;
					this.$buefy.toast.open({
						duration: 3000,
						message: res.data.message,
						type: 'is-danger'
					})
					console.error(res.data.message)
				} else {
					this.getDiskList(true);
				}
			}).catch(err => {
				this.isCreating = false
				this.$buefy.toast.open({
					duration: 3000,
					message: err.response.data.message,
					type: 'is-danger'
				})
				console.error(err)
			})
		},

		getDiskType(item) {
			if (item.children_number == 0)
				return "format"
			return item.need_format ? "format" : "mountable"
		}
	}
}
</script>


<style lang="scss" scoped>
.create-container {
	height: 2.25rem;
	position: absolute;
	right: 0;
	display: flex;
	align-items: center;
}

.storage-modal {
	.modal-card-body {
		overflow-y: hidden;
		transition: height 0.3s;
		padding: 2rem 2rem 2rem 2rem;
		position: relative;

		.tab-item {
			max-height: calc(100vh - 15rem);
			overflow-y: overlay;
		}
	}

	.modal-card-foot {
		padding-top: 0;
	}

	.close-container {
		position: absolute;
		right: 1.25rem;
		top: 1rem;
	}

	.tab-content {
		padding: 0;
	}

	.status {
		min-width: 7.75rem;
	}
}

.popper {
	background-color: #505459;
	padding: 0.35rem 0.75rem;
	box-shadow: 0px 1px 2px 1px rgba(0, 1, 0, 0.2);
	border: none;
	color: #ffffff;
	border-radius: 6px;
	font-size: 0.85rem;
	font-weight: 400;
}

.b-line {
	border-bottom: 1px solid hsla(208, 16%, 94%, 1);
}

.tab-content {
	padding: 1.5rem 0 0;
}

._background-tips {
	background-color: hsla(208, 16%, 98%, 1);
	border: 1px solid hsla(208, 16%, 91%, 1);
	border-radius: 0.5rem;
}
</style>

<style lang="scss">
.popper[x-placement^="bottom"].dark .popper__arrow {
	border-color: transparent transparent #505459 transparent !important;
}

.region-box .tabs ul {
	display: flex;
	justify-content: center;
	align-content: center;
	flex-grow: 0 !important;
	background-color: hsla(208, 16%, 91%, 1);
	border-radius: 6px;
	border: 2px solid hsla(208, 16%, 91%, 1);

	li {
		/* Text 500Medium/Text03 */

		font-family: 'Roboto';
		font-style: normal;
		font-weight: 500 !important;
		font-size: 14px;
		line-height: 20px;
		/* identical to box height, or 143% */

		text-align: center;
		font-feature-settings: 'pnum' on, 'lnum' on;

		&.is-active {
			background-color: hsla(0, 0%, 100%, 1);
			border-radius: 4px;
			height: 1.75rem;

			a {
				border-bottom: none !important;
				color: hsla(208, 20%, 20%, 1);
			}
		}

		a {
			margin: 0 0.75rem !important;
			padding: 0.25rem 0;
			color: hsla(208, 14%, 58%, 1);
			border-bottom: none !important;
		}
	}
}
</style>
