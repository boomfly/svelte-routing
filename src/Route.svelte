<script>
  import { getContext, onDestroy } from "svelte";
  import { ROUTER, LOCATION } from "./contexts.js";

  export let path = "";
  export let component = null;
  export let cache = false;

  const { registerRoute, unregisterRoute, activeRoute } = getContext(ROUTER);
  const location = getContext(LOCATION);

  const route = {
    path,
    // If no path prop is given, this Route will act as the default Route
    // that is rendered if no other Route in the Router is a match.
    default: path === ""
  };
  let routeParams = {};
  let routeProps = {};
  let isActive = false;

  $: if ($activeRoute && $activeRoute.route === route) {
    routeParams = $activeRoute.params;
    isActive = true;
    console.log('isActive', isActive, cache, path)
  } else {
    isActive = false;
    console.log('isActive', isActive, cache, path)
  }

  $: {
    const { path, component, ...rest } = $$props;
    routeProps = rest;
  }

  registerRoute(route);

  // There is no need to unregister Routes in SSR since it will all be
  // thrown away anyway.
  if (typeof window !== "undefined") {
    onDestroy(() => {
      unregisterRoute(route);
    });
  }
</script>

{#if ($activeRoute !== null && $activeRoute.route === route) || cache}
  {#if component !== null}
    <svelte:component this="{component}" location={$location} {...routeParams} {...routeProps} {isActive}  />
  {:else}
    <slot params="{routeParams}" location={$location} {isActive}></slot>
  {/if}
{/if}
