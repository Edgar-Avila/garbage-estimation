<script>
  import "../app.postcss";
  import {
    AppShell,
    AppBar,
    Toast,
    initializeStores,
  } from "@skeletonlabs/skeleton";
  initializeStores();
</script>

<!-- App Shell -->
<AppShell class="min-h-screen">
  <svelte:fragment slot="header">
    <!-- App Bar -->
    <AppBar>
      <svelte:fragment slot="lead">
        <strong class="text-xl uppercase"
          ><a href="/">Estimación de Residuos</a></strong
        >
      </svelte:fragment>
      <svelte:fragment slot="trail">
        <a class="btn btn-sm variant-ghost-surface" href="/about">Acerca de</a>
        <a class="btn btn-sm variant-ghost-surface" href="/prediccion">Predicción</a>
        <a class="btn btn-sm variant-ghost-surface" href="/map">Mapa</a>
      </svelte:fragment>
    </AppBar>
  </svelte:fragment>
  <!-- Page Route Content -->
  <slot />
</AppShell>
<Toast />
