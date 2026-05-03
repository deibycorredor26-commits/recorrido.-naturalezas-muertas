<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Recorrido Interactivo: Ruta del Secuestro</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap" rel="stylesheet">
    
    <!-- Leaflet CSS para el Mapa Real -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin=""/>
    
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f8fafc;
        }

        /* Asegurar que el mapa ocupe todo el espacio de su contenedor */
        #map {
            width: 100%;
            height: 100%;
            z-index: 1; /* Mantener debajo del header y panel */
        }

        /* Custom scrollbar para el panel de información */
        .custom-scrollbar::-webkit-scrollbar {
            width: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: #f1f1f1; 
            border-radius: 4px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background: #cbd5e1; 
            border-radius: 4px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover {
            background: #94a3b8; 
        }

        /* Transición suave para los textos */
        .fade-text {
            transition: opacity 0.3s ease-in-out;
        }
        
        /* Personalización de los popups de Leaflet */
        .leaflet-tooltip {
            font-family: 'Inter', sans-serif;
            font-weight: 600;
            border-radius: 8px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>
<body class="h-screen flex flex-col text-slate-800">

    <!-- Header -->
    <header class="bg-slate-900 text-white p-4 shadow-md z-20 relative">
        <div class="max-w-7xl mx-auto flex justify-between items-center">
            <div>
                <h1 class="text-2xl font-bold tracking-tight">La Ruta de los Farallones al Naya</h1>
                <p class="text-sm text-slate-400 mt-1">Explora el recorrido sobre el mapa topográfico real de la región.</p>
            </div>
            <div class="hidden sm:block text-slate-300 text-sm bg-slate-800 px-4 py-2 rounded-full border border-slate-700">
                Mapa Topográfico Interactivo
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="flex-1 flex flex-col md:flex-row overflow-hidden relative z-10">
        
        <!-- Interactive Geographic Map Area -->
        <section id="map" class="flex-1 relative border-r border-slate-200 min-h-[50vh] md:min-h-0"></section>

        <!-- Information Panel -->
        <section class="w-full md:w-1/3 lg:w-2/5 bg-white flex flex-col shadow-[-4px_0_15px_rgba(0,0,0,0.1)] z-20 relative">
            
            <!-- Encabezado del Panel (Sin imágenes) -->
            <div class="p-8 bg-slate-50 border-b border-slate-200 fade-text" id="infoTextHeader">
                <span id="infoStepBadge" class="inline-block px-3 py-1 bg-emerald-100 text-emerald-800 text-xs font-bold rounded-full mb-3 uppercase tracking-wider shadow-sm">
                    Paso 1
                </span>
                <h2 id="infoTitle" class="text-2xl md:text-3xl font-bold text-slate-800 mb-2 leading-tight">
                    Lugar inicial
                </h2>
                <div class="flex items-center text-slate-500">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
                    </svg>
                    <p id="infoSubtitle" class="font-medium text-sm">
                        Cargando ubicación...
                    </p>
                </div>
            </div>
            
            <!-- Contenido textual inferior -->
            <div class="p-8 flex-1 overflow-y-auto custom-scrollbar bg-white">
                <h3 class="text-sm font-bold text-slate-400 uppercase tracking-wider mb-5 border-b border-slate-100 pb-2">
                    Sucesos en este punto
                </h3>
                <ol id="infoEventsList" class="space-y-4 list-decimal pl-5 text-slate-700 leading-relaxed marker:text-emerald-600 marker:font-bold fade-text">
                    <!-- Los eventos se inyectan aquí -->
                </ol>
            </div>
            
            <!-- Controles de navegación en el panel -->
            <div class="p-4 bg-white border-t border-slate-100 flex justify-between">
                <button id="btnPrev" class="px-4 py-2 text-sm font-semibold text-slate-600 bg-slate-100 rounded hover:bg-slate-200 transition-colors disabled:opacity-50 disabled:cursor-not-allowed">
                    &larr; Anterior
                </button>
                <button id="btnNext" class="px-4 py-2 text-sm font-semibold text-white bg-slate-800 rounded hover:bg-slate-700 transition-colors disabled:opacity-50 disabled:cursor-not-allowed">
                    Siguiente &rarr;
                </button>
            </div>
        </section>

    </main>

    <!-- Leaflet JS -->
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>

    <script>
        /**
         * JSON de Datos del Recorrido con Coordenadas Geográficas Reales
         * Latitud y Longitud aproximadas de la zona del Valle del Cauca y Cauca
         */
        const journeyData = [
            {
                id: 1,
                title: "Lugar inicial de la retención",
                subtitle: "Vía al mar (Aprox. Km 18 / Afueras de Cali)",
                lat: 3.518, 
                lng: -76.623, 
                events: [
                    "Retención inicial en restaurantes de la vía.",
                    "Desde allí se llevan a 54 personas por la fuerza."
                ]
            },
            {
                id: 2,
                title: "Los Farallones de Cali",
                subtitle: "Cordillera occidental (Ascenso a 3.400 msnm)",
                lat: 3.328, 
                lng: -76.680, 
                events: [
                    "Ascenso abrupto desde 1.000 hasta 3.400 metros de altitud.",
                    "Selva fría, niebla constante, terreno muy hostil.",
                    "El clima y la topografía producen agotamiento inmediato en los retenidos."
                ]
            },
            {
                id: 3,
                title: "Campamentos en la selva",
                subtitle: "Vertiente del Pacífico (Alta montaña)",
                lat: 3.250, 
                lng: -76.850, 
                events: [
                    "Construcción de campamentos improvisados.",
                    "Plásticos y hules tendidos directamente sobre el barro.",
                    "Refugios precarios donde la lluvia y el hambre marcan la rutina diaria.",
                    "Allí se sienten más intensamente la presión del ejército y la escasez de alimentos."
                ]
            },
            {
                id: 4,
                title: "Valle del Naya",
                subtitle: "Zona media de la cuenca del Naya",
                lat: 3.150, 
                lng: -76.950, 
                events: [
                    "Descenso hacia selva cálida y húmeda, zona de cultivos de coca y ranchos para procesar cocaína.",
                    "Se esperaba encontrar bodegas de comida, pero estaban vacías por operaciones del ejército.",
                    "Escenario de nuevas divisiones internas del grupo armado y combates cercanos."
                ]
            },
            {
                id: 5,
                title: "Ranchos abandonados",
                subtitle: "Refugios en zona cocalera",
                lat: 3.080, 
                lng: -77.100, 
                events: [
                    "Espacios donde se ven obligados a dejar a los secuestrados más enfermos (como el médico Nassif).",
                    "Uso de ranchos cocaleros como refugios momentáneos.",
                    "Entorno rodeado de cultivos ilícitos, con guerrilleros debilitados física y logísticamente."
                ]
            },
            {
                id: 6,
                title: "Riberas del río Naya",
                subtitle: "Selva baja profunda (Hacia el Pacífico)",
                lat: 3.000, 
                lng: -77.250, 
                events: [
                    "Últimos descensos geográficos hacia los mil metros de altitud.",
                    "Terreno sumamente húmedo, lluvias torrenciales constantes.",
                    "Mayor presencia y presión militar.",
                    "Allí se intensifica la descomposición física severa de los secuestrados y la desesperación total de los captores."
                ]
            }
        ];

        // Variables Globales
        let map;
        let leafletMarkers = {};
        let activeNodeId = 1;

        // Elementos del panel informativo
        const infoTitle = document.getElementById('infoTitle');
        const infoSubtitle = document.getElementById('infoSubtitle');
        const infoEventsList = document.getElementById('infoEventsList');
        const infoStepBadge = document.getElementById('infoStepBadge');
        const infoTextHeader = document.getElementById('infoTextHeader');
        
        // Botones de navegación
        const btnPrev = document.getElementById('btnPrev');
        const btnNext = document.getElementById('btnNext');

        function initMap() {
            // Inicializar el mapa centrado en la zona general
            map = L.map('map').setView([3.25, -76.9], 9);

            // Usar OpenTopoMap para mostrar claramente el relieve y las montañas
            L.tileLayer('https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png', {
                maxZoom: 17,
                attribution: 'Map data: &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, <a href="http://viewfinderpanoramas.org">SRTM</a> | Map style: &copy; <a href="https://opentopomap.org">OpenTopoMap</a> (<a href="https://creativecommons.org/licenses/by-sa/3.0/">CC-BY-SA</a>)'
            }).addTo(map);

            // Extraer las coordenadas para dibujar la línea
            const routeCoordinates = journeyData.map(node => [node.lat, node.lng]);

            // Dibujar la ruta (Polilínea punteada roja)
            const routeLine = L.polyline(routeCoordinates, {
                color: '#ef4444', 
                weight: 4, 
                dashArray: '10, 10',
                opacity: 0.8
            }).addTo(map);

            // Crear marcadores para cada punto
            journeyData.forEach(node => {
                const marker = L.circleMarker([node.lat, node.lng], {
                    radius: 8,
                    fillColor: "#fb923c", // Naranja por defecto
                    color: "#ffffff",
                    weight: 2,
                    opacity: 1,
                    fillOpacity: 0.9
                }).addTo(map);

                // Añadir un tooltip (etiqueta al pasar el ratón)
                marker.bindTooltip(`<b>${node.id}. ${node.title}</b>`, {
                    direction: 'right',
                    offset: [10, 0]
                });

                // Evento de clic en el marcador
                marker.on('click', () => {
                    selectNode(node.id);
                });

                // Guardar referencia al marcador
                leafletMarkers[node.id] = marker;
            });

            // Ajustar el mapa para que se vea toda la ruta
            map.fitBounds(routeLine.getBounds(), { padding: [50, 50] });
        }

        // Función para actualizar la interfaz y el mapa al seleccionar un punto
        function selectNode(id) {
            activeNodeId = id;
            const nodeData = journeyData.find(n => n.id === id);
            if (!nodeData) return;

            // --- Actualizar Marcadores en el Mapa ---
            Object.values(leafletMarkers).forEach(m => {
                m.setStyle({ fillColor: '#fb923c', radius: 8 }); // Restaurar estilo
            });
            
            // Resaltar el marcador activo
            leafletMarkers[id].setStyle({ fillColor: '#ef4444', radius: 12 });
            leafletMarkers[id].bringToFront();

            // Mover la cámara del mapa suavemente al punto
            map.flyTo([nodeData.lat, nodeData.lng], 11, {
                duration: 1.5 // Duración de la animación en segundos
            });

            // --- Actualizar Textos del Panel ---
            // Efecto de desvanecimiento
            infoTextHeader.style.opacity = 0;
            infoEventsList.style.opacity = 0;
            
            setTimeout(() => {
                // Actualizar información
                infoStepBadge.textContent = `Punto ${nodeData.id} de ${journeyData.length}`;
                infoTitle.textContent = nodeData.title;
                infoSubtitle.textContent = nodeData.subtitle;
                
                // Construir lista de eventos
                infoEventsList.innerHTML = '';
                nodeData.events.forEach(eventText => {
                    const li = document.createElement('li');
                    li.textContent = eventText;
                    li.className = "pl-2";
                    infoEventsList.appendChild(li);
                });

                // Actualizar estado de los botones
                btnPrev.disabled = (id === 1);
                btnNext.disabled = (id === journeyData.length);

                // Reaparecer textos
                infoTextHeader.style.opacity = 1;
                infoEventsList.style.opacity = 1;
            }, 300);
        }

        // Eventos de los botones
        btnPrev.addEventListener('click', () => {
            if (activeNodeId > 1) selectNode(activeNodeId - 1);
        });
        
        btnNext.addEventListener('click', () => {
            if (activeNodeId < journeyData.length) selectNode(activeNodeId + 1);
        });

        // Iniciar la aplicación cuando cargue la página
        window.onload = () => {
            initMap();
            // Retrasar ligeramente la selección del primer nodo para asegurar que el mapa cargue
            setTimeout(() => selectNode(1), 500); 
        };

    </script>
</body>
</html>
