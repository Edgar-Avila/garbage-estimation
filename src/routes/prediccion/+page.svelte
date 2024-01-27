<script>
  import { getToastStore } from "@skeletonlabs/skeleton";
  import Icon from "@iconify/svelte";

  let rural = 25000;
  let urbana = 25000;
  let result = undefined;
  let promise = undefined;

  const toast = getToastStore();

  const calcGarbage = async () => {
    try {
      const response = await fetch(
        `https://garbage-estimation-api.onrender.com/calc-residuos?rural=${rural}&urbana=${urbana}`,
      );
      const data = await response.json();
      result = data.prediccion;
    } catch (error) {
      console.error(error);
      toast.trigger({
        message: "No se pudo calcular la estimación",
        type: "error",
      });
    }
  };
</script>

<div class="container py-12 px-4 mx-auto space-y-4">
  <h1 class="h1">Estimación</h1>
  <label>
    <span>Urbana: {urbana}</span>
    <input type="range" min={0} max={50000} step={1} bind:value={urbana} />
  </label>
  <label>
    <span>Rural: {rural}</span>
    <input type="range" min={0} max={50000} step={1} bind:value={rural} />
  </label>

  <button
    class="btn variant-filled-primary"
    on:click={() => (promise = calcGarbage())}
  >
    Calcular
  </button>

  {#await promise}
    <div class="text-center justify-center items-center flex gap-1">
      <Icon icon="line-md:loading-loop" height={30} class="inline"/>
      Calculando...
    </div>
  {:then}
    {#if result !== undefined}
      <p class="text-center">
        Se generarán aproximadamente
        <span class="badge variant-filled">{result.toFixed(3)}</span>
        toneladas de residuos.
      </p>
    {/if}
  {/await}
</div>
