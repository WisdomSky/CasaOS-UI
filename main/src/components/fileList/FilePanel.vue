<!--
  * @LastEditors: zhanghengxin ezreal.zhang@icewhale.org
  * @LastEditTime: 2023/4/6 下午6:50
  * @FilePath: /CasaOS-UI/src/components/fileList/FilePanel.vue
  * @Description:
  *
  * Copyright (c) 2023 by IceWhale, All Rights Reserved.

  -->

<!--
 * @Author: JerryK
 * @Date: 2021-10-14 14:08:40
 * @LastEditors: Jerryk jerry@icewhale.org
 * @LastEditTime: 2023-02-06 17:45:53
 * @Description: 
 * @FilePath: /CasaOS-UI/src/components/fileList/FilePanel.vue
-->
<template>
	<div class="modal-card">
		<!-- Modal-Card Header Start -->
		<header class="modal-card-head">
			<div class="is-flex-grow-1">
				<h3 class="title is-4 has-text-weight-normal">{{ $t('Select') }}</h3>
			</div>
		</header>
		<!-- Modal-Card Header End -->
		<!-- Modal-Card Body Start -->
		<section class="modal-card-body">

			<nav aria-label="breadcrumbs" class="breadcrumb ">
				<ul>
					<li>
						<a v-if="showPopUp" @click="getParentList()">
							<b-icon icon="arrow-up"></b-icon>
						</a>
					</li>
					<li v-if="showPopUp"><a @click="getFileList(rootPath)">{{ rootName }}</a></li>
					<li v-if="showPopUp & showDots"><a @click="getParentList()">...</a></li>
					<li class="is-active ">
						<div>{{ lastFolder }}</div>
					</li>
				</ul>
			</nav>
			<ul class="file-list scrollbars-light">
				<list-item v-for="(item) in fileList" :id="item.path" :key="item.path" :IsDir="item.is_dir" :item="item"
						   :name="item.name" :path="item.path" :state="checkActive(item.path)" @active="activeFile"
						   @expand="getFileList"></list-item>
			</ul>
		</section>
		<!-- Modal-Card Body End -->
		<!-- Modal-Card Footer Start-->
		<footer class="modal-card-foot is-flex is-align-items-center">
			<div class="is-flex-grow-1">
				<div v-if="rootPath == '/DATA'">
					<b-tooltip :label="$t('Create Folder')" position="is-right" type="is-dark">
						<a class="add-button" @click="showCreatePanel(true)">
							<b-icon icon="folder-plus"></b-icon>
						</a>
					</b-tooltip>
					<template v-if="rootPath != path && showFile">
						<b-tooltip :label="$t('Create File')" position="is-right" type="is-dark">
							<a class="add-button" @click="showCreatePanel(false)">
								<b-icon icon="file-plus-outline"></b-icon>
							</a>
						</b-tooltip>
					</template>
				</div>
			</div>
			<div>
				<b-button :label="$t('Cancel')" rounded type="is-grey" @click="$emit('close')"/>
				<b-button :label="$t('Select')" rounded type="is-dark" @click="selectFile()"/>
			</div>
		</footer>
		<!-- Modal-Card Footer End-->
	</div>
</template>

<script>
import ListItem    from "./ListItem.vue"
import CreatePanel from './CreatePanel.vue'
import trimStart   from 'lodash/trimStart'
import dropRight   from 'lodash/dropRight'

export default {
	name: "file-panel",
	components: {
		ListItem,
	},
	data() {
		return {
			path: this.initPath,
			activePath: this.initPath,
			fileList: [],
		}
	},
	props: {
		initPath: String,
		rootPath: String,
		showFile: {
			type: Boolean,
			default: true
		},
	},
	computed: {
		// get Last foler name for breadcrumb
		lastFolder() {
			return this.path.split("/").pop()
		},
		// check show breadcrumb
		showPopUp() {
			console.log('showItem', this.path, this.rootPath)
			return this.path != this.rootPath
		},
		// check show breadcrumb dots
		showDots() {
			return this.path.split("/").length > 3
		},
		// Root Name
		rootName() {
			return trimStart(this.rootPath, '/');
		},
	},
	created() {
		this.path = (this.path == this.rootPath) ? this.path : dropRight(this.path.split("/"), 1).join("/")
		this.getFileList(this.path, true);
	},

	methods: {
		// get file list from api
		getFileList(path, locate = false) {
			this.$api.folder.getList(path).then(res => {
				if (res.data.success == 200) {
					this.path = path
					if (this.showFile) {
						this.fileList = res.data.data.content;
					} else {
						this.fileList = res.data.data.content.filter((item) => {
							return item.is_dir
						});
					}
					if (locate) {
						this.locateFile();
					} else {
						this.activePath = path
					}
				}
			})
		},

		locateFile() {
			this.$nextTick(() => {
				const activeItem = document.getElementById(this.activePath)
				if (activeItem != null) {
					activeItem.scrollIntoView()
				}
			})
		},

		// get parent list
		getParentList() {
			let backDir = dropRight(this.path.split("/"), 1).join("/");
			if (backDir === "")
				backDir = "/"

			this.getFileList(backDir);
		},
		selectFile() {
			this.$emit('close');
			this.$emit('updatePath', this.activePath);
		},
		activeFile(val) {
			this.activePath = (this.activePath == val) ? this.path : val;
		},
		checkActive(val) {
			return this.activePath == val
		},
		// show create folder or file panel
		showCreatePanel(isFolder) {
			this.$buefy.modal.open({
				parent: this,
				component: CreatePanel,
				hasModalCard: true,
				customClass: 'file-sel-modal',
				trapFocus: true,
				canCancel: ['escape'],
				scroll: "keep",
				animation: "zoom-in",
				events: {
					'reloadPath': (path) => {

						this.getFileList(this.path);
						this.activePath = path;
					}
				},
				props: {
					initPath: (this.path == "") ? this.rootPath : this.path,
					isDir: isFolder
				}
			})
		}
	},
}
</script>
<style lang="scss" scoped>
.file-sel-modal {
	.breadcrumb {
		margin-bottom: 0.5rem;

		a {
			color: #0e9aff;
		}

		ul {
			overflow: hidden;
			flex-wrap: nowrap;
			flex-direction: row;
			justify-content: flex-start;

			li {
				white-space: nowrap;

				div {
					max-width: 100%;
					padding: 0 0.5em;
					overflow: hidden;
					white-space: nowrap;
					text-overflow: ellipsis;
				}

				&:first-child {
					a {
						padding-right: 0;

						.icon {
							margin-left: 0;
						}
					}
				}

				&:last-child {
					min-width: 0;
					width: 100%;
				}

				&:nth-child(2) {
					&::before {
						content: "|";
					}
				}
			}
		}
	}

	.modal-card {
		width: 30rem;
	}

	.file-list {
		height: 19.6875rem;
		overflow-x: hidden;
		overflow-y: auto;

		li {
			border-bottom: #e4e4e4 1px solid;
			line-height: 1.5em;
			border-radius: 4px;
			transition: background-color 0.2s;
			cursor: pointer;

			&:hover {
				background-color: #e0e0e0;
			}

			&.active {
				background-color: #b6e0ff;
			}
		}
	}
}
</style>