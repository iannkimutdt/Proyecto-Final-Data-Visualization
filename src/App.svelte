<script>
  import L from 'leaflet';
  import { onMount } from 'svelte';
  import siniestros from "/src/data/siniestros_hechos.csv";
  
  /******************* Mapa Integral Interactivo **********************/
  let map;
  let pointsLayer = [];

  let filterValues = {
      comuna: null,
      mes: null,
      dia: null,
      hora: null,
      causante: null,
      afectado: null
  };

  onMount(() => {
    map = L.map('map', {
        center: [-34.61915, -58.4265],
        zoom: 12,
        minZoom: 11,
        scrollWheelZoom: false,
        zoomControl: false,
        attributionControl: false
    }); 

    L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
        attribution: '© OpenStreetMap'
    }).addTo(map);

    const zoomMessageControl = L.control({ position: 'bottomright' });

    zoomMessageControl.onAdd = () => {
      const div = L.DomUtil.create('div', 'zoom-message');
      div.innerHTML = `
        <div style="
          background: rgba(0, 0, 0, 0.6);
          color: white;
          padding: 8px 12px;
          border-radius: 5px;
          font-size: 14px;
          font-family: Arial, sans-serif;
          text-align: center;
        ">
          <b>Zoom</b> Ctrl + Scroll
        </div>
      `;
      return div;
    };

    zoomMessageControl.addTo(map);

    map.getContainer().addEventListener('wheel', (event) => {
      if (event.ctrlKey) {
        event.preventDefault();
      if (event.deltaY > 0) {
        map.zoomOut();
      } else {
        map.zoomIn();
      }
    }
    });

    addPoints();
    loadFlourishScrolly();
  });


  const addPoints = () => {
    pointsLayer.forEach(layer => map.removeLayer(layer));
    pointsLayer = [];

    const filteredPoints = siniestros.filter(item => {
      const horaMatch = filterValues.hora === null || item.hora.toString() === filterValues.hora.toString();
      return (!filterValues.comuna || parseInt(item.comuna) === filterValues.comuna) &&
        (!filterValues.mes || item.mes === filterValues.mes) &&
        (!filterValues.dia || item.dia_semana === filterValues.dia) &&
        horaMatch &&
        (!filterValues.causante || item.causante === filterValues.causante) &&
        (!filterValues.afectado || item.afectado === filterValues.afectado);
  });

    filteredPoints.forEach((incident) => {
      const lat = parseFloat(incident.latitud);
      const lng = parseFloat(incident.longitud);
      
      // Crear el contenido del popup con la información del incidente
      const popupContent = `
        <div class="popup-content">
          <h3><strong>Detalles del Siniestro</strong></h3>
          <p>Fecha: ${incident.fecha}</p>
          <p>Hora: ${incident.hora_detallada}</p>
          <p>Día: ${incident.dia_semana}</p>
          <p>Dirección: ${incident.direccion}</p>
          <p>Comuna: ${incident.comuna}</p>
          <p>Causante: ${incident.causante}</p>
          <p>Afectado: ${incident.afectado}</p>
          ${incident.n_victimas > 0 ? `<p>Número de víctimas: ${incident.n_victimas}</p>` : ''}
        </div>
        `;

      const point = L.circle([lat, lng], {
        color: 'red',
        fillColor: 'red',
        fillOpacity: 0.5,
        radius: 4
      })
      .bindPopup(popupContent)
      .addTo(map);
      
      pointsLayer.push(point);
    });
  };

  const handleFilterChange = (event) => {
    const { name, value } = event.target;
    const filterName = name.replace('filtro_', '').toLowerCase();
    
    if (filterName === 'hora') {
      filterValues[filterName] = value === "" ? null : parseInt(value, 10);
    } else if (filterName === 'comuna') {
      filterValues[filterName] = value === "" ? null : parseInt(value, 10);
    } else {
      filterValues[filterName] = value || null;
    }

    addPoints();
  };
  


  /****************** Scrolly Flourish *********************/
  function loadFlourishScrolly() {
    const script = document.createElement('script')
    script.src = "https://cdn.flourish.rocks/flourish-scrolly-v3.1.0.min.js"
    script.type = "text/javascript"
    script.onload = () => initFlourishScrolly()
    document.body.appendChild(script)
  }

  const slides1 = [
    "En 2023, hubo <span style='color:red'>6799</span> accidentes.",
    "En los meses de vacaciones, ocurrieron menos siniestros.",
    "Durante los días hábiles, hubo una tendencia creciente. <br> Mientras que en los fines de semana, decreciente.",
    "En las madrugadas de los fines de semana, se ve un incremento de más del 100% comparado con el resto de los días."
  ]
  const slides2 = [
    "Hubo una tendencia similar durante los días hábiles.",
    "En horas diurnas, el número de siniestros fue mayor que el promedio. En horas nocturnas, fue menor.",
    "En los fines de semanana, ocurrió lo opuesto."
  ]
