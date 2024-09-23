<script context="module">
	let JSZip, saveFile
	import('jszip').then((module) => (JSZip = module.default))
	import('file-saver').then((module) => (saveFile = module.saveAs))
</script>

<script>
	import axios from 'axios'
	import Icon from '@iconify/svelte'
	import _ from 'lodash-es'
	import { fade } from 'svelte/transition'
	import { format } from 'timeago.js'
	import TextInput from '$lib/ui/TextInput.svelte'
	import Select from '$lib/ui/inputs/Select.svelte'
	import modal from '$lib/stores/app/modal'
	import Spinner from '$lib/ui/Spinner.svelte'
	import DropdownButton from '$lib/ui/DropdownButton.svelte'
	import site, { active_deployment } from '$lib/stores/data/site'
	import pages from '$lib/stores/data/pages'
	import symbols from '$lib/stores/data/symbols'
	import { push_site, build_site_bundle } from './Deploy'
	import PrimaryButton from '$lib/ui/PrimaryButton.svelte'
	import { dataChanged } from '$lib/database'

	async function download_site() {
		downloading = true
		const files = await build_site_bundle({ pages: $pages, symbols: $symbols })
		if (!files) {
			downloading = false
			return
		}
		
		/////////////////////////////

		const bundle = await create_site_zip(files)
		saveFile(bundle, `${$site.name}.zip`)
		modal.hide()

		async function create_site_zip(files) {
			const zip = new JSZip()
			files.forEach(({ path, content }) => {
				zip.file(path, content)
			})
			return await zip.generateAsync({ type: 'blob' })
		}
		downloading = false;
	}

	async function publish_site() {
		publishing = true
		const files = await build_site_bundle({ pages: $pages, symbols: $symbols })
		if (!files) {
			publishing = false
			return
		}
		
		console.log(files);
		console.log($site);
		await fetch('/push', {
			method: "POST",
			headers: {
				"Content-Type": "application/json",
			},
			body: JSON.stringify({
				site: $site,
				files
			})
		});
		publishing = false;
		return;
	}


	let downloading = false;
	let publishing = false;
</script>

<div class="Deploy primo-reset">
	<header>
		<h2>
			<Icon icon="ic:sharp-publish" />
			<span>Deploy Site</span>
		</h2>
		<button on:click={() => modal.hide()}>
			<Icon icon="ic:baseline-close" />
		</button>
	</header>
	<div class="container">
		<p class="description">
			To publish your website, you'll need to upload it to a web host. You can either <a
				href="https://docs.primocms.org/publishing"
			>
				manually upload
			</a>
			your site, or
			<a href="https://docs.primocms.org/publishing">deploy it from a Github repo.</a>
		</p>
		<div class="buttons">
			<button class="primo-button" on:click={download_site}>
				<Icon icon={downloading ? 'eos-icons:loading' : 'ic:baseline-download'} />
				<span>Download</span>
			</button>

			<button class="primo-button" on:click={publish_site}>
				<Icon icon={publishing ? 'eos-icons:loading' : 'ic:baseline-publish'} />
				<span>Publish</span>
			</button>
		</div>
	</div>
</div>

<style lang="postcss">
	.Deploy.primo-reset {
		background: var(--primo-color-black);
		padding: 1.125rem 1.25rem;
		display: grid;
		gap: 1rem;
	}
	header {
		display: flex;
		align-items: center;
		justify-content: space-between;

		h2 {
			display: flex;
			align-items: center;
			gap: 0.5rem;
			font-weight: 500;
		}
	}
	.container {
		display: grid;
		gap: 1rem;

		a {
			text-decoration: underline;
		}
	}
	.account-card {
		display: flex;
		justify-content: space-between;
		padding: 0.75rem 0.5rem;
		border-radius: 0.25rem;
		border: 0.8px solid #6e6e6e;

		.user {
			font-weight: 400;
			font-size: 0.75rem;
			line-height: 1.125rem;
			color: #cecece;
			display: flex;
			align-items: center;
			gap: 0.5rem;

			img {
				width: 2rem;
				aspect-ratio: 1 / 1;
				border-radius: 50%;
			}
		}
	}

	.description {
		a {
			text-decoration: underline;
		}
	}

	.primo-link {
		text-align: left;
		font-size: 0.875rem;
		color: var(--color-gray-3);
		text-decoration: underline;
	}

	.create-repo {
		header {
			display: flex;
			align-items: center;
			button {
				font-size: 0.75rem;
				color: #9d9d9d;
				text-decoration: underline;
			}
		}
		.form-label {
			font-size: 12px;
			color: #b6b6b6;
			margin-bottom: 0.125rem;
		}
		footer {
			font-size: 0.625rem;
			color: #858585;
			margin-top: 0.25rem;
		}
	}

	.repo-card {
		display: flex;

		div {
			flex: 1;
			display: grid;
			justify-content: flex-start;
		}

		.name {
			text-decoration: underline;
		}

		.last-updated {
			font-size: 0.75rem;
			color: #b6b6b6;
		}
	}

	.buttons {
		display: flex;
		justify-content: flex-end;
		align-items: center;
		gap: 1rem;
	}
	.primo-button {
		display: flex;
		align-items: center;
		gap: 0.25rem;
		padding: 7px 16px;
		background: #1f1f1f;
		border-radius: 0.25rem;
		height: 100%;
	}
	.primo-button.primary {
		border: 1px solid #35d994;
		background: transparent;
	}
	form {
		margin-top: 0.5rem;

		div {
			display: grid;
			gap: 0.5rem;
			grid-template-columns: 1fr auto;
			gap: 0.5rem;
		}
	}
	:global(form > label) {
		flex: 1;
	}
</style>
