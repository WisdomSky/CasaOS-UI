<!--
 * @Author: Jerryk jerry@icewhale.org
 * @Date: 2022-08-05 15:13:41
 * @LastEditors: Jerryk jerry@icewhale.org
 * @LastEditTime: 2022-08-10 15:27:39
 * @FilePath: /CasaOS-UI/src/components/share/ShareModal.vue
 * @Description:
 *
 * Copyright (c) 2022 by IceWhale, All Rights Reserved.
-->
<template>
	<div class="modal-card">
		<!-- Modal-Card Header Start -->
		<header class="modal-card-head">
			<div class="is-flex-grow-1">
				<h3 class="title is-3">{{ $t('Share {CasaOS}', {CasaOS: TITLE}) }}</h3>
			</div>
			<div>
				<button class="delete" type="button" @click="$emit('close')"/>
			</div>
		</header>
		<!-- Modal-Card Header End -->
		<!-- Modal-Card Body Start -->
		<section class="modal-card-body ">
			<div class="node-card  mt-5 mb-5">

				<div>
					<div class=" is-size-14px">{{
							$t('Please invite more friends who are concerned about family and data privacy to join and use CasaOS.')
						}}
					</div>

					<b-image :src="require('@/assets/img//social/share_img.png')"
							 class="share-img-shadow share-img"></b-image>
				</div>

				<div class="buttons is-justify-content-center mb-6 mt-4">
					<ShareNetwork v-for="site in shareSites" :key="site" :description="shareTitle" :img="githubUrl"
								  :network="site"
								  :title="shareTitle" :url="githubUrl" hashtags="homecloud,opensource">
						<b-button :icon-left="site" :type="`is-${site}`" class="ml-3 mr-3" icon-pack="casa">
							Share
						</b-button>
					</ShareNetwork>
				</div>

			</div>
		</section>
		<!-- Modal-Card Body End -->
	</div>
</template>

<script>
import {marked} from 'marked'

export default {
	inject: ['TITLE'],
	props: {
		changeLog: {
			type: String,
			default: ""
		},
	},
	data() {
		return {
			timer: 0,
			updateTimer: 0,
			githubUrl: `https://raw.githubusercontent.com/IceWhaleTech/logo/main/casaos/0.4/casaos_social_share.png`,
			shareTitle: `I'm using CasaOS, a simple, easy-to-use, elegant open-source home cloud system, try it like me.`,
			shareSites: [
				'facebook',
				'twitter',
				'reddit'
			]
		};
	},
	computed: {
		markdownToHtml() {
			return marked.parse(this.changeLog);
		}
	},
	methods: {
		/**
		 * @description: Update System Version and check update state
		 * @return {*} void
		 */
		async updateSystem() {
			this.isUpdating = true;
			await this.$api.sys.updateCasaOS();
			// this.checkUpdateState();
			this.getUpdateLogs()
		},

		/**
		 * @description: Get update logs
		 * @return {*} void
		 */
		getUpdateLogs() {
			this.updateTimer = setInterval(() => {
				// this.$api.file.getContent(`/var/log/casaos/upgrade.log`).then(res => {
				this.$openAPI.iceFile.getFileDownload(`/var/log/casaos/upgrade.log`).then(res => {

					this.updateLogs = res.data.data;
					if (this.updateLogs.includes(`CasaOS upgrade successfully`)) {
						clearInterval(this.updateTimer);
						setTimeout(() => {
							location.reload();
						}, 1000);
					} else if (this.updateLogs.includes(`CasaOS upgrade failed`)) {
						this.$buefy.toast.open({
							message: this.$t(`There seems to be a problem with the upgrade process, please try again!`),
							type: 'is-danger'
						})
						clearInterval(this.updateTimer);
						setTimeout(() => {
							this.isUpdating = false;
						}, 1000);

					}
				})
			}, 200);
		},
		/**
		 * @description: check update state if is_need is false then reload page
		 * @return {*} void
		 */
		checkUpdateState() {
			this.timer = setInterval(() => {
				this.$api.sys.getVersion().then(res => {
					if (res.data.success == 200) {
						if (!res.data.data.is_need) {
							clearInterval(this.timer);
							location.reload();
						}
					}
				})
			}, 3000)
		},
	},
}
</script>

<style lang="scss">
.share-img-shadow {
	box-shadow: 0px 16px 24px 2px rgba(115, 120, 128, 0.4);
}

.share-img {
	margin: 3rem 6rem 4rem 6rem;
}

.update-info-container {
	overflow: hidden;
	min-height: 20rem;
	background: #f8f8f8;
	border: 1px solid rgba(0, 0, 0, 0.1);
	border-radius: 0.75rem;
	padding: 1.5rem;

	h1,
	h2,
	h3,
	h4,
	h5,
	h6 {
		font-weight: bold;
		margin-bottom: 0.4rem;
	}

	h1 {
		font-size: 2em;
	}

	h2 {
		font-size: 1.5em;
	}

	h3 {
		font-size: 1.17em;
	}

	h4 {
		font-size: 1em;
	}

	h5 {
		font-size: 0.83em;
	}

	h6 {
		font-size: 0.67em;
	}

	ul {
		margin-bottom: 0.5em;

		li {
			list-style: disc;
			margin-left: 1rem;
		}
	}
}
</style>
