<!--
 * @Author: Jerryk jerry@icewhale.org
 * @Date: 2022-02-18 10:20:10
 * @LastEditors: Jerryk jerry@icewhale.org
 * @LastEditTime: 2023-02-12 18:13:52
 * @FilePath: \CasaOS-UI-0.4.2\src\components\Apps\AppSection.vue
 * @Description:
 *
 * Copyright (c) 2022 by IceWhale, All Rights Reserved.
-->

<template>
	<div class="home-section has-text-left">
		<!-- Title Bar Start -->
		<div class="is-flex is-align-items-center mb-4">
			<app-section-title-tip id="appTitle1" class="is-flex-grow-1 has-text-sub-04" label="Drag icons to sort."
								   title="Apps"></app-section-title-tip>

			<b-dropdown animation="fade1" aria-role="menu" class="file-dropdown" position="is-bottom-left">
				<template #trigger>
					<b-icon class="polymorphic is-clickable has-text-grey-100" icon="plus-outline" pack="casa"
							size="is-24"></b-icon>
				</template>
				<b-dropdown-item aria-role="menuitem" @click="showInstall(0, 'custom')">
					{{ $t('Custom Install APP') }}
				</b-dropdown-item>
				<b-dropdown-item aria-role="menuitem" @click="showExternalLinkPanel">
					{{ $t('Add external link/APP') }}
				</b-dropdown-item>
			</b-dropdown>
		</div>
		<!-- Title Bar End -->

		<!-- App List Start -->
		<draggable v-model="appList" :draggable="draggable"
				   class="columns is-variable is-2 is-multiline app-list contextmenu-canvas" tag="div"
				   v-bind="dragOptions"
				   @end="onSortEnd" @start="drag = true">

			<!-- App Icon Card Start -->
			<template v-if="!isLoading">
				<div v-for="(item) in appList" :id="'app-' + item.name" :key="'app-' + item.name"
					 class="column is-narrow is-3 handle">
					<app-card :item="item" @configApp="showConfigPanel" @importApp="showContainerPanel"
							  @updateState="getList"></app-card>
				</div>
			</template>
			<template v-else>
				<div v-for="(index) in skCount" :id="'app-' + index" :key="'app-' + index"
					 class="column is-narrow is-3 handle">
					<app-card-skeleton :index="index"></app-card-skeleton>
				</div>
			</template>
			<!-- App Icon Card End -->
			<!-- <b-loading slot="footer" v-model="isLoading" :is-full-page="false"></b-loading> -->
		</draggable>
		<!-- App List End -->

	</div>
</template>

<script>
import AppCard                      from './AppCard.vue'
import AppCardSkeleton              from './AppCardSkeleton.vue';
import AppPanel                     from './AppPanel.vue'
import ExternalLinkPanel            from "@/components/Apps/ExternalLinkPanel";
import AppSectionTitleTip           from './AppSectionTitleTip.vue'
import draggable                    from 'vuedraggable'
import xor                          from 'lodash/xor'
import concat                       from 'lodash/concat'
import events                       from '@/events/events';
import last                         from 'lodash/last';
import business_ShowNewAppTag       from "@/mixins/app/Business_ShowNewAppTag";
import business_LinkApp             from "@/mixins/app/Business_LinkApp";
import isEqual                      from "lodash/isEqual";
import {loadMicroApp, prefetchApps} from 'qiankun';
import {MIRCO_APP_ACTION_ENUM}      from "@/const";
import {externalMircoApp}           from "@/router";

const SYNCTHING_STORE_ID = 74

// meta_data :: build-in app
const builtInApplications = [
	{
		id: "1",
		name: "App Store",
		title: {
			en_us: "App Store",
		},
		icon: require(`@/assets/img/app/appstore.svg`),
		status: "running",
		app_type: "system"
	}
]

const orderConfig = "app_order"

