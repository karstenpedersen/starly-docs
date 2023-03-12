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
	import rehypeKatex from 'rehype-katex';
	import rehypeStringify from 'rehype-stringify';
	import remarkGfm from 'remark-gfm';
	import remarkMath from 'remark-math';
	import remarkParse from 'remark-parse';
	import remarkRehype from 'remark-rehype';
	import { onMount } from 'svelte';
	import { unified } from 'unified';
	import { reporter } from 'vfile-reporter';

	import { javascript } from '@codemirror/lang-javascript';
	import CodeMirror from 'svelte-codemirror-editor';

	let directoryPath = '/home/karsten/Documents/StarlyDocProject';
	let files: FileEntry[] = [];
	let selectedFile: FileEntry | null = null;
	let fileContent = '';
	let markdown = '';
	let parseMessage = '';

	let recentProjects: string[] = ['Hello'];

	let createFileName = '';

	onMount(() => {
		if (directoryPath !== '') {
			openProject(directoryPath);
		}
	});

	const openDirectory = async () => {
		directoryPath = ((await open({ directory: true })) as string) ?? '';
		recentProjects.push(directoryPath);
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
		if (selectedFile === null) return;

		fileContent = await readTextFile(selectedFile.path);
	};

	const createFile = async () => {
		if (createFileName === '') return;

		const filePath = await resolve(directoryPath, createFileName);
		await writeFile(filePath, '');
	};

	const saveFile = async () => {
		if (selectedFile === null) return;

		await writeFile(selectedFile.path, fileContent);
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

	const openProject = async (projectPath: string) => {
		directoryPath = projectPath;
		await getFiles();
	};

	const toMarkdown = async () => {
		const file = await unified()
			.use(remarkParse)
			.use(remarkGfm)
			.use(remarkMath)
			.use(remarkRehype, { allowDangerousHtml: true })
			.use(rehypeKatex)
			.use(rehypeStringify)
			.process(fileContent);

		console.log(String(file));
		parseMessage = reporter(file);
		markdown = String(file);
	};

	$: {
		fileContent;
		toMarkdown();
	}
</script>

<svelte:window on:keydown={handleKeyDown} on:keyup={handleKeyUp} />

{#if directoryPath !== ''}
	<div class="test">
		<header class="bg-zinc-800 text-white flex items-center justify-between">
			<p>{directoryPath}</p>

			<button
				on:click={() => {
					directoryPath = '';
				}}>Exit Project</button
			>
		</header>

		<div class="content">
			<aside class="bg-zinc-900 text-white p-3">
				<ul>
					{#each files as file}
						<li>
							<button
								on:click={() => {
									selectedFile = file;
									readFile();
								}}
								>{file.name}
							</button>
						</li>
					{/each}

					<li>
						<form
							on:submit={() => {
								createFile();
							}}
						>
							<input
								type="text"
								bind:value={createFileName}
								placeholder="File name"
								class="w-full text-zinc-900"
							/>
						</form>
					</li>
				</ul>
			</aside>

			<main class="h-full">
				<div class="p-3 flex items-center justify-between">
					<h1 class="text-xl">{selectedFile?.name}</h1>
					<p>Settings</p>
				</div>

				<div class="grid md:grid-cols-2 gap-1 h-full overflow-hidden">
					<CodeMirror bind:value={fileContent} lang={javascript()} />
					<div class="p-3 prose prose-zinc bg-white overflow-scroll">
						{@html markdown}
					</div>
				</div>
			</main>
		</div>

		<footer class="bg-zinc-900 text-white">
			<p>{parseMessage}</p>
		</footer>
	</div>
{:else}
	<section class="grid gap-4 place-content-center h-screen">
		<h1>Starly Docs</h1>

		<div>
			<button on:click={openDirectory}>Open Project</button>
			<button on:click={createDirectory}>Create Project</button>
		</div>

		{#if recentProjects.length > 0}
			<div>
				<h2>Recent Projects</h2>
				<ul class="flex flex-col gap-1">
					{#each recentProjects as project}
						<button
							on:click={() => {
								openProject(project);
							}}
							class="text-left">{project}</button
						>
					{/each}
				</ul>
			</div>
		{/if}
	</section>
{/if}

<style>
	.test {
		height: 100dvh;
		display: grid;
		grid-template-rows: auto 1fr auto;
	}

	.content {
		height: 100%;
		display: grid;
		grid-template-columns: 150px 1fr;
	}
</style>
