<script lang="ts">
	import { open } from '@tauri-apps/api/dialog';
	import {
		createDir,
		exists,
		readDir,
		readTextFile,
		writeFile,
		type FileEntry
	} from '@tauri-apps/api/fs';
	import { resolve } from '@tauri-apps/api/path';

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

	const createFile = async () => {
		const fileName = 'tesst.md';
		const filePath = await resolve(directoryPath, fileName);
		await writeFile(filePath, '');
		console.log('Hello');
	};

	const saveFile = async () => {
		await writeFile(selectedFile, fileContent);
	};

	let activeKeys: { [key: string]: boolean } = {};

	const handleKeyDown = (e: KeyboardEvent) => {
		activeKeys[e.key] = true;

		if (activeKeys['Control'] && activeKeys['s']) {
			saveFile();
		}
	};

	const handleKeyUp = (e: KeyboardEvent) => {
		delete activeKeys[e.key];
	};
</script>

<svelte:window on:keydown={handleKeyDown} on:keyup={handleKeyUp} />

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

		<li>
			<button
				on:click={() => {
					createFile();
				}}>+</button
			>
		</li>
	</ul>

	<div contenteditable="true" bind:innerHTML={fileContent} />

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