export default {
	mixins: [business_ShowNewAppTag, business_LinkApp],
	data() {
		return {
			user_id: localStorage.getItem("user_id"),
			appList: [],
			appConfig: {},
			drag: false,
			isLoading: false,
			isShowing: false,
			importHelpText: "Click icon to import.",
			appHelpText: 'Drag icons to sort.',
			draggable: ".handle",
			retryCount: 0,
			appListErrorMessage: "",
			skCount: 0,
			ListRefreshTimer: null,
			// prefetched: false,
			mircoAppInstanceMap: new Map(),
			mircoAppList: []
		}
	},
	components: {
		AppCard,
		draggable,
		AppSectionTitleTip,
		AppCardSkeleton
	},
	provide() {
		return {
			openAppStore: this.showInstall,
			homeShowFiles: this.showMircoApp,
			showMircoApp: this.showMircoApp,
		};
	},
	computed: {
		dragOptions() {
			return {
				animation: 300,
				group: "description",
				disabled: false,
				ghostClass: "ghost",
			};
		},
		showDragTip() {
			return this.draggable === ".handle"
		},
		// exsitingAppsShow() {
		// 	return this.$store.state.existingAppsSwitch
		// }
	},
	async created() {
		await this.initMircoApp();
		this.getList();
		this.draggable = this.isMobile() ? "" : ".handle";
		this.$EventBus.$on(events.OPEN_APP_STORE_AND_GOTO_SYNCTHING, () => {
			this.showInstall(SYNCTHING_STORE_ID)
		});

		this.$EventBus.$on(events.RELOAD_APP_LIST, () => {
			this.getList();
		});

		this.ListRefreshTimer = setInterval(() => {
			// this.getList();
		}, 5000)
	},
	beforeDestroy() {
		this.$EventBus.$off(events.OPEN_APP_STORE_AND_GOTO_SYNCTHING);
		window.removeEventListener('resize', this.getSkCount);

		clearInterval(this.ListRefreshTimer);
		this.mircoAppInstanceMap.forEach((v, name) => {
			this.destroyMircoApp(name);
		});
	},
	mounted() {
		window.addEventListener('resize', this.getSkCount);
		this.getSkCount()
	},
	methods: {

		isMobile() {
			let flag = navigator.userAgent.match(/(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i)
			return flag
		},

		getSkCount() {
			const windowWidth = window.innerWidth
			if (windowWidth < 1024) {
				this.skCount = 4
			} else if (windowWidth < 1216) {
				this.skCount = 6
			} else if (windowWidth < 1408) {
				this.skCount = 8
			} else {
				this.skCount = 10
			}
		},

		/**
		 * @description: Fetch the list of installed apps
		 * @return {*} void
		 */
		async getList() {

			try {
				const orgAppList = await this.$openAPI.appGrid.getAppGrid().then(res => res.data.data || []);
				orgAppList.forEach((item) => {
					item.hostname = item.hostname || this.$baseHostname;
					// Container app does not have icon.
					item.icon = item.icon || require(`@/assets/img/app/default.svg`);
				})
				let listLinkApp = await this.getLinkAppList();
				listLinkApp.forEach((item) => {
					// linkApp does not have title.
					item.title = {
						en_us: item.name
					}
				})
				// all app list
				let casaAppList = concat(builtInApplications, orgAppList, this.mircoAppList, listLinkApp)
				// get app sort info.
				let lateSortList = await this.$api.users.getCustomStorage(orderConfig).then(res => res.data.data.data || []);

				// filter anything not in casaAppList.
				const propList = casaAppList.map((obj) => obj.name);
				const existingList = lateSortList.filter((item) => propList.includes(item));
				const futureList = propList.filter((item) => !lateSortList.includes(item));
				const newSortList = existingList.concat(futureList);

				// then sort.
				const sortedAppList = casaAppList.sort((obj1, obj2) => {
					return newSortList.indexOf(obj1.name) - newSortList.indexOf(obj2.name);
				});

				const sortedList = sortedAppList.map((obj) => obj.name);
				this.appList = sortedAppList;
				if (!isEqual(lateSortList, sortedList)) {
					this.saveSortData()
				}

				this.isLoading = false;
				this.retryCount = 0;
				this.appListErrorMessage = ""
			} catch (error) {
				console.error(error);
				this.isLoading = true;
				if (this.retryCount < 5) {
					setTimeout(() => {
						this.retryCount++
						// this.getList();
					}, 2000)
				} else {

					this.appListErrorMessage = "Failed to get app list."
					this.$buefy.toast.open({
						message: this.$t(`Failed to load apps, please refresh later.`),
						type: 'is-danger'
					})
				}
			}
		},

		async initMircoApp() {
			const mircoAppListRaw = await this.$api.sys.getEntry().then(res => res.data.data || []);
			const prefetchMircoAppList = [];
			this.mircoAppList = mircoAppListRaw.map(item => {
				if (item.prefetch) {
					prefetchMircoAppList.push({name: item.title, entry: item.entry});
				}
				return {
					name: item.name,
					entry: item.entry,
					title: item.title,
					icon: item.icon,
					status: "running",
					app_type: "mircoApp",
					// TODO Resolve metadata structure conflicts and ensure uniformity and non-redundancy in the application's data models.
					formality: item.formality,
					prefetch: item.prefetch
				}
			});
			prefetchApps(prefetchMircoAppList);
			if (externalMircoApp) {
				this.$messageBus('mircoapp_communicate', {
					action: MIRCO_APP_ACTION_ENUM.OPEN,
					name: externalMircoApp
				})
			}
		},

		createMircoApp(app) {
			const customVNode = this.$createElement('div', {
				class: "full-screen-container",
				attrs: {
					id: app.name
				}
			});
			const customModal = this.$buefy.modal.open({
				content: [customVNode],
				fullScreen: app.formality?.props?.fullscreen || true,
				hasModalCard: app.formality?.props?.hasModalCard || true,
				destroyOnHide: false,
				animation: app.formality?.props?.animation || "zoom-in",
				canCancel: ["escape", "x"],
				onCancel: () => {
					this.hideMircoApp(app.name);
				}
			});

			this.$nextTick(() => {
				try {
					const customAppInstance = loadMicroApp({
						name: app.name,
						entry: app.entry,
						container: `#${app.name}`,
						props: {
							store: { // sync necessary store status to child mirco app
								device_id: this.$store.state.device_id,
								access_id: this.$store.state.access_id,
								access_token: this.$store.state.access_token,
								refresh_token: this.$store.state.refresh_token,
								casaos_lang: this.$store.state.casaos_lang,
							}
						}
					}, {
						sandbox: {
							experimentalStyleIsolation: true
						}
					});
					this.mircoAppInstanceMap.set(app.name, {
						instance: customAppInstance,
						modal: customModal
					});
				} catch (e) {
					this.$buefy.toast.open({
						message: `Error occured in loading mirco app ${app.name}, please check mirco app`,
						duration: 5000,
						type: "is-danger"
					});
				}
			});
		},

		hideMircoApp(peerType = '') { // NOTICE: hide all mirco app for now
			if (peerType) {
				let {modal} = this.mircoAppInstanceMap.get(peerType) || {};
				modal?.close();
				return
			}
			this.mircoAppInstanceMap.forEach(({modal}) => modal?.close());
		},

		showMircoApp(appName) {
			const {instance = null, modal = null} = this.mircoAppInstanceMap.get(appName) || {};
			if (!instance) {
				let app = this.mircoAppList.find(app => app.name === appName)
				app && this.createMircoApp(app);
			} else {
				modal?.open();
			}
		},

		destroyMircoApp(name = '') {
			this.hideMircoApp();
			if (this.mircoAppInstanceMap.has(name)) {
				const {instance, modal} = this.mircoAppInstanceMap.get(name);
				instance?.unmount();
				modal?.close();
				this.mircoAppInstanceMap.delete(name);
			}
		},

		/**
		 * @description:
		 * @param {Array} oriList
		 * @param {Array} newList
		 * @return {*}
		 */
		getNewSortList(oriList, newList) {
			let xorList = xor(oriList, newList)
			// xorList.reverse()
			return concat(oriList, xorList)
		},

		/**
		 * @description: Save Sort Table
		 * @param {*}
		 * @return {*}
		 */
		saveSortData() {
			let newList = this.appList.map((item) => {
				// compose milestone :: name is unique, global index.
				return item.name
			})
			let data = {
				data: newList
			}
			this.$api.users.setCustomStorage(orderConfig, data)
		},
		/**
		 * @description: Handle on Sort End
		 * @param {*}
		 * @return {*}
		 */
		onSortEnd() {
			this.drag = false
			this.saveSortData()
		},

		/**
		 * @description: Show Install Panel Programmatic
		 * @return {*} void
		 */
		async showInstall(storeId = 0, mode) {
			if (mode === 'custom') {
				this.$messageBus('apps_custominstall');
			}
			this.isShowing = true

			const networks = await this.$api.container.getNetworks();
			const memory = this.$store.state.hardwareInfo.mem;
			const configData = {
				networks: networks.data.data,
				memory: memory
			}
			this.isShowing = false
			this.$buefy.modal.open({
				parent: this,
				component: AppPanel,
				hasModalCard: true,
				customClass: 'app-panel',
				trapFocus: true,
				canCancel: ['escape'],
				scroll: "keep",
				animation: "zoom-in",
				events: {
					'updateState': () => {
						this.getList()
					}
				},
				props: {
					id: "0",
					state: "install",
					configData: configData,
					storeId: storeId,
					// TODO transfer to yaml string.
					settingData: mode !== 'custom' ? undefined : {},
				}
			})
		},

		/**
		 * @description: Show Settings Panel Programmatic
		 * @param {Object} {id:String,status:String }
		 * @param {Boolean} isCasa
		 * @return {*}
		 */
		async showConfigPanel(item, isCasa) {
			let name = item.name;
			this.$messageBus('appsexsiting_open', name);
			try {
				if (item.app_type === 'LinkApp') {
					await this.showExternalLinkPanel(item)
					return
				}
				const networks = await this.$api.container.getNetworks();
				const memory = this.$store.state.hardwareInfo.mem;
				const configData = {
					networks: networks.data.data,
					memory: memory
				}
				const ret = await this.$openAPI.appManagement.compose.myComposeApp(name, {
					headers: {
						'content-type': 'application/yaml',
						'accept': 'application/yaml'
					}
				});
				this.$buefy.modal.open({
					parent: this,
					component: AppPanel,
					hasModalCard: true,
					customClass: '',
					trapFocus: true,
					canCancel: [''],
					scroll: "keep",
					animation: "zoom-in",
					events: {
						'updateState': () => {
							this.getList()
						}
					},
					props: {
						id: name,
						state: "update",
						isCasa: isCasa,
						// 区分 terminal
						runningStatus: item.status,
						configData: configData,
						// settingData: ret.data,
						settingComposeData: ret.data,
					}
				})
			} catch (e) {
				console.error(e)
			}
		},

		async showContainerPanel(item) {
			this.$messageBus('appsexsiting_open', item.name);
			let id = item.name
			const networks = await this.$api.container.getNetworks();
			const memory = this.$store.state.hardwareInfo.mem;
			const configData = {
				networks: networks.data.data,
				memory: memory
			}
			const ret = await this.$api.container.getInfo(id);
			this.$buefy.modal.open({
				parent: this,
				component: AppPanel,
				hasModalCard: true,
				customClass: '',
				trapFocus: true,
				canCancel: [''],
				scroll: "keep",
				animation: "zoom-in",
				events: {
					'updateState': () => {
						this.getList()
					}
				},
				props: {
					id: id,
					state: "update",
					isCasa: false,
					runningStatus: item.status,
					configData: configData,
					settingData: ret.data.data
				}
			})
		},

		async showExternalLinkPanel(item = {}) {
			this.$buefy.modal.open({
				parent: this,
				component: ExternalLinkPanel,
				hasModalCard: true,
				customClass: '',
				trapFocus: true,
				canCancel: [''],
				scroll: "keep",
				animation: "zoom-in",
				events: {
					'updateState': () => {
						this.$messageBus('apps_external');
						this.getList().then(() => {
							this.scrollToNewApp();
						})
					}
				},
				props: {
					linkName: item.name,
					linkHost: item.hostname,
					linkIcon: item.icon,
				}
			})
		},

		scrollToNewApp() {
			// business :: scroll to last position
			let name = last(this.newAppIds);
			let showEl = document.getElementById("app-" + name)
			showEl && showEl.scrollIntoView({behavior: "smooth", block: 'end'});
		},

		messageBusToast(message, type) {
			let duration = 5000
			this.$buefy.toast.open({
				message: message,
				duration,
				type,
			})
		}
	},
	sockets: {
		"app:install-end"() {
			this.getList().then(() => {
				this.scrollToNewApp();
			});
		},
		"app:install-error"() {
			this.getList().then(() => {
				this.scrollToNewApp();
			});
		},
		"app:uninstall-end"() {
			this.getList();
		},
		"app:apply-changes-error"(res) {
			// toast info
			this.messageBusToast(res.Properties.message, 'is-danger');
		},
		"app:apply-changes-end"(res) {
			// toast info
			this.messageBusToast(res.Properties['app:name'] + ' is OK', 'is-success');

			// business :: Tagging of new app / scrollIntoView
			this.addIdToSessionStorage(res.Properties['app:name'])

			this.getList().then(() => {
				this.scrollToNewApp();
			});
		},
		/**
		 * @description: Update App Version
		 * @param {Object} data
		 * @return {void}
		 */
		'app:update-end'(data) {
			if (data.Properties['docker:image:updated'] === "true") {
				// business :: Tagging of new app / scrollIntoView
				this.addIdToSessionStorage(data.Properties['app:name'])

				this.$buefy.toast.open({
					message: this.$t(`{name} has been updated to the latest version!`, {
						name: data.Properties.name
					}),
					type: 'is-success'
				})
				this.getList().then(() => {
					this.scrollToNewApp();
				});
			}
		},
		'app:update-error'(data) {
			if (data.Properties.cid === this.item.id) {
				this.isUpdating = false;
				this.$buefy.toast.open({
					message: this.$t(data.Properties['error']),
					type: 'is-danger'
				})
			}
		},
		"casaos-ui:app:mircoapp_communicate"(res) {
			const data = res.Properties;
			if (data.access_id === this.$store.state.access_id) {
				switch (data.action) {
					case MIRCO_APP_ACTION_ENUM.OPEN:
						this.showMircoApp(data.name);
						break;
					case MIRCO_APP_ACTION_ENUM.CLOSE:
						this.hideMircoApp(data.name);
						break;
					// case MIRCO_APP_ACTION_ENUM.LOGIN:
					// 	this.$router.push("/login");
					// 	break;
					default:
						break;
				}
			}
		}
	}
}
</script>

<style lang="scss" scoped>
.app-list {
	position: relative;
}

@media screen and (max-width: 480px) {
	.app-list {
		display: flex;

		.column {
			flex: none;
			width: 50%;
		}
	}
}

@media screen and (max-width: $tablet) {
	.app-list {
		display: flex;

		.column {
			flex: none;
			width: 50%;
		}
	}
}

@media screen and (min-width: $tablet) {
	.app-list {
		.column {
			flex: none;
			width: 50%;
		}
	}
}

@media screen and (min-width: $desktop) {
	.app-list {
		.column {
			flex: none;
			width: 33.333333%;
		}
	}
}

@media screen and (min-width: $widescreen) {
	.app-list {
		.column {
			flex: none;
			width: 25%;
		}
	}
}

@media screen and (min-width: $fullhd) {
	.app-list {
		.column {
			flex: none;
			width: 20%;
		}
	}
}
</style>
