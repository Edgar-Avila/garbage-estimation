<script>
  import { dataset } from "$lib/dataset_small.js";
</script>

<div class="container py-12 px-4 mx-auto space-y-8">
  <h1 class="h1">Acerca de</h1>

  <p>
    Trabajo final del curso de Inteligencia Computacional de la Universidad
    Nacional del Altiplano Puno.
  </p>

  <p>
    Puedes encontrar el codigo de la API
    <a href="https://github.com/Edgar-Avila/garbage-estimation-api">Aquí</a>
  </p>

  <h2 class="h2">API</h2>
  <p>
    Puedes hacer uso de la API para obtener la estimación de residuos
    <a
      href="https://garbage-estimation-api.onrender.com/calc-residuos?rural=20000&urbana=10000"
      class="underline text-primary-500"
    >
      Aquí
    </a>
  </p>

  <h2 class="h2">Dataset</h2>

  <p>
    Dataset obtenido de la Plataforma Nacional de Datos Abiertos del Perú:
    <a
      href="https://www.datosabiertos.gob.pe/dataset/residuos-municipales-generados-anualmente"
      class="underline text-primary-500">Dataset</a
    >
  </p>

  <div class="table-container">
    <table class="table table-hover">
      <thead>
        <tr>
          <td class="p-2 text-center font-bold">N°</td>
          <td class="p-2 text-center font-bold">Departamento</td>
          <td class="p-2 text-center font-bold">Provincia</td>
          <td class="p-2 text-center font-bold">Distrito</td>
          <td class="p-2 text-center font-bold">Población Total</td>
          <td class="p-2 text-center font-bold">Población Urbana</td>
          <td class="p-2 text-center font-bold">Población Rural</td>
          <td class="p-2 text-center font-bold">Residuos Domésticos</td>
          <td class="p-2 text-center font-bold">Residuos No Domésticos</td>
          <td class="p-2 text-center font-bold">Residuos Municipales</td>
        </tr>
      </thead>
      <tbody>
        {#each dataset as row, i}
          <tr>
            <td>{i + 1}</td>
            <td>{row.DEPARTAMENTO}</td>
            <td>{row.PROVINCIA}</td>
            <td>{row.DISTRITO}</td>
            <td>{row.POB_TOTAL}</td>
            <td>{row.POB_URBANA}</td>
            <td>{row.POB_RURAL}</td>
            <td>{row.QRESIDUOS_DOM}</td>
            <td>{row.QRESIDUOS_NO_DOM}</td>
            <td>{row.QRESIDUOS_MUN}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
</div>
