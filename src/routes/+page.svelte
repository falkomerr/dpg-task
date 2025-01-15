<script lang="ts">
	/**
	 * Dependencencies
	 */
	import { fade } from 'svelte/transition';
	import { getPosts } from '$lib/api/methods/posts';

	/**
	 * Components
	 */
	import Loader from '$lib/components/Loader.svelte';
	import { toast, Toaster } from 'svelte-sonner';

	const loadData = async () => {
		const response = await getPosts();
		// Получаем посты из локального хранилища и парсим
		const savedPosts = localStorage.getItem('posts');
		const parsedPosts = (savedPosts && JSON.parse(savedPosts)) as
			| Awaited<ReturnType<typeof getPosts>>
			| false;

		// Если запрос неудачный, но в локальном хранилище есть посты - передаем их
		if (!response.success && parsedPosts && parsedPosts.success) {
			// Смотрим, есть ли посты в локальном хранилище
			if (parsedPosts.data.length > 0) {
				return parsedPosts.data;
			} else {
				toastError();
				throw new Error('Нет данных для отображения.');
			}
		}

		// Если запрос неудачный и нет удачного сохраненного запроса - отображаем ошибку
		if (!response.success && (!parsedPosts || !parsedPosts.success)) {
			toastError();
			throw new Error('Нет данных для отображения.');
		}

		localStorage.setItem('posts', JSON.stringify(response));

		// Все кейсы обработаны, это для тайпскрипта
		if (!response.success) return [];

		// Запрос удачный, возвращаем(смотрим, есть ли посты)
		if (response.data.length > 0) {
			return response.data;
		} else {
			toastError();
			throw new Error('Нет данных для отображения.');
		}
	};

	function toastError(message?: string) {
		toast.error(message ?? 'Ошибка');
	}
</script>

<div class="layout">
	<h1>Latest posts</h1>

	{#await loadData()}
		<Loader />
	{:then posts}
		<div class="posts">
			{#each posts as post (post.id)}
				<div class="posts__item" in:fade>
					<h4>{post.title}</h4>
					<p>{post.body}</p>
				</div>
			{/each}
		</div>
	{:catch e}
		<h1>{e}</h1>
	{/await}
</div>

<Toaster position="top-right" />

<style lang="scss">
	.layout {
		padding: 25px 50px;

		display: flex;
		flex-direction: column;

		justify-content: center;
		align-items: center;

		gap: 50px;

		width: 100%;
		height: max-content;

		h1 {
			text-align: center;
		}
	}

	.posts {
		display: flex;
		flex-direction: column;
		gap: 5px;
		width: 500px;

		&__item {
			padding: 10px 15px;
			display: flex;
			flex-direction: column;
			gap: 15px;

			height: 150px;
			width: 100%;

			border-radius: 8px;

			background: rgba(0, 0, 0, 0.08);

			h4 {
				color: #2b302c;
				overflow: hidden;
				text-overflow: ellipsis;
				white-space: nowrap;
			}

			p {
				overflow: hidden;
				display: -webkit-box;
				-webkit-line-clamp: 2; /* number of lines to show */
				line-clamp: 2;
				-webkit-box-orient: vertical;

				color: rgba(43, 48, 44, 0.7);
			}
		}
	}
</style>
