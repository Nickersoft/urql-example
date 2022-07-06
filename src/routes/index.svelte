<script lang="ts" context="module">
  import { gql } from '@urql/svelte';
  import { createClient, dedupExchange, fetchExchange } from '@urql/core';
  import type { Load } from '@sveltejs/kit';
  import { cacheExchange } from '@urql/exchange-graphcache';

  export const load: Load = async ({ fetch }) => {
    const client = createClient({
      url: 'https://swapi-graphql.netlify.app/.netlify/functions/index',
      fetch: fetch as typeof window.fetch,
      fetchOptions: {
        credentials: 'omit'
      },
      exchanges: [dedupExchange, cacheExchange(), fetchExchange]
    });

    const result = await client
      .query(
        gql`
          query {
            allFilms {
              films {
                id
                title
              }
            }
          }
        `
      )
      .toPromise();

    if (result.error) {
      throw new Error(result.error.message);
    }

    return {
      props: {
        films: result?.data?.allFilms ?? [],
        client
      }
    };
  };
</script>

<script lang="ts">
  export let films: any[] = [];
</script>

<ol>
  {#each films as film}
    <li>{film.title}</li>
  {/each}
</ol>
