<script>
  import deps from "$lib/deps.json";
  import { onMount } from "svelte";
  import { getToastStore } from "@skeletonlabs/skeleton";

  let fetching = false;
  const toast = getToastStore();
  let year = 2023;
  let departamento = undefined;
  let result = undefined;

  onMount(async () => {
    const map = L.map("map").setView([-9.800678, -74.900665], 5.5);
    L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution:
        '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
    }).addTo(map);
    let style = {
      fillOpacity: 0.25,
    };
    let geojsonLayer = L.geoJSON(deps, { style }).addTo(map);
    geojsonLayer.eachLayer((layer) => {
      layer.bindTooltip(layer.feature.properties.NOMBDEP);
      layer.on("mouseover", function (e) {
        e.target.setStyle({
          fillOpacity: 0.4,
        });
      });
      layer.on("mouseout", function (e) {
        e.target.setStyle({
          fillOpacity: 0.25,
        });
      });
    });
    map.on("click", (e) => {
      if (fetching) {
        toast.trigger({
          message: "Ya se está calculando una estimación",
          type: "error",
        });
        return;
      }
      const lnglat = [e.latlng.lng, e.latlng.lat];
      const results = leafletPip.pointInLayer(lnglat, geojsonLayer);
      if (results.length === 0) {
        toast.trigger({
          message: "No se encontró el departamento",
          type: "error",
        });
        return;
      }
      const nomDep = results[0].feature.properties.NOMBDEP;
      departamento = nomDep;
    });
  });

  const predictPopulation = async () => {
    try {
      fetching = true;
      if (departamento === undefined) {
        toast.trigger({
          message: "No se ha seleccionado un departamento",
          type: "error",
        });
        return;
      }
      const response = await fetch(
        `https://garbage-estimation-api.onrender.com/calc-poblacion?departamento=${departamento}&anio=${year}`,
      );
      result = await response.json();
      if (result.error) {
        toast.trigger({
          message: result.error,
          type: "error",
        });
        result = undefined;
      }
    } catch (error) {
      console.error(error);
      toast.trigger({
        message: "No se pudo calcular la estimación",
        type: "error",
      });
    } finally {
      fetching = false;
    }
  };
</script>

<svelte:head>
  <link
    rel="stylesheet"
    href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
    integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY="
    crossorigin=""
  />
  <script
    src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"
    integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo="
    crossorigin=""
  ></script>
  <script
    src="https://unpkg.com/@mapbox/leaflet-pip@latest/leaflet-pip.js"
  ></script>
</svelte:head>

<div class="relative">
  <div id="map" class="h-screen"></div>
  <div
    class="border border-black p-4 bg-white absolute right-2 top-2 z-[50000]"
  >
    <label class="label">
      <b>Año a predecir:</b>
      <input
        type="number"
        min={2014}
        step={1}
        max={2028}
        bind:value={year}
        class="p-0 m-0 border-none leading-none"
      />
    </label>
    <label class="label">
      <b>Departamento:</b>
      <input
        type="text"
        bind:value={departamento}
        class="p-0 m-0 border-none leading-none"
        readonly
      />
    </label>
    <button
      class="btn variant-filled-primary btn-sm my-2"
      on:click={predictPopulation}
    >
      Predecir
    </button>
    <hr  class="mb-2"/>
    {#if result}
        <b>Población rural: </b> {result.rural} <br />
        <b>Población urbana: </b> {result.urbana} <br />
        <b>Residuos (Toneladas): </b> {result.residuos.toFixed(3)} <br />
    {/if}
    
  </div>
</div>
