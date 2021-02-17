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

  $: if ($activeRoute && $activeRoute.route === route) {
    let params = $activeRoute.params;
    if (params['*']) {
      delete params['*'];
    }
    routeParams = params;
  }

  $: {
    const { path, component, cache, ...rest } = $$props;
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
    <svelte:component this="{component}" location={$location} {...routeParams} {...routeProps} />
  {:else}
    <slot params="{routeParams}" location={$location}></slot>
  {/if}
{/if}
