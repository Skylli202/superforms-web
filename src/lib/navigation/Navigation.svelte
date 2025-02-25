<script lang="ts">
  import { onMount } from 'svelte';
  import { drawerStore } from '@skeletonlabs/skeleton';
  import A from './A.svelte';
  import Title from './Title.svelte';
  import { page } from '$app/stores';
  import { concepts } from './sections';
  import skeleton from '$lib/assets/skeleton.svg?raw';
  import mdsvex from '$lib/assets/mdsvex.svg?raw';
  import SearchButton from '../../routes/SearchButton.svelte';

  function drawerClose(): void {
    drawerStore.close();
  }

  let nav: HTMLElement;

  onMount(() => {
    nav.addEventListener('click', (e) => {
      const target = e.target as HTMLElement;
      if (target && target.tagName == 'A') {
        drawerClose();
      }
    });
  });

  $: active = $page.url.pathname;
</script>

<nav class="list-nav p-4" bind:this={nav}>
  <SearchButton cls="md:hidden mx-4 mb-3" width="w-40" />
  <ul>
    <li><A href="/">Home</A></li>
    <li><A href="/get-started">Get started</A></li>
    <li><A href="/api">API</A></li>
    <li><A href="/sponsors">Sponsors</A></li>

    <Title id="version-1">Version 1.0</Title>

    <li><A href="/whats-new-v1">What's new?</A></li>
    <li><A href="/migration">Migration guide</A></li>
    <li>
      <A href="https://superforms-legacy.vercel.app/" target="_blank"
        >0.x documentation</A>
    </li>

    <Title id="concepts">Concepts</Title>

    {#each concepts as concept}
      <li><A href={concept[0]}>{concept[1]}</A></li>
    {/each}

    <Title id="concepts">Guides</Title>

    <li><A href="/components">Componentization</A></li>
    <li><A href="/crud">CRUD tutorial</A></li>
    <li><A href="/default-values">Default values</A></li>
    <li><A href="/super-debug">SuperDebug</A></li>

    <Title id="concepts">Other topics</Title>

    <li><A href="/contributing">Contribute</A></li>
    <li><A href="/faq">FAQ</A></li>
    <li><A href="/flash-messages">Flash messages</A></li>
    <li><A href="/playground">Playground</A></li>
    <li><A href="/rate-limiting">Rate limiting</A></li>
  </ul>

  <div class="p-4 text-gray-600">
    <hr class="mt-5 mb-2" />
    Built with<br />
    <a
      href="https://www.skeleton.dev/"
      target="_blank"
      class="mt-1 !p-0 text-gray-400 hover:text-gray-300 hover:!bg-transparent"
      >{@html skeleton}</a>
    <a
      href="https://mdsvex.com/"
      target="_blank"
      class="w-20 mt-0 !p-0 text-gray-400 hover:text-gray-300 hover:!bg-transparent"
      >{@html mdsvex}</a>
  </div>
</nav>