</script>

<main>
  <div class="header">
    <h1 style="color: red; font-weight: bold;">PUNTOS DE IMPACTO: CABA 2023</h1>
    <h4>¿Tenés miedo de chocar al manejar? <br>
      Este análisis te muestra tendencias, cuándo y dónde ocurrieron los accidentes durante 2023.
    </h4>
    <p>
      En la Ciudad Autónoma de Buenos Aires, el parque automotor en 2023 fue de 1.616.510, el 9,1% del total nacional. <br>
      Aunque no necesariamente la totalidad de esos vehículos circulan por las calles, se puede aproximar que esa cantidad, <br>
      o incluso aún más (por los vehículos que ingresan de la Provincia de Buenos Aires), transitan por la ciudad.
    </p>
    <p>Este elevado número hace que CABA sea una <b style="color: red;">zona de alto riesgo</b>.</p>
  </div>

  <!------------------------->
  <!-- Scrolly Flourish #1 -->
  <!------------------------->  
  <div id="scrolly_flourish_1">
    <div class="flourish-embed" data-src="story/2739344" data-url="https://flo.uri.sh/story/2739344/embed" data-height="100vh"></div>
    {#each slides1 as slide, index1}
      <h5>
        {@html slide}
        <!-- svelte-ignore a11y-missing-content -->
        <a href={"#story/2739344/slide-" + (index1 + 1)}></a>
      </h5>
    {/each}
  </div>


  <!----------------------------->
  <!-- Info/Datos Interesantes -->
  <!-----------------------------> 
    <div class="texto_container">
      <h4>
        <span style="color:red;">21,77</span> accidentes por cada 10.000 habitantes <br>
        <span style="color:red;">0,42%</span> de los vehículos del parque automotor tuvo un accidente <br>
      </h4>
    </div>
  

  <!------------------------->
  <!-- Scrolly Flourish #2 -->
  <!------------------------->  
  <div class="header">
    <h3 style="color: red; font-weight: bold;">EL FENÓMENO INVERSO</h3>
  </div>

  <div id="scrolly_flourish_2">
    <div class="flourish-embed" data-src="story/2752082" data-url="https://flo.uri.sh/story/2752082/embed" data-height="100vh"></div>
    {#each slides2 as slide, index2}
      <h5>
        {@html slide}
        <!-- svelte-ignore a11y-missing-content -->
        <a href={"#story/2752082/slide-" + (index2 + 1)} style="flex: 1; max-height: 40%;"></a>
      </h5>
    {/each}
  </div>


  <!----------------------------->
  <!-- Info/Datos Interesantes -->
  <!-----------------------------> 
  <div class="texto_container">
    <h4>
      Horas de <span style="color:red;">mayor riesgo</span> (en relación al promedio) <br>
      Días Hábiles: 06:00 a 21:00 <br>
      Fines de Semana: 20:00 a 06:00
    </h4>
  </div>

    
  <!----------------------->
  <!-- Flourish Heatmaps -->
  <!-----------------------> 
  <div class="header">
    <h3 style="color: red; font-weight: bold;">LAS ÁREAS CRÍTICAS DE LA CIUDAD</h3>
    <h5>La mayoría son intersecciones entre avenidas importantes y de gran flujo de vehículos.</h5>
  </div>
  
  <div style="display: flex; justify-content: center; gap: 20px;">
    <div style="flex: 1; max-width: 40%;">
      <div class="flourish-embed flourish-map" data-src="visualisation/20538016"><script src="https://public.flourish.studio/resources/embed.js"></script><noscript><img src="https://public.flourish.studio/visualisation/20538016/thumbnail" style="width: 100%;" alt="map visualization" /></noscript></div>
    </div>
    <div style="flex: 1; max-width: 40%;">
      <div class="flourish-embed flourish-map" data-src="visualisation/20176310"><script src="https://public.flourish.studio/resources/embed.js"></script><noscript><img src="https://public.flourish.studio/visualisation/20176310/thumbnail" style="width: 100%;" alt="map visualization" /></noscript></div>
    </div>
  </div>


  <!----------------------------->
  <!-- Info/Datos Interesantes -->
  <!-----------------------------> 
  <div class="texto_container">
    <h4>
      Zonas de <span style="color:red;">mayor concentración</span> <br>
    </h4>
    <h6>
      Microcentro <br>
      Intersección entre Palermo, Villa Crespo y Chacarita <br>
      Belgrano <br>
      Av. Rivadavia y Av. Nazca <br>
      Av. General Paz
    </h6>
  </div>


  <!------------------------------->
  <!-- Mapa Integral Interactivo -->
  <!------------------------------->  
  <div class="header">
    <h3 style="color: red; font-weight: bold;">EL MAPA DE SINIESTROS</h3>
  </div>

  <div id="map">
    <div class="filtros_container">
      <select class="filtro" name="filtro_Comuna" on:change="{handleFilterChange}">
          <option class="filtro_name" value="">Comuna</option>
          <option value=1>Comuna 1</option>
          <option value=2>Comuna 2</option>
          <option value=3>Comuna 3</option>
          <option value=4>Comuna 4</option>
          <option value=5>Comuna 5</option>
          <option value=6>Comuna 6</option>
          <option value=7>Comuna 7</option>
          <option value=8>Comuna 8</option>
          <option value=9>Comuna 9</option>
          <option value=10>Comuna 10</option>
          <option value=11>Comuna 11</option>
          <option value=12>Comuna 12</option>
          <option value=13>Comuna 13</option>
          <option value=14>Comuna 14</option>
          <option value=15>Comuna 15</option>
      </select>
      <select class="filtro" name="filtro_Mes" on:change="{handleFilterChange}">
          <option class="filtro_name" value="">Mes</option>
          <option value="Enero">Enero</option>
          <option value="Febrero">Febrero</option>
          <option value="Marzo">Marzo</option>
          <option value="Abril">Abril</option>
          <option value="Mayo">Mayo</option>
          <option value="Junio">Junio</option>
          <option value="Julio">Julio</option>
          <option value="Agosto">Agosto</option>
          <option value="Septiembre">Septiembre</option>
          <option value="Octubre">Octubre</option>
          <option value="Noviembre">Noviembre</option>
          <option value="Diciembre">Diciembre</option>
      </select>
      <select class="filtro" name="filtro_Dia" on:change="{handleFilterChange}">
          <option class="filtro_name" value="">Día</option>
          <option value="Lunes">Lunes</option>
          <option value="Martes">Martes</option>
          <option value="Miércoles">Miércoles</option>
          <option value="Jueves">Jueves</option>
          <option value="Viernes">Viernes</option>
          <option value="Sábado">Sábado</option>
          <option value="Domingo">Domingo</option>
      </select>
      <select class="filtro" name="filtro_Hora" on:change="{handleFilterChange}">
          <option class="filtro_name" value="">Hora</option>
          <option value="0">00:00</option>
          <option value="1">01:00</option>
          <option value="2">02:00</option>
          <option value="3">03:00</option>
          <option value="4">04:00</option>
          <option value="5">05:00</option>
          <option value="6">06:00</option>
          <option value="7">07:00</option>
          <option value="8">08:00</option>
          <option value="9">09:00</option>
          <option value="10">10:00</option>
          <option value="11">11:00</option>
          <option value="12">12:00</option>
          <option value="13">13:00</option>
          <option value="14">14:00</option>
          <option value="15">15:00</option>
          <option value="16">16:00</option>
          <option value="17">17:00</option>
          <option value="18">18:00</option>
          <option value="19">19:00</option>
          <option value="20">20:00</option>
          <option value="21">21:00</option>
          <option value="22">22:00</option>
          <option value="23">23:00</option>
      </select>
      <select class="filtro" name="filtro_Causante" on:change="{handleFilterChange}">
          <option class="filtro_name" value="">Causante</option>
          <option value="Auto">Auto</option>
          <option value="Moto">Moto</option>
          <option value="Bicicleta">Bicicleta</option>
          <option value="Monopatín">Monopatín</option>
          <option value="Peatón">Peatón</option>
          <option value="Transporte público">Transporte público</option>
          <option value="Camión">Camión</option>
          <option value="Taxi">Taxi</option>
          <option value="Utilitario">Utilitario</option>
      </select>
      <select class="filtro" name="filtro_Afectado" on:change="{handleFilterChange}">
          <option class="filtro_name" value="">Afectado</option>
          <option value="Auto">Auto</option>
          <option value="Moto">Moto</option>
          <option value="Bicicleta">Bicicleta</option>
          <option value="Monopatín">Monopatín</option>
          <option value="Peatón">Peatón</option>
          <option value="Transporte público">Transporte público</option>
          <option value="Camión">Camión</option>
          <option value="Taxi">Taxi</option>
          <option value="Utilitario">Utilitario</option>
          <option value="Múltiple">Múltiple</option>
          <option value="Objeto fijo">Objeto fijo</option>
      </select>
  </div>
</div>


  <!------------>
  <!-- Footer -->
  <!------------>
  <footer>
    <p>Creado por Ian David Yong Joon Kim</p>
    <p>
      <a href="https://github.com/iannkimutdt" target="_blank">GitHub</a> |
      <a href="https://www.linkedin.com/in/iandavidyongjoonkim/" target="_blank">LinkedIn</a> 
    </p>
    <p>Visualización de Datos - Universidad Torcuato Di Tella</p>
  </footer>

</main>

<style>
  /******************/
  /* Estilos Textos */
  /******************/
  .header {
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
    flex-direction: column;
    margin-top: 50px;
    margin-bottom: 20px;
  }
  .texto_container {
    text-align: center;
    margin-bottom: 140px;
  }


  /****************************/
  /* Estilos Scrolly Flourish */
  /****************************/
  #scrolly_flourish_1 {
    margin: auto;
    max-width: 80%;
  }
  #scrolly_flourish_2 {
    margin: auto;
    max-width: 80%;
    max-height: 50%;
  }


  /*************************************/
  /* Estilos Mapa Integral Interactivo */
  /*************************************/
  #map {
    height: 85vh;
    width: 100%;
    position: relative;
  }
  /* Estilos para botones de los filtros */
  .filtros_container {
    font-family: 'Coda', sans-serif;
    display: flex;
    flex-direction: column;
    gap: 8px;
    position: absolute;
    top: 10px;
    left: 10px; 
    z-index: 1000;
    background-color: rgb(0, 0, 0, 0.50); 
    padding: 8px;
    border-radius: 4px;
  }
  .filtro_name {
    font-weight: bold;
    font-size: 14px;
    margin-bottom: 4px;
    color: red;
  }
  select {
    font-family: 'Coda', sans-serif;
    padding: 6px;
    font-size: 14px;
    color: #c8c8c8;
    background-color: rgb(0, 0, 0, 1);
    border: 1px solid rgb(44, 44, 44);
    border-radius: 4px;
    outline: none;
    width: 200px;
    transition: border-color 0.2s ease-in-out;
  }
  select:focus {
    border-color: red;
  }
  select:hover {
    border-color: red;
  }
  /* Estilos Popup */
  :global(.popup-content) {
    font-family: 'Coda', sans-serif;
  }
  :global(.popup-content h3) {
      margin: 0 0 10px 0;
      font-size: 16px;
      color: #333;
  }
  :global(.popup-content p) {
      margin: 5px 0;
      font-size: 14px;
  }


  /******************/
  /* Estilos Footer */
  /******************/
  footer {
    text-align: center;
    font-size: 15px;
    margin-top: 120px;
  }
  footer a {
    color: red;
    margin: 0 3px;
  }
  footer a:hover {
    color: lightblue;
  }

</style>

