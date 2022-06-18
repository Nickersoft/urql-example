<script lang="ts">
	import { initContextClient, gql, queryStore, mutationStore } from '@urql/svelte';
	import { dedupExchange, fetchExchange } from '@urql/core';
	import { cacheExchange } from '@urql/exchange-graphcache';

	const client = initContextClient({
		url: 'https://swapi-graphql.netlify.app/.netlify/functions/index',
		fetchOptions: {
			credentials: 'omit'
		},
		exchanges: [
			dedupExchange,
			cacheExchange({
				resolvers: {
					Query: {
						testQuery(parent, args, cache, info) {
							cache.updateQuery(
								{
									query: gql`
										query {
											allFilms {
												films {
													id
													title
												}
											}
										}
									`
								},
								(data) => {
									data?.allFilms?.films?.push({
										__typename: 'Film',
										id: 'fakeID',
										title: 'Hello World'
									});

									return data;
								}
							);
							return true;
						}
					}
				}
			}),
			fetchExchange
		]
	});

	const result = queryStore({
		client,
		variables: {},
		query: gql`
			query {
				allFilms {
					films {
						id
						title
					}
				}
			}
		`
	});

	function mutate() {
		queryStore({
			client,
			variables: {},
			query: gql`
				query {
					testQuery
				}
			`
		});
	}
</script>

<ol>
	{#each $result.data?.allFilms?.films ?? [] as film}
		<li>{film.title}</li>
	{/each}
</ol>

<button on:click={mutate}>Add New</button>
