<script>
  import deps from "$lib/deps.json";
  import countries from "$lib/countries.json";
  import { onMount } from "svelte";
  import { getToastStore } from "@skeletonlabs/skeleton";
  import Icon from "@iconify/svelte";

  const predictionType = Object.freeze({
    DEPARTAMENTO: "departamento",
    PAIS: "pais",
  });

  let fetching = false;
  const toast = getToastStore();
  let year = 2023;
  let departamento = undefined;
  let paisName = undefined;
  let paisCode = undefined;
  let result = undefined;
  let by = predictionType.DEPARTAMENTO;
  let map = undefined;
  let style = { fillOpacity: 0.25 };
  let depsLayer = undefined;
  let countriesLayer = undefined;

  onMount(async () => {
    map = L.map("map").setView([-9.800678, -74.900665], 5.5);
    L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution:
        '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
    }).addTo(map);
    map.on("click", (e) => {
      if (fetching) {
        toast.trigger({
          message: "Ya se está calculando una estimación",
          type: "error",
        });
        return;
      }
      const lnglat = [e.latlng.lng, e.latlng.lat];
      if (by === predictionType.PAIS) {
        const results = leafletPip.pointInLayer(lnglat, countriesLayer);
        if (results.length === 0) {
          toast.trigger({
            message: "No se encontró el país",
            type: "error",
          });
          return;
        }
        const nomPais = results[0].feature.properties.ADMIN;
        const codPais = results[0].feature.properties.ISO_A3;
        paisCode = codPais;
        paisName = nomPais;
      } else {
        const results = leafletPip.pointInLayer(lnglat, depsLayer);
        if (results.length === 0) {
          toast.trigger({
            message: "No se encontró el departamento",
            type: "error",
          });
          return;
        }
        const nomDep = results[0].feature.properties.NOMBDEP;
        departamento = nomDep;
      }
    });
    addDepsLayer();
  });

  const addDepsLayer = () => {
    if (countriesLayer) {
      map.removeLayer(countriesLayer);
      countriesLayer = undefined;
    }
    if (depsLayer) return;
    depsLayer = L.geoJSON(deps, { style }).addTo(map);
    depsLayer.eachLayer((layer) => {
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
  };

  const addCountriesLayer = () => {
    if (depsLayer) {
      map.removeLayer(depsLayer);
      depsLayer = undefined;
    }
    if (countriesLayer) return;
    countriesLayer = L.geoJSON(countries, { style }).addTo(map);
    countriesLayer.eachLayer((layer) => {
      layer.bindTooltip(layer.feature.properties.ADMIN);
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
  };

  const changePredictionType = () => {
    if (by === predictionType.DEPARTAMENTO) {
      by = predictionType.PAIS;
      addCountriesLayer();
    } else {
      by = predictionType.DEPARTAMENTO;
      addDepsLayer();
    }
    result = undefined;
  };

  const predictPopulation = async () => {
    try {
      fetching = true;
      if ((by === predictionType.DEPARTAMENTO) && (departamento === undefined)) {
        toast.trigger({
          message: "No se ha seleccionado un departamento",
          type: "error",
        });
        return;
      }

      if ((by === predictionType.PAIS) && (paisCode === undefined)) {
        toast.trigger({
          message: "No se ha seleccionado un país",
          type: "error",
        });
        return;
      }

      let url = `https://garbage-estimation-api.onrender.com/calc-poblacion?departamento=${departamento}&anio=${year}`;
      if(by === predictionType.PAIS) {
        url = `https://garbage-estimation-api.onrender.com/calc-poblacion/paises?pais=${paisCode}&anio=${year}`;
      }
      const response = await fetch(url);
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
    <h4 class="h4">Parámetros</h4>
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
    {#if by === predictionType.PAIS}
      <label class="label">
        <b>País:</b>
        <input
          type="text"
          bind:value={paisName}
          class="p-0 m-0 border-none leading-none"
          readonly
        />
      </label>
    {:else}
      <label class="label">
        <b>Departamento:</b>
        <input
          type="text"
          bind:value={departamento}
          class="p-0 m-0 border-none leading-none"
          readonly
        />
      </label>
    {/if}
    <button
      class="btn variant-filled-primary btn-sm my-2"
      on:click={predictPopulation}
    >
      Predecir
    </button>
    <button
      class="btn variant-filled-secondary btn-sm my-2"
      on:click={changePredictionType}
    >
      Por {by === predictionType.DEPARTAMENTO ? "país" : "departamento"}
    </button>
    <hr class="mb-2" />
    {#if fetching}
      <Icon icon="line-md:loading-loop" height={30} class="inline"/>
    {/if}
    {#if result}
      <h4 class="h4">Parámetros</h4>
      <b>Población rural: </b>
      {result.rural} <br />
      <b>Población urbana: </b>
      {result.urbana} <br />
      <b>Residuos (Toneladas): </b>
      {result.residuos.toFixed(3)} <br />
      <hr class="mb-2" />
      <h4 class="h4">Confianza del resultado:</h4>
      <b>Población rural</b>
      {(result.score_rural * 100).toFixed(2)} %<br />
      <b>Población urbana</b>
      {(result.score_urbana * 100).toFixed(2)} %<br />
      <b>Residuos:</b>
      {(result.score_total * 100).toFixed(2)} %<br />

    {/if}
  </div>
</div>
