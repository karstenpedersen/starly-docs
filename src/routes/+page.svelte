<script lang="ts">
	import { open } from '@tauri-apps/api/dialog';
	import { createDir, exists, readDir, readTextFile, type FileEntry } from '@tauri-apps/api/fs';
	// Check if the `$APPDATA/avatar.png` file exists

	let directoryPath = '';
	let files: FileEntry[] = [];
	let selectedFile = '';
	let fileContent = '';

	const openDirectory = async () => {
		directoryPath = ((await open({ directory: true })) as string) ?? '';
		await getFiles();
	};

	const createDirectory = async () => {
		directoryPath = '';
		await createDir(directoryPath);
		await getFiles();
	};

	const getFiles = async () => {
		if (directoryPath === '' || !exists(directoryPath)) return;

		files = await readDir(directoryPath);
	};

	const readFile = async () => {
		fileContent = await readTextFile(selectedFile);
	};
</script>

{#if directoryPath !== ''}
	<p>{directoryPath}</p>
	<p>{files.length}</p>

	<ul>
		{#each files as file}
			<li>
				<button
					on:click={() => {
						selectedFile = file.path;
						readFile();
					}}
					>{file.name}
				</button>
			</li>
		{/each}
	</ul>

	<div>
		{fileContent}
	</div>

	<button
		on:click={() => {
			directoryPath = '';
		}}>Exit Project</button
	>
{:else}
	<section>
		<h1>Starly Docs</h1>

		<div>
			<button on:click={openDirectory}>Open Project</button>
			<button on:click={createDirectory}>Create Project</button>
		</div>
	</section>
{/if}

<style>
</style>
