# Inventario-```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StockMark - Control de Inventario & Marketing Inteligente</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js for beautiful analytics -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: {
                            50: '#f0fdf4',
                            100: '#dcfce7',
                            500: '#22c55e',
                            600: '#16a34a',
                            700: '#15803d',
                        },
                        brand: {
                            dark: '#0f172a',
                            card: '#1e293b',
                            accent: '#38bdf8'
                        }
                    }
                }
            }
        }
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700&display=swap');
        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
        }
        /* Custom scrollbar for premium look */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #1e293b;
        }
        ::-webkit-scrollbar-thumb {
            background: #475569;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #22c55e;
        }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex flex-col">

    <!-- Sistema de Notificaciones Flotantes (Toast) -->
    <div id="toast-container" class="fixed top-5 right-5 z-50 flex flex-col gap-2"></div>

    <!-- Contenedor Principal de la App -->
    <div class="flex flex-col md:flex-row min-h-screen">
        
        <!-- Sidebar de Navegación -->
        <aside class="w-full md:w-64 bg-slate-900 border-b md:border-b-0 md:border-r border-slate-800 flex flex-col justify-between p-4 shrink-0">
            <div>
                <!-- Logotipo -->
                <div class="flex items-center gap-3 px-2 py-4 mb-6">
                    <div class="bg-primary-500 text-slate-950 p-2 rounded-xl flex items-center justify-center shadow-lg shadow-primary-500/20">
                        <i data-lucide="boxes" class="w-6 h-6"></i>
                    </div>
                    <div>
                        <h1 class="font-bold text-lg tracking-tight">Stock<span class="text-primary-500">Mark</span></h1>
                        <p class="text-xs text-slate-400">Control & Promoción</p>
                    </div>
                </div>

                <!-- Enlaces del Menú -->
                <nav class="space-y-1.5">
                    <button onclick="switchTab('dashboard')" id="btn-tab-dashboard" class="nav-btn w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium transition-all bg-primary-600 text-white">
                        <i data-lucide="layout-dashboard" class="w-4 h-4"></i>
                        <span>Panel General</span>
                    </button>
                    <button onclick="switchTab('inventory')" id="btn-tab-inventory" class="nav-btn w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium transition-all text-slate-400 hover:bg-slate-800 hover:text-white">
                        <i data-lucide="package" class="w-4 h-4"></i>
                        <span>Inventario Físico</span>
                    </button>
                    <button onclick="switchTab('marketing')" id="btn-tab-marketing" class="nav-btn w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium transition-all text-slate-400 hover:bg-slate-800 hover:text-white">
                        <i data-lucide="megaphone" class="w-4 h-4"></i>
                        <span>Campañas MKT</span>
                    </button>
                    <button onclick="switchTab('gemini')" id="btn-tab-gemini" class="nav-btn w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium transition-all text-slate-400 hover:bg-slate-800 hover:text-white border border-transparent hover:border-primary-500/20">
                        <i data-lucide="sparkles" class="text-primary-400 w-4 h-4"></i>
                        <span class="text-slate-200">Asistente Copys IA</span>
                    </button>
                </nav>
            </div>

            <!-- Estado de Seguridad / API Key -->
            <div class="mt-8 pt-4 border-t border-slate-800">
                <div class="bg-slate-800/50 rounded-xl p-3 border border-slate-700/50">
                    <div class="flex items-center gap-2 mb-1.5">
                        <i data-lucide="cpu" class="w-4 h-4 text-primary-400"></i>
                        <span class="text-xs font-semibold text-slate-300">Inteligencia Artificial</span>
                    </div>
                    <p class="text-[10px] text-slate-400 leading-relaxed mb-2">Conectado al motor Gemini para potenciar tus copys promocionales.</p>
                    <div class="flex items-center gap-1.5 text-[11px] text-primary-400 bg-primary-950/40 px-2 py-1 rounded border border-primary-900/50">
                        <span class="w-1.5 h-1.5 rounded-full bg-primary-500 animate-pulse"></span>
                        Servicio Activo (Runtime)
                    </div>
                </div>
            </div>
        </aside>

        <!-- Contenido Principal -->
        <main class="flex-1 bg-slate-950 flex flex-col overflow-y-auto">
            
            <!-- Barra Superior de Información -->
            <header class="h-16 border-b border-slate-800 px-6 flex items-center justify-between shrink-0 bg-slate-900/40 backdrop-blur-sm sticky top-0 z-30">
                <div class="flex items-center gap-2">
                    <h2 id="current-tab-title" class="text-lg font-semibold text-white">Panel General</h2>
                </div>
                <!-- Notificaciones Rápidas de Stock Bajo -->
                <div class="flex items-center gap-4">
                    <div id="alert-pill" class="hidden flex items-center gap-2 bg-rose-500/15 border border-rose-500/30 text-rose-400 px-3 py-1.5 rounded-full text-xs font-medium">
                        <span class="relative flex h-2 w-2">
                            <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-rose-400 opacity-75"></span>
                            <span class="relative inline-flex rounded-full h-2 w-2 bg-rose-500"></span>
                        </span>
                        <span id="alert-pill-text">0 alertas de stock</span>
                    </div>
                </div>
            </header>

            <!-- Contenido de las Pestañas -->
            <div class="flex-1 p-6 space-y-6">

                <!-- 1. TAB: DASHBOARD -->
                <section id="tab-dashboard" class="space-y-6">
                    <!-- Tarjetas de Resumen Rápido -->
                    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                        <div class="bg-slate-900 border border-slate-800 p-5 rounded-2xl relative overflow-hidden">
                            <div class="absolute top-0 right-0 p-4 opacity-10">
                                <i data-lucide="package-search" class="w-20 h-20 text-white"></i>
                            </div>
                            <p class="text-slate-400 text-xs font-medium uppercase tracking-wider">Productos Registrados</p>
                            <h3 id="stat-total-products" class="text-3xl font-bold mt-2 text-white">0</h3>
                            <p class="text-xs text-slate-400 mt-2"><span id="stat-total-categories" class="text-emerald-400 font-semibold">0</span> categorías activas</p>
                        </div>
                        <div class="bg-slate-900 border border-slate-800 p-5 rounded-2xl relative overflow-hidden">
                            <div class="absolute top-0 right-0 p-4 opacity-10">
                                <i data-lucide="circle-dollar-sign" class="w-20 h-20 text-white"></i>
                            </div>
                            <p class="text-slate-400 text-xs font-medium uppercase tracking-wider">Valor Total Inventario</p>
                            <h3 id="stat-total-value" class="text-3xl font-bold mt-2 text-white">$0.00</h3>
                            <p class="text-xs text-slate-400 mt-2">A precio de coste</p>
                        </div>
                        <div class="bg-slate-900 border border-slate-800 p-5 rounded-2xl relative overflow-hidden">
                            <div class="absolute top-0 right-0 p-4 opacity-10">
                                <i data-lucide="trending-up" class="w-20 h-20 text-white"></i>
                            </div>
                            <p class="text-slate-400 text-xs font-medium uppercase tracking-wider">Ganancia Estimada</p>
                            <h3 id="stat-potential-revenue" class="text-3xl font-bold mt-2 text-emerald-400">$0.00</h3>
                            <p class="text-xs text-slate-400 mt-2">Margen bruto estimado</p>
                        </div>
                        <div class="bg-slate-900 border border-slate-800 p-5 rounded-2xl relative overflow-hidden">
                            <div class="absolute top-0 right-0 p-4 opacity-10">
                                <i data-lucide="sparkles" class="w-20 h-20 text-white"></i>
                            </div>
                            <p class="text-slate-400 text-xs font-medium uppercase tracking-wider">Campañas Activas</p>
                            <h3 id="stat-active-campaigns" class="text-3xl font-bold mt-2 text-sky-400">0</h3>
                            <p class="text-xs text-slate-400 mt-2">Impulsando ventas hoy</p>
                        </div>
                    </div>

                    <!-- Gráficos & Distribuciones -->
                    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                        <!-- Distribución por Categorías -->
                        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 lg:col-span-1">
                            <h4 class="font-semibold text-slate-200 mb-4 flex items-center gap-2">
                                <i data-lucide="pie-chart" class="w-4 h-4 text-emerald-400"></i>
                                Stock por Categoría
                            </h4>
                            <div class="relative h-64 flex items-center justify-center">
                                <canvas id="categoryChart"></canvas>
                            </div>
                        </div>

                        <!-- Panel de Acción Rápida de Inventario Inactivo o en Alerta -->
                        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 lg:col-span-2 flex flex-col justify-between">
                            <div>
                                <h4 class="font-semibold text-slate-200 mb-2 flex items-center justify-between">
                                    <span class="flex items-center gap-2">
                                        <i data-lucide="alert-triangle" class="w-4 h-4 text-amber-500"></i>
                                        Sugerencias de Rotación y Marketing
                                    </span>
                                    <span class="text-xs text-primary-400 font-normal">Recomendado por IA</span>
                                </h4>
                                <p class="text-xs text-slate-400 mb-4">La Inteligencia Artificial detecta productos con exceso de almacenamiento o stock crítico para sugerir campañas de liquidación.</p>
                                
                                <div id="mkt-suggestions-list" class="space-y-3 max-h-64 overflow-y-auto pr-1">
                                    <!-- Generado dinámicamente -->
                                </div>
                            </div>
                            
                            <div class="mt-4 pt-4 border-t border-slate-800 flex justify-end">
                                <button onclick="switchTab('gemini')" class="text-xs bg-slate-800 hover:bg-slate-700 text-slate-200 font-semibold py-2 px-4 rounded-xl flex items-center gap-2 transition-colors">
                                    <span>Ir al Redactor Creativo</span>
                                    <i data-lucide="arrow-right" class="w-3.5 h-3.5"></i>
                                </button>
                            </div>
                        </div>
                    </div>

                    <!-- Tabla de Alertas Críticas Recientes -->
                    <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5">
                        <div class="flex items-center justify-between mb-4">
                            <h4 class="font-semibold text-slate-200 flex items-center gap-2">
                                <i data-lucide="shield-alert" class="w-4 h-4 text-rose-500"></i>
                                Alertas de Stock Crítico o Ruptura
                            </h4>
                            <button onclick="switchTab('inventory')" class="text-xs text-primary-400 hover:underline">Ver todo el inventario</button>
                        </div>
                        <div class="overflow-x-auto">
                            <table class="w-full text-left border-collapse">
                                <thead>
                                    <tr class="border-b border-slate-800 text-slate-400 text-xs font-semibold">
                                        <th class="pb-3">Producto</th>
                                        <th class="pb-3">Categoría</th>
                                        <th class="pb-3 text-right">Existencia</th>
                                        <th class="pb-3 text-right">Límite Mín.</th>
                                        <th class="pb-3 text-right">Estado</th>
                                        <th class="pb-3 text-center">Acciones MKT</th>
                                    </tr>
                                </thead>
                                <tbody id="critical-stock-table-body" class="divide-y divide-slate-800/50 text-sm">
                                    <!-- Cargado por JS -->
                                </tbody>
                            </table>
                        </div>
                    </div>
                </section>

                <!-- 2. TAB: INVENTARIO FÍSICO -->
                <section id="tab-inventory" class="space-y-6 hidden">
                    <div class="flex flex-col sm:flex-row gap-4 items-center justify-between bg-slate-900 p-4 rounded-2xl border border-slate-800">
                        <!-- Buscador y Filtro -->
                        <div class="flex flex-col sm:flex-row gap-3 w-full sm:w-auto flex-1">
                            <div class="relative flex-1 max-w-md">
                                <i data-lucide="search" class="w-4 h-4 absolute left-3.5 top-1/2 -translate-y-1/2 text-slate-400"></i>
                                <input type="text" id="search-product" oninput="renderInventoryTable()" placeholder="Buscar producto por nombre o proveedor..." class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl pl-10 pr-4 py-2 text-sm w-full focus:outline-none focus:border-primary-500 transition-colors">
                            </div>
                            <select id="filter-category" onchange="renderInventoryTable()" class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-4 py-2 text-sm focus:outline-none focus:border-primary-500 transition-colors">
                                <option value="all">Todas las Categorías</option>
                                <!-- Categorías cargadas dinámicamente -->
                            </select>
                        </div>
                        <!-- Botón para añadir producto -->
                        <button onclick="openProductModal()" class="w-full sm:w-auto bg-primary-600 hover:bg-primary-500 text-slate-950 font-bold px-4 py-2 rounded-xl flex items-center justify-center gap-2 transition-all">
                            <i data-lucide="plus-circle" class="w-4 h-4"></i>
                            <span>Nuevo Producto</span>
                        </button>
                    </div>

                    <!-- Tabla Principal de Inventario -->
                    <div class="bg-slate-900 border border-slate-800 rounded-2xl overflow-hidden">
                        <div class="overflow-x-auto">
                            <table class="w-full text-left border-collapse">
                                <thead>
                                    <tr class="bg-slate-950/50 border-b border-slate-800 text-slate-400 text-xs font-semibold">
                                        <th class="p-4">SKU / Producto</th>
                                        <th class="p-4">Categoría</th>
                                        <th class="p-4">Proveedor</th>
                                        <th class="p-4 text-right">Costo</th>
                                        <th class="p-4 text-right">Precio Venta</th>
                                        <th class="p-4 text-right">Existencia / Mínimo</th>
                                        <th class="p-4 text-center">Estado</th>
                                        <th class="p-4 text-right">Acciones</th>
                                    </tr>
                                </thead>
                                <tbody id="inventory-table-body" class="divide-y divide-slate-800 text-sm">
                                    <!-- Dinámico por JS -->
                                </tbody>
                            </table>
                        </div>
                    </div>
                </section>

                <!-- 3. TAB: CAMPAÑAS DE MARKETING -->
                <section id="tab-marketing" class="space-y-6 hidden">
                    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                        
                        <!-- Formulario para Crear/Editar Campaña -->
                        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 h-fit">
                            <h3 class="font-bold text-slate-200 mb-4 flex items-center gap-2">
                                <i data-lucide="megaphone" class="w-5 h-5 text-sky-400"></i>
                                Planificar Campaña
                            </h3>
                            <form id="campaign-form" onsubmit="handleCampaignSubmit(event)" class="space-y-4">
                                <input type="hidden" id="campaign-id">
                                
                                <div>
                                    <label class="block text-xs font-semibold text-slate-400 mb-1.5">Nombre de la Campaña</label>
                                    <input type="text" id="campaign-name" placeholder="Ej: Liquidación de Invierno" required class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-sm w-full focus:outline-none focus:border-primary-500">
                                </div>

                                <div>
                                    <label class="block text-xs font-semibold text-slate-400 mb-1.5">Producto Vinculado</label>
                                    <select id="campaign-product-id" onchange="autoCalculatePromo()" required class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-sm w-full focus:outline-none focus:border-primary-500">
                                        <option value="">Seleccione un producto...</option>
                                        <!-- Cargar dinámicamente -->
                                    </select>
                                </div>

                                <div class="grid grid-cols-2 gap-3">
                                    <div>
                                        <label class="block text-xs font-semibold text-slate-400 mb-1.5">Tipo Descuento</label>
                                        <select id="campaign-discount-type" onchange="autoCalculatePromo()" class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-sm w-full focus:outline-none focus:border-primary-500">
                                            <option value="percent">Porcentaje (%)</option>
                                            <option value="fixed">Fijo ($)</option>
                                        </select>
                                    </div>
                                    <div>
                                        <label class="block text-xs font-semibold text-slate-400 mb-1.5">Valor Descuento</label>
                                        <input type="number" id="campaign-discount-val" oninput="autoCalculatePromo()" placeholder="15" min="0" required class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-sm w-full focus:outline-none focus:border-primary-500">
                                    </div>
                                </div>

                                <div class="p-3 bg-slate-950 rounded-xl border border-slate-800 space-y-1">
                                    <p class="text-[11px] text-slate-400">Simulación Financiera:</p>
                                    <div class="flex justify-between text-xs">
                                        <span>Precio normal:</span>
                                        <span id="promo-normal-price" class="font-semibold">$0.00</span>
                                    </div>
                                    <div class="flex justify-between text-xs text-emerald-400">
                                        <span>Nuevo Precio Promo:</span>
                                        <span id="promo-calculated-price" class="font-bold">$0.00</span>
                                    </div>
                                    <div class="flex justify-between text-[11px] text-rose-400 border-t border-slate-800 pt-1 mt-1">
                                        <span>Margen Resultante:</span>
                                        <span id="promo-calculated-margin" class="font-semibold">$0.00</span>
                                    </div>
                                </div>

                                <div>
                                    <label class="block text-xs font-semibold text-slate-400 mb-1.5">Canal de Difusión Sugerido</label>
                                    <select id="campaign-channel" class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-sm w-full focus:outline-none focus:border-primary-500">
                                        <option value="Instagram / Facebook">Instagram / Facebook Ads</option>
                                        <option value="Newsletter">Boletín Informativo por Correo</option>
                                        <option value="Tienda Fisica">Cartelería / Oferta Tienda Física</option>
                                        <option value="WhatsApp Business">WhatsApp Business Broadcast</option>
                                    </select>
                                </div>

                                <div class="grid grid-cols-2 gap-3">
                                    <div>
                                        <label class="block text-xs font-semibold text-slate-400 mb-1.5">Fecha Inicio</label>
                                        <input type="date" id="campaign-start" required class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-xs w-full focus:outline-none focus:border-primary-500">
                                    </div>
                                    <div>
                                        <label class="block text-xs font-semibold text-slate-400 mb-1.5">Fecha Fin</label>
                                        <input type="date" id="campaign-end" required class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-xs w-full focus:outline-none focus:border-primary-500">
                                    </div>
                                </div>

                                <button type="submit" class="w-full bg-sky-500 hover:bg-sky-400 text-slate-950 font-bold py-2.5 rounded-xl text-sm flex items-center justify-center gap-2 transition-all">
                                    <i data-lucide="plus" class="w-4 h-4"></i>
                                    <span>Guardar Campaña</span>
                                </button>
                            </form>
                        </div>

                        <!-- Panel de Lista de Campañas Activas/Programadas -->
                        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 lg:col-span-2 flex flex-col justify-between">
                            <div>
                                <h3 class="font-bold text-slate-200 mb-4 flex items-center gap-2">
                                    <i data-lucide="calendar-range" class="w-5 h-5 text-emerald-400"></i>
                                    Campañas y Promociones Programadas
                                </h3>
                                
                                <div id="campaigns-grid" class="grid grid-cols-1 md:grid-cols-2 gap-4 max-h-[500px] overflow-y-auto pr-1">
                                    <!-- Dinámico -->
                                </div>
                            </div>
                        </div>

                    </div>
                </section>

                <!-- 4. TAB: ASISTENTE GEMINI IA (COPYWRITER) -->
                <section id="tab-gemini" class="space-y-6 hidden">
                    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                        
                        <!-- Selector de Contexto de Producto -->
                        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 h-fit">
                            <h3 class="font-bold text-slate-200 mb-2 flex items-center gap-2">
                                <i data-lucide="brain-circuit" class="w-5 h-5 text-primary-400"></i>
                                Parámetros del Copy
                            </h3>
                            <p class="text-xs text-slate-400 mb-4">Selecciona un producto del inventario para que el Asistente Gemini redacte copys de venta altamente profesionales y persuasivos.</p>
                            
                            <div class="space-y-4">
                                <div>
                                    <label class="block text-xs font-semibold text-slate-400 mb-1.5">1. Seleccionar Producto</label>
                                    <select id="gemini-product-select" onchange="autoFillGeminiParams()" class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2.5 text-sm w-full focus:outline-none focus:border-primary-500">
                                        <option value="">-- Elige un producto --</option>
                                        <!-- Carga dinámica -->
                                    </select>
                                </div>

                                <div class="p-3 bg-slate-950 rounded-xl border border-slate-800 space-y-1.5 text-xs">
                                    <span class="text-slate-400 font-semibold block mb-1">Información que recibirá el IA:</span>
                                    <div class="flex justify-between">
                                        <span class="text-slate-500">Categoría:</span>
                                        <span id="gemini-ctx-category" class="text-slate-300">-</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span class="text-slate-500">Precio normal:</span>
                                        <span id="gemini-ctx-price" class="text-slate-300">-</span>
                                    </div>
                                    <div class="flex justify-between">
                                        <span class="text-slate-500">Stock Actual:</span>
                                        <span id="gemini-ctx-stock" class="text-slate-300">-</span>
                                    </div>
                                </div>

                                <div>
                                    <label class="block text-xs font-semibold text-slate-400 mb-1.5">2. Tipo de Copy o Publicación</label>
                                    <select id="gemini-copy-type" class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2.5 text-sm w-full focus:outline-none focus:border-primary-500">
                                        <option value="instagram_post">Publicación de Instagram/Facebook (Creativo con Emojis)</option>
                                        <option value="newsletter">Correo Informativo (Newsletter de Oferta)</option>
                                        <option value="excess_liquidation">Liquidación Urgente (Enfoque en Exceso de Stock)</option>
                                        <option value="funny_hook">Anuncio Gracioso / Memorable</option>
                                    </select>
                                </div>

                                <div>
                                    <label class="block text-xs font-semibold text-slate-400 mb-1.5">3. Tono de Comunicación</label>
                                    <select id="gemini-copy-tone" class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2.5 text-sm w-full focus:outline-none focus:border-primary-500">
                                        <option value="persuasivo">Persuasivo & Enérgico</option>
                                        <option value="profesional">Profesional & Elegante</option>
                                        <option value="urgente">Frenético (Última Oportunidad)</option>
                                        <option value="cercano">Amigable & Divertido</option>
                                    </select>
                                </div>

                                <div>
                                    <label class="block text-xs font-semibold text-slate-400 mb-1.5">Instrucciones Adicionales (Opcional)</label>
                                    <textarea id="gemini-custom-prompt" rows="2" placeholder="Ej: Mencionar que el envío es gratuito..." class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl p-3 text-xs w-full focus:outline-none focus:border-primary-500 resize-none"></textarea>
                                </div>

                                <button onclick="requestGeminiMarketingCopy()" class="w-full bg-gradient-to-r from-emerald-500 to-primary-600 hover:from-emerald-400 hover:to-primary-500 text-slate-950 font-bold py-3 rounded-xl text-sm flex items-center justify-center gap-2 transition-all shadow-lg shadow-emerald-500/10">
                                    <i data-lucide="sparkles" class="w-4 h-4 text-slate-950"></i>
                                    <span>Redactar Copy de Venta</span>
                                </button>
                            </div>
                        </div>

                        <!-- Espacio de Resultado Creativo -->
                        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 lg:col-span-2 flex flex-col justify-between">
                            <div class="flex-1">
                                <div class="flex items-center justify-between mb-4">
                                    <h3 class="font-bold text-slate-200 flex items-center gap-2">
                                        <i data-lucide="text-quote" class="w-5 h-5 text-sky-400"></i>
                                        Resultado Generado por IA
                                    </h3>
                                    <button onclick="copyGeneratedText()" class="text-xs text-primary-400 hover:text-primary-300 flex items-center gap-1.5">
                                        <i data-lucide="copy" class="w-3.5 h-3.5"></i>
                                        <span>Copiar Texto</span>
                                    </button>
                                </div>

                                <!-- Loading / Esperando -->
                                <div id="gemini-loading" class="hidden flex flex-col items-center justify-center h-64 space-y-3">
                                    <div class="animate-spin rounded-full h-10 w-10 border-t-2 border-b-2 border-primary-500"></div>
                                    <p class="text-sm text-slate-400 animate-pulse">Gemini está analizando los datos del producto y redactando tu campaña...</p>
                                </div>

                                <!-- Texto Generado -->
                                <div id="gemini-result-container" class="bg-slate-950 border border-slate-800/80 rounded-2xl p-5 min-h-[300px] h-[350px] overflow-y-auto">
                                    <p id="gemini-placeholder-text" class="text-slate-500 text-sm italic h-full flex items-center justify-center text-center">Completa los parámetros de la izquierda y haz clic en "Redactar Copy de Venta" para comenzar.</p>
                                    <div id="gemini-response-text" class="text-slate-200 text-sm leading-relaxed whitespace-pre-wrap select-all hidden"></div>
                                </div>
                            </div>

                            <div class="mt-4 pt-4 border-t border-slate-800 flex items-center justify-between text-xs text-slate-400">
                                <span class="flex items-center gap-1.5">
                                    <i data-lucide="info" class="w-3.5 h-3.5 text-slate-500"></i>
                                    Puedes editar o personalizar el texto directamente.
                                </span>
                                <span class="text-slate-500">Powered by Gemini-2.5-Flash</span>
                            </div>
                        </div>

                    </div>
                </section>

            </div>

            <!-- Footer con estado o info -->
            <footer class="h-10 bg-slate-900 border-t border-slate-800 px-6 flex items-center justify-between text-xs text-slate-400 shrink-0">
                <span>© 2026 StockMark Inc. Control Total de Stock.</span>
                <span class="flex items-center gap-1 text-emerald-400">
                    <span class="w-2 h-2 rounded-full bg-emerald-500"></span> Base de Datos Local
                </span>
            </footer>
        </main>
    </div>

    <!-- MODAL: AGREGAR / EDITAR PRODUCTO -->
    <div id="product-modal" class="fixed inset-0 z-50 flex items-center justify-center bg-slate-950/80 backdrop-blur-sm opacity-0 pointer-events-none transition-all duration-300">
        <div class="bg-slate-900 border border-slate-800 w-full max-w-xl rounded-2xl overflow-hidden shadow-2xl m-4 transform scale-95 transition-all duration-300">
            <!-- Header Modal -->
            <div class="px-6 py-4 bg-slate-950 border-b border-slate-800 flex items-center justify-between">
                <h3 id="modal-title" class="font-bold text-slate-100 text-lg">Añadir Nuevo Producto</h3>
                <button onclick="closeProductModal()" class="text-slate-400 hover:text-white transition-colors">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>
            
            <!-- Cuerpo Formulario -->
            <form id="product-form" onsubmit="handleProductSubmit(event)" class="p-6 space-y-4">
                <input type="hidden" id="product-id">
                
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label class="block text-xs font-semibold text-slate-400 mb-1.5">Nombre del Producto *</label>
                        <input type="text" id="prod-name" placeholder="Ej: Camisa Slim Fit Algodón" required class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-sm w-full focus:outline-none focus:border-primary-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-400 mb-1.5">Categoría *</label>
                        <input type="text" id="prod-category" placeholder="Ej: Ropa, Tecnología, Hogar..." required class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-sm w-full focus:outline-none focus:border-primary-500">
                    </div>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label class="block text-xs font-semibold text-slate-400 mb-1.5">Proveedor</label>
                        <input type="text" id="prod-supplier" placeholder="Ej: Distribuidora Textil S.A." class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-sm w-full focus:outline-none focus:border-primary-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-400 mb-1.5">Código SKU / Identificador</label>
                        <input type="text" id="prod-sku" placeholder="Ej: PROD-9921" class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-sm w-full focus:outline-none focus:border-primary-500">
                    </div>
                </div>

                <div class="grid grid-cols-2 md:grid-cols-4 gap-3">
                    <div>
                        <label class="block text-xs font-semibold text-slate-400 mb-1.5">Precio Costo ($) *</label>
                        <input type="number" step="0.01" min="0" id="prod-cost" placeholder="10.00" required class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-sm w-full focus:outline-none focus:border-primary-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-400 mb-1.5">Precio Venta ($) *</label>
                        <input type="number" step="0.01" min="0" id="prod-price" placeholder="25.00" required class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-sm w-full focus:outline-none focus:border-primary-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-400 mb-1.5">Existencia *</label>
                        <input type="number" min="0" id="prod-stock" placeholder="100" required class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-sm w-full focus:outline-none focus:border-primary-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-400 mb-1.5">Límite Alerta *</label>
                        <input type="number" min="0" id="prod-min-stock" placeholder="15" required class="bg-slate-950 border border-slate-800 text-slate-100 rounded-xl px-3 py-2 text-sm w-full focus:outline-none focus:border-primary-500">
                    </div>
                </div>

                <!-- Botones Acción -->
                <div class="pt-4 border-t border-slate-800 flex justify-end gap-3">
                    <button type="button" onclick="closeProductModal()" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-xl text-sm font-semibold transition-colors">Cancelar</button>
                    <button type="submit" class="px-5 py-2 bg-primary-600 hover:bg-primary-500 text-slate-950 font-bold rounded-xl text-sm transition-colors">Guardar Producto</button>
                </div>
            </form>
        </div>
    </div>

    <!-- LOGIC SYSTEM AND STATE CONTROL -->
    <script>
        // --- 1. DATOS INICIALES Y CONFIGURACIÓN ---
        let products = [
            { id: "1", sku: "TECH-109", name: "Audífonos Bluetooth Over-Ear", category: "Tecnología", supplier: "GlobalTech S.A.", cost: 15.00, price: 39.99, stock: 120, minStock: 25 },
            { id: "2", sku: "CLOT-882", name: "Zapatos Deportivos RunnerX", category: "Ropa", supplier: "Estilo Calzado Inc.", cost: 22.50, price: 65.00, stock: 8, minStock: 15 },
            { id: "3", sku: "HOME-043", name: "Humidificador Ultrasónico Led", category: "Hogar", supplier: "Ambiental S.L.", cost: 8.00, price: 24.50, stock: 45, minStock: 10 },
            { id: "4", sku: "TECH-502", name: "Soporte Magnético Premium", category: "Tecnología", supplier: "GlobalTech S.A.", cost: 3.50, price: 12.99, stock: 150, minStock: 20 },
            { id: "5", sku: "CLOT-203", name: "Camiseta Deportiva Transpirable", category: "Ropa", supplier: "Fábrica Nacional Textil", cost: 5.00, price: 18.00, stock: 3, minStock: 12 },
            { id: "6", sku: "HME-901", name: "Lámpara de Escritorio Inteligente", category: "Hogar", supplier: "Ambiental S.L.", cost: 12.00, price: 35.00, stock: 95, minStock: 15 }
        ];

        let campaigns = [
            { id: "1", name: "Liquidación Urgente RunnerX", productId: "2", discountType: "percent", discountVal: 20, channel: "Instagram / Facebook", start: "2026-07-01", end: "2026-07-15" }
        ];

        let activeTab = "dashboard";
        let categoryChartInstance = null;

        // --- 2. GESTION DE INICIO ---
        window.onload = function() {
            lucide.createIcons();
            // Cargar de LocalStorage si existe
            const savedProducts = localStorage.getItem('sm_products');
            const savedCampaigns = localStorage.getItem('sm_campaigns');
            if (savedProducts) products = JSON.parse(savedProducts);
            if (savedCampaigns) campaigns = JSON.parse(savedCampaigns);

            updateAllData();
        }

        // --- 3. NOTIFICACIONES PERSONALIZADAS ---
        function showToast(title, message, type = 'info') {
            const container = document.getElementById('toast-container');
            const toast = document.createElement('div');
            toast.className = `p-4 rounded-xl shadow-xl border flex items-start gap-3 transition-all duration-300 transform translate-y-2 opacity-0 max-w-sm w-full bg-slate-900 ${
                type === 'success' ? 'border-emerald-500/30 text-emerald-400' :
                type === 'error' ? 'border-rose-500/30 text-rose-400' :
                type === 'warning' ? 'border-amber-500/30 text-amber-400' : 'border-sky-500/30 text-sky-400'
            }`;

            let icon = 'info';
            if (type === 'success') icon = 'check-circle';
            if (type === 'error') icon = 'x-circle';
            if (type === 'warning') icon = 'alert-triangle';

            toast.innerHTML = `
                <div class="mt-0.5"><i data-lucide="${icon}" class="w-5 h-5"></i></div>
                <div class="flex-1">
                    <h5 class="text-xs font-bold text-white">${title}</h5>
                    <p class="text-[11px] text-slate-400 mt-0.5">${message}</p>
                </div>
                <button class="text-slate-500 hover:text-white" onclick="this.parentElement.remove()"><i data-lucide="x" class="w-4 h-4"></i></button>
            `;
            container.appendChild(toast);
            lucide.createIcons();

            // Animación Entrada
            setTimeout(() => {
                toast.classList.remove('translate-y-2', 'opacity-0');
            }, 50);

            // Desvanecer después de 4.5 segundos
            setTimeout(() => {
                toast.classList.add('opacity-0', 'translate-y-[-10px]');
                setTimeout(() => toast.remove(), 300);
            }, 4500);
        }

        // --- 4. CONTROL DE PESTAÑAS (TABS) ---
        function switchTab(tabName) {
            // Desactivar todas las pestañas
            document.querySelectorAll('section[id^="tab-"]').forEach(section => {
                section.classList.add('hidden');
            });
            document.querySelectorAll('.nav-btn').forEach(btn => {
                btn.className = "nav-btn w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium transition-all text-slate-400 hover:bg-slate-800 hover:text-white";
            });

            // Activar seleccionada
            document.getElementById(`tab-${tabName}`).classList.remove('hidden');
            const targetBtn = document.getElementById(`btn-tab-${tabName}`);
            if (targetBtn) {
                targetBtn.className = "nav-btn w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium transition-all bg-primary-600 text-white";
            }

            activeTab = tabName;
            
            // Actualizar título de cabecera
            const titles = {
                dashboard: "Panel General de Rendimiento",
                inventory: "Gestión de Inventario Físico",
                marketing: "Campañas y Promociones Activas",
                gemini: "Asistente Redactor Gemini IA"
            };
            document.getElementById('current-tab-title').innerText = titles[tabName] || "Control de Inventario";

            // Acciones especiales al cambiar de tab
            if (tabName === 'dashboard') {
                updateAllData();
            } else if (tabName === 'inventory') {
                renderInventoryTable();
            } else if (tabName === 'marketing') {
                renderCampaignList();
                autoCalculatePromo();
            } else if (tabName === 'gemini') {
                populateGeminiSelectors();
            }
        }

        // --- 5. ACTUALIZACIÓN GLOBAL DE DATOS ---
        function updateAllData() {
            saveToStorage();
            calculateSummaryStats();
            renderCategoryChart();
            renderMarketingSuggestions();
            renderCriticalAlertsTable();
            updateLowStockAlertPill();
        }

        function saveToStorage() {
            localStorage.setItem('sm_products', JSON.stringify(products));
            localStorage.setItem('sm_campaigns', JSON.stringify(campaigns));
        }

        // --- 6. PANEL DASHBOARD LOGIC ---
        function calculateSummaryStats() {
            const totalProducts = products.length;
            const uniqueCategories = [...new Set(products.map(p => p.category.trim()))].length;
            
            let totalValue = 0;
            let totalPotentialRevenue = 0;
            
            products.forEach(p => {
                totalValue += p.cost * p.stock;
                totalPotentialRevenue += (p.price - p.cost) * p.stock;
            });

            document.getElementById('stat-total-products').innerText = totalProducts;
            document.getElementById('stat-total-categories').innerText = uniqueCategories;
            document.getElementById('stat-total-value').innerText = `$${totalValue.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2})}`;
            document.getElementById('stat-potential-revenue').innerText = `$${totalPotentialRevenue.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2})}`;
            document.getElementById('stat-active-campaigns').innerText = campaigns.length;
        }

        function updateLowStockAlertPill() {
            const lowStockProducts = products.filter(p => p.stock <= p.minStock);
            const alertPill = document.getElementById('alert-pill');
            const alertText = document.getElementById('alert-pill-text');
            
            if (lowStockProducts.length > 0) {
                alertPill.classList.remove('hidden');
                alertText.innerText = `${lowStockProducts.length} Alertas de Stock Bajo`;
            } else {
                alertPill.classList.add('hidden');
            }
        }

        // Sugerencias Inteligentes de Rotación en base al stock
        function renderMarketingSuggestions() {
            const container = document.getElementById('mkt-suggestions-list');
            container.innerHTML = "";

            // 1. Encontrar productos críticos (bajo stock)
            const lowStock = products.filter(p => p.stock <= p.minStock);
            // 2. Encontrar productos sobrestockeados (por ejemplo > 100 de stock y sin campaña)
            const overStock = products.filter(p => p.stock >= 90 && !campaigns.some(c => c.productId === p.id));

            let suggestionsHtml = "";

            if (overStock.length > 0) {
                overStock.forEach(p => {
                    suggestionsHtml += `
                        <div class="bg-slate-950 p-4 rounded-xl border border-primary-500/20 hover:border-primary-500/40 flex flex-col md:flex-row items-start md:items-center justify-between gap-3 transition-colors">
                            <div class="space-y-1">
                                <div class="flex items-center gap-2">
                                    <span class="bg-primary-950 text-primary-400 text-[10px] px-2 py-0.5 rounded border border-primary-900">Alto Almacenamiento</span>
                                    <span class="text-xs text-slate-500">SKU: ${p.sku}</span>
                                </div>
                                <h5 class="text-sm font-semibold text-white">${p.name}</h5>
                                <p class="text-xs text-slate-400">Hay <strong class="text-slate-200">${p.stock} unidades</strong> acumuladas. Te recomendamos crear una campaña de descuento para acelerar su salida.</p>
                            </div>
                            <button onclick="quickActionCreateCampaign('${p.id}')" class="shrink-0 bg-primary-600 hover:bg-primary-500 text-slate-950 font-bold px-3 py-1.5 rounded-lg text-xs flex items-center gap-1 transition-all">
                                <i data-lucide="sparkles" class="w-3.5 h-3.5"></i>
                                <span>Crear Promoción</span>
                            </button>
                        </div>
                    `;
                });
            }

            if (lowStock.length > 0) {
                lowStock.forEach(p => {
                    suggestionsHtml += `
                        <div class="bg-slate-950 p-4 rounded-xl border border-rose-500/20 hover:border-rose-500/40 flex flex-col md:flex-row items-start md:items-center justify-between gap-3 transition-colors">
                            <div class="space-y-1">
                                <div class="flex items-center gap-2">
                                    <span class="bg-rose-950 text-rose-400 text-[10px] px-2 py-0.5 rounded border border-rose-900">Riesgo de Ruptura</span>
                                    <span class="text-xs text-slate-500">Existencia: ${p.stock}/${p.minStock}</span>
                                </div>
                                <h5 class="text-sm font-semibold text-white">${p.name}</h5>
                                <p class="text-xs text-slate-400">Producto estrella bajo nivel mínimo. Considera avisar a tu proveedor o programar un newsletter de "Últimas Unidades".</p>
                            </div>
                            <button onclick="quickActionLaunchCopy('${p.id}')" class="shrink-0 bg-slate-800 hover:bg-slate-700 text-slate-200 font-medium px-3 py-1.5 rounded-lg text-xs flex items-center gap-1 transition-all">
                                <i data-lucide="megaphone" class="w-3.5 h-3.5"></i>
                                <span>Crear Copy Urgente</span>
                            </button>
                        </div>
                    `;
                });
            }

            if (suggestionsHtml === "") {
                suggestionsHtml = `
                    <div class="text-center py-6 text-slate-500 text-xs">
                        <i data-lucide="check" class="w-8 h-8 mx-auto text-emerald-400 mb-2"></i>
                        Tu inventario se encuentra balanceado y sin incidencias críticas actualmente.
                    </div>
                `;
            }

            container.innerHTML = suggestionsHtml;
            lucide.createIcons();
        }

        // Renderizado de Alertas Críticas Recientes
        function renderCriticalAlertsTable() {
            const body = document.getElementById('critical-stock-table-body');
            body.innerHTML = "";

            const lowStockProducts = products.filter(p => p.stock <= p.minStock);

            if (lowStockProducts.length === 0) {
                body.innerHTML = `
                    <tr>
                        <td colspan="6" class="py-6 text-center text-slate-500 text-xs italic">No hay alertas críticas de stock en este momento.</td>
                    </tr>
                `;
                return;
            }

            lowStockProducts.forEach(p => {
                const statusBadge = p.stock === 0 ? 
                    `<span class="bg-rose-950 text-rose-400 text-[10px] px-2.5 py-1 rounded-full border border-rose-900 font-semibold">Agotado</span>` : 
                    `<span class="bg-amber-950 text-amber-400 text-[10px] px-2.5 py-1 rounded-full border border-amber-900 font-semibold">Bajo Stock</span>`;

                body.innerHTML += `
                    <tr class="hover:bg-slate-800/20 transition-colors">
                        <td class="py-3 font-medium text-white">${p.name} <span class="text-slate-500 text-xs block">SKU: ${p.sku}</span></td>
                        <td class="py-3 text-slate-400">${p.category}</td>
                        <td class="py-3 text-right font-bold ${p.stock === 0 ? 'text-rose-400' : 'text-amber-400'}">${p.stock} uds</td>
                        <td class="py-3 text-right text-slate-500">${p.minStock} uds</td>
                        <td class="py-3 text-right">${statusBadge}</td>
                        <td class="py-3 text-center">
                            <button onclick="quickActionLaunchCopy('${p.id}')" class="text-xs bg-slate-800 hover:bg-slate-700 text-slate-300 py-1 px-2.5 rounded-lg transition-colors inline-flex items-center gap-1">
                                <i data-lucide="sparkles" class="w-3 h-3 text-primary-400"></i>
                                Redactar Urgente
                            </button>
                        </td>
                    </tr>
                `;
            });
            lucide.createIcons();
        }

        // Gráfico Circular de Stock por Categoría
        function renderCategoryChart() {
            const ctx = document.getElementById('categoryChart').getContext('2d');
            
            // Agrupar productos por categoría
            const catMap = {};
            products.forEach(p => {
                const cat = p.category.trim();
                catMap[cat] = (catMap[cat] || 0) + p.stock;
            });

            const labels = Object.keys(catMap);
            const data = Object.values(catMap);

            if (categoryChartInstance) {
                categoryChartInstance.destroy();
            }

            if (labels.length === 0) {
                return;
            }

            categoryChartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [{
                        data: data,
                        backgroundColor: [
                            '#22c55e', '#38bdf8', '#f59e0b', '#ec4899', '#8b5cf6', '#3b82f6', '#14b8a6'
                        ],
                        borderWidth: 2,
                        borderColor: '#0f172a'
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            position: 'bottom',
                            labels: {
                                color: '#cbd5e1',
                                font: {
                                    family: 'Plus Jakarta Sans',
                                    size: 11
                                },
                                boxWidth: 12
                            }
                        }
                    },
                    cutout: '65%'
                }
            });
        }

        // --- 7. TAB: INVENTARIO FÍSICO CONTROL ---
        function renderInventoryTable() {
            const body = document.getElementById('inventory-table-body');
            const searchVal = document.getElementById('search-product').value.toLowerCase();
            const filterCat = document.getElementById('filter-category').value;
            
            body.innerHTML = "";

            // Actualizar select de categorías
            const categories = [...new Set(products.map(p => p.category.trim()))];
            const selectCat = document.getElementById('filter-category');
            const currentSelected = selectCat.value;
            selectCat.innerHTML = `<option value="all">Todas las Categorías</option>`;
            categories.forEach(c => {
                selectCat.innerHTML += `<option value="${c}" ${c === currentSelected ? 'selected' : ''}>${c}</option>`;
            });

            // Filtrado
            const filtered = products.filter(p => {
                const matchesSearch = p.name.toLowerCase().includes(searchVal) || 
                                      p.supplier.toLowerCase().includes(searchVal) || 
                                      p.sku.toLowerCase().includes(searchVal);
                const matchesCategory = filterCat === 'all' || p.category === filterCat;
                return matchesSearch && matchesCategory;
            });

            if (filtered.length === 0) {
                body.innerHTML = `
                    <tr>
                        <td colspan="8" class="p-8 text-center text-slate-500 text-sm">
                            No se encontraron productos que coincidan con la búsqueda.
                        </td>
                    </tr>
                `;
                return;
            }

            filtered.forEach(p => {
                const isCritical = p.stock <= p.minStock;
                const statusBadge = p.stock === 0 ? 
                    `<span class="bg-rose-950/80 text-rose-400 text-xs px-2.5 py-1 rounded-full border border-rose-900/50 font-medium">Sin Stock</span>` :
                    isCritical ? 
                    `<span class="bg-amber-950/80 text-amber-400 text-xs px-2.5 py-1 rounded-full border border-amber-900/50 font-medium">Bajo Stock</span>` :
                    `<span class="bg-emerald-950/80 text-emerald-400 text-xs px-2.5 py-1 rounded-full border border-emerald-900/50 font-medium font-semibold">Excelente</span>`;

                body.innerHTML += `
                    <tr class="hover:bg-slate-900/40 transition-colors">
                        <td class="p-4">
                            <span class="text-xs text-slate-500 font-mono block">${p.sku || 'S/N'}</span>
                            <span class="font-semibold text-slate-200">${p.name}</span>
                        </td>
                        <td class="p-4 text-slate-300 font-medium">${p.category}</td>
                        <td class="p-4 text-slate-400 text-xs">${p.supplier || 'N/A'}</td>
                        <td class="p-4 text-right font-mono text-slate-300">$${p.cost.toFixed(2)}</td>
                        <td class="p-4 text-right font-mono font-semibold text-emerald-400">$${p.price.toFixed(2)}</td>
                        <td class="p-4 text-right">
                            <span class="font-bold text-slate-100">${p.stock}</span>
                            <span class="text-slate-500 text-xs"> / ${p.minStock}</span>
                        </td>
                        <td class="p-4 text-center">${statusBadge}</td>
                        <td class="p-4 text-right">
                            <div class="flex items-center justify-end gap-2">
                                <button onclick="quickActionCreateCampaign('${p.id}')" title="Crear Campaña" class="p-1.5 bg-sky-950/50 text-sky-400 hover:bg-sky-900 border border-sky-800 rounded-lg transition-colors">
                                    <i data-lucide="megaphone" class="w-4 h-4"></i>
                                </button>
                                <button onclick="openProductModal('${p.id}')" title="Editar Producto" class="p-1.5 bg-slate-800 text-slate-300 hover:bg-slate-700 rounded-lg transition-colors">
                                    <i data-lucide="edit" class="w-4 h-4"></i>
                                </button>
                                <button onclick="deleteProduct('${p.id}')" title="Eliminar Producto" class="p-1.5 bg-rose-950/50 text-rose-400 hover:bg-rose-950 rounded-lg border border-rose-900/30 transition-colors">
                                    <i data-lucide="trash-2" class="w-4 h-4"></i>
                                </button>
                            </div>
                        </td>
                    </tr>
                `;
            });
            lucide.createIcons();
        }

        // --- MODAL DE PRODUCTOS LOGIC ---
        function openProductModal(id = null) {
            const modal = document.getElementById('product-modal');
            const title = document.getElementById('modal-title');
            
            document.getElementById('product-form').reset();
            document.getElementById('product-id').value = "";

            if (id) {
                title.innerText = "Editar Producto Existente";
                const p = products.find(prod => prod.id === id);
                if (p) {
                    document.getElementById('product-id').value = p.id;
                    document.getElementById('prod-name').value = p.name;
                    document.getElementById('prod-category').value = p.category;
                    document.getElementById('prod-supplier').value = p.supplier;
                    document.getElementById('prod-sku').value = p.sku;
                    document.getElementById('prod-cost').value = p.cost;
                    document.getElementById('prod-price').value = p.price;
                    document.getElementById('prod-stock').value = p.stock;
                    document.getElementById('prod-min-stock').value = p.minStock;
                }
            } else {
                title.innerText = "Añadir Nuevo Producto";
                // Autogenerar SKU sencillo
                document.getElementById('prod-sku').value = "SKU-" + Math.floor(1000 + Math.random() * 9000);
            }

            modal.classList.remove('opacity-0', 'pointer-events-none');
            modal.querySelector('div').classList.remove('scale-95');
        }

        function closeProductModal() {
            const modal = document.getElementById('product-modal');
            modal.classList.add('opacity-0', 'pointer-events-none');
            modal.querySelector('div').classList.add('scale-95');
        }

        function handleProductSubmit(event) {
            event.preventDefault();
            const id = document.getElementById('product-id').value;
            const name = document.getElementById('prod-name').value;
            const category = document.getElementById('prod-category').value;
            const supplier = document.getElementById('prod-supplier').value;
            const sku = document.getElementById('prod-sku').value;
            const cost = parseFloat(document.getElementById('prod-cost').value);
            const price = parseFloat(document.getElementById('prod-price').value);
            const stock = parseInt(document.getElementById('prod-stock').value);
            const minStock = parseInt(document.getElementById('prod-min-stock').value);

            if (price < cost) {
                showToast("Aviso Financiero", "El precio de venta es menor al precio de costo. El margen será negativo.", "warning");
            }

            if (id) {
                // Editar
                const idx = products.findIndex(p => p.id === id);
                if (idx !== -1) {
                    products[idx] = { id, name, category, supplier, sku, cost, price, stock, minStock };
                    showToast("Producto Actualizado", `Se ha guardado los cambios en "${name}" de forma local.`, "success");
                }
            } else {
                // Crear
                const newId = (products.length + 1).toString();
                products.push({ id: newId, name, category, supplier, sku, cost, price, stock, minStock });
                showToast("Producto Creado", `"${name}" se ha añadido exitosamente al inventario.`, "success");
            }

            closeProductModal();
            updateAllData();
            renderInventoryTable();
        }

        function deleteProduct(id) {
            const p = products.find(prod => prod.id === id);
            if (!p) return;
            
            // Confirmación en modal nativo no está permitida por políticas de iframe, usaremos confirmación de interfaz limpia o borrado directo con opción de deshacer.
            // Para simplicidad, borramos directo y notificamos
            products = products.filter(prod => prod.id !== id);
            showToast("Producto Eliminado", `El producto "${p.name}" fue retirado del inventario.`, "error");
            
            updateAllData();
            renderInventoryTable();
        }

        // --- 8. TAB: CAMPAÑAS DE MARKETING LOGIC ---
        function populateCampaignProductSelect() {
            const select = document.getElementById('campaign-product-id');
            const currentSelected = select.value;
            
            select.innerHTML = `<option value="">Seleccione un producto...</option>`;
            products.forEach(p => {
                select.innerHTML += `<option value="${p.id}" ${p.id === currentSelected ? 'selected' : ''}>${p.name} (Stock: ${p.stock})</option>`;
            });
        }

        function autoCalculatePromo() {
            populateCampaignProductSelect();
            
            const prodId = document.getElementById('campaign-product-id').value;
            const discountType = document.getElementById('campaign-discount-type').value;
            const discountVal = parseFloat(document.getElementById('campaign-discount-val').value) || 0;

            const normalPriceLabel = document.getElementById('promo-normal-price');
            const promoPriceLabel = document.getElementById('promo-calculated-price');
            const marginLabel = document.getElementById('promo-calculated-margin');

            if (!prodId) {
                normalPriceLabel.innerText = "$0.00";
                promoPriceLabel.innerText = "$0.00";
                marginLabel.innerText = "$0.00";
                return;
            }

            const p = products.find(prod => prod.id === prodId);
            if (p) {
                let calculatedPrice = p.price;
                if (discountType === 'percent') {
                    calculatedPrice = p.price * (1 - (discountVal / 100));
                } else {
                    calculatedPrice = p.price - discountVal;
                }

                if (calculatedPrice < 0) calculatedPrice = 0;

                const margin = calculatedPrice - p.cost;

                normalPriceLabel.innerText = `$${p.price.toFixed(2)}`;
                promoPriceLabel.innerText = `$${calculatedPrice.toFixed(2)}`;
                
                marginLabel.innerText = `$${margin.toFixed(2)}`;
                if (margin <= 0) {
                    marginLabel.className = "font-bold text-rose-500";
                } else {
                    marginLabel.className = "font-bold text-emerald-400";
                }
            }
        }

        function handleCampaignSubmit(event) {
            event.preventDefault();
            const id = document.getElementById('campaign-id').value;
            const name = document.getElementById('campaign-name').value;
            const productId = document.getElementById('campaign-product-id').value;
            const discountType = document.getElementById('campaign-discount-type').value;
            const discountVal = parseFloat(document.getElementById('campaign-discount-val').value);
            const channel = document.getElementById('campaign-channel').value;
            const start = document.getElementById('campaign-start').value;
            const end = document.getElementById('campaign-end').value;

            if (id) {
                // Editar
                const idx = campaigns.findIndex(c => c.id === id);
                if (idx !== -1) {
                    campaigns[idx] = { id, name, productId, discountType, discountVal, channel, start, end };
                    showToast("Campaña Actualizada", `Los cambios para "${name}" fueron guardados.`, "success");
                }
            } else {
                // Crear
                const newId = (campaigns.length + 1).toString();
                campaigns.push({ id: newId, name, productId, discountType, discountVal, channel, start, end });
                showToast("Campaña Programada", `Campaña "${name}" se encuentra activa e integrada.`, "success");
            }

            // Resetear
            document.getElementById('campaign-form').reset();
            document.getElementById('campaign-id').value = "";
            
            updateAllData();
            renderCampaignList();
            autoCalculatePromo();
        }

        function renderCampaignList() {
            const container = document.getElementById('campaigns-grid');
            container.innerHTML = "";

            if (campaigns.length === 0) {
                container.innerHTML = `
                    <div class="col-span-2 text-center py-12 text-slate-500 text-sm italic">
                        No hay campañas de mercadeo programadas para este mes.
                    </div>
                `;
                return;
            }

            campaigns.forEach(c => {
                const prod = products.find(p => p.id === c.productId);
                if (!prod) return;

                const promoPrice = c.discountType === 'percent' ? 
                    prod.price * (1 - (c.discountVal / 100)) : 
                    prod.price - c.discountVal;

                container.innerHTML += `
                    <div class="bg-slate-950 border border-slate-800 rounded-xl p-4 flex flex-col justify-between hover:border-sky-500/30 transition-all">
                        <div class="space-y-2">
                            <div class="flex items-center justify-between">
                                <span class="bg-sky-950 text-sky-400 text-[10px] font-semibold px-2 py-0.5 rounded border border-sky-900">${c.channel}</span>
                                <div class="flex gap-1">
                                    <button onclick="editCampaign('${c.id}')" class="text-slate-400 hover:text-white p-1"><i data-lucide="edit" class="w-3.5 h-3.5"></i></button>
                                    <button onclick="deleteCampaign('${c.id}')" class="text-rose-400 hover:text-rose-300 p-1"><i data-lucide="trash" class="w-3.5 h-3.5"></i></button>
                                </div>
                            </div>
                            <h4 class="font-bold text-white text-sm">${c.name}</h4>
                            <div class="bg-slate-900/60 p-2.5 rounded-lg border border-slate-800 text-xs space-y-1">
                                <p class="text-slate-400">Producto: <span class="text-slate-200 font-semibold">${prod.name}</span></p>
                                <div class="flex justify-between">
                                    <span class="text-slate-400">Precio Promo:</span>
                                    <span class="text-emerald-400 font-bold">$${promoPrice.toFixed(2)} <span class="text-[10px] text-slate-500 font-normal">($${prod.price.toFixed(2)})</span></span>
                                </div>
                            </div>
                            <p class="text-[11px] text-slate-500 flex items-center gap-1">
                                <i data-lucide="calendar" class="w-3 h-3"></i>
                                Vigencia: ${c.start} al ${c.end}
                            </p>
                        </div>
                        <div class="mt-4 pt-3 border-t border-slate-900 flex justify-end">
                            <button onclick="quickActionLaunchCopy('${prod.id}', '${c.name}')" class="text-[11px] bg-sky-900/50 hover:bg-sky-900 text-sky-300 border border-sky-800 px-3 py-1.5 rounded-lg font-semibold flex items-center gap-1 transition-all">
                                <i data-lucide="sparkles" class="w-3 h-3 text-sky-400"></i>
                                Redactar Post IA
                            </button>
                        </div>
                    </div>
                `;
            });
            lucide.createIcons();
        }

        function editCampaign(id) {
            const c = campaigns.find(camp => camp.id === id);
            if (c) {
                document.getElementById('campaign-id').value = c.id;
                document.getElementById('campaign-name').value = c.name;
                document.getElementById('campaign-product-id').value = c.productId;
                document.getElementById('campaign-discount-type').value = c.discountType;
                document.getElementById('campaign-discount-val').value = c.discountVal;
                document.getElementById('campaign-channel').value = c.channel;
                document.getElementById('campaign-start').value = c.start;
                document.getElementById('campaign-end').value = c.end;

                autoCalculatePromo();
                showToast("Modo Edición", `Cargando datos de campaña "${c.name}" en el formulario.`, "info");
            }
        }

        function deleteCampaign(id) {
            campaigns = campaigns.filter(c => c.id !== id);
            showToast("Campaña Eliminada", "La campaña publicitaria fue cancelada.", "warning");
            updateAllData();
            renderCampaignList();
        }

        // Acciones rápidas cruzadas
        function quickActionCreateCampaign(productId) {
            switchTab('marketing');
            document.getElementById('campaign-product-id').value = productId;
            autoCalculatePromo();
            showToast("Acción Rápida", "Por favor completa la simulación de descuento para el producto seleccionado.", "info");
        }

        function quickActionLaunchCopy(productId, campaignName = '') {
            switchTab('gemini');
            document.getElementById('gemini-product-select').value = productId;
            if (campaignName) {
                document.getElementById('gemini-custom-prompt').value = `Redactar enfocado en la campaña: ${campaignName}`;
            } else {
                document.getElementById('gemini-custom-prompt').value = "";
            }
            autoFillGeminiParams();
        }


        // --- 9. TAB: ASISTENTE GEMINI IA WRITER ---
        function populateGeminiSelectors() {
            const select = document.getElementById('gemini-product-select');
            const currentSelected = select.value;

            select.innerHTML = `<option value="">-- Elige un producto --</option>`;
            products.forEach(p => {
                select.innerHTML += `<option value="${p.id}" ${p.id === currentSelected ? 'selected' : ''}>${p.name} (Ref: ${p.sku})</option>`;
            });
        }

        function autoFillGeminiParams() {
            const prodId = document.getElementById('gemini-product-select').value;
            const categoryEl = document.getElementById('gemini-ctx-category');
            const priceEl = document.getElementById('gemini-ctx-price');
            const stockEl = document.getElementById('gemini-ctx-stock');

            if (!prodId) {
                categoryEl.innerText = "-";
                priceEl.innerText = "-";
                stockEl.innerText = "-";
                return;
            }

            const p = products.find(prod => prod.id === prodId);
            if (p) {
                categoryEl.innerText = p.category;
                priceEl.innerText = `$${p.price.toFixed(2)}`;
                stockEl.innerText = `${p.stock} unidades`;
            }
        }

        // Copiar texto generado al portapapeles
        function copyGeneratedText() {
            const responseText = document.getElementById('gemini-response-text').innerText;
            if (!responseText) {
                showToast("Portapapeles", "No hay texto generado para copiar.", "warning");
                return;
            }

            // Crear área temporal para copiar (execCommand es el más seguro en iFrames restringidos de Canvas)
            const tempTextArea = document.createElement("textarea");
            tempTextArea.value = responseText;
            document.body.appendChild(tempTextArea);
            tempTextArea.select();
            document.execCommand("copy");
            document.body.removeChild(tempTextArea);

            showToast("¡Copiado con Éxito!", "El texto se ha copiado a tu portapapeles listo para usar.", "success");
        }

        // LLAMADA API GEMINI CON RESPALDO Y EXPONENTIAL BACKOFF
        async function requestGeminiMarketingCopy() {
            const prodId = document.getElementById('gemini-product-select').value;
            if (!prodId) {
                showToast("Campo requerido", "Debes seleccionar un producto del inventario para darle contexto a la IA.", "warning");
                return;
            }

            const p = products.find(prod => prod.id === prodId);
            const copyType = document.getElementById('gemini-copy-type').value;
            const copyTone = document.getElementById('gemini-copy-tone').value;
            const customInstructions = document.getElementById('gemini-custom-prompt').value;

            // Mostrar cargando
            document.getElementById('gemini-placeholder-text').classList.add('hidden');
            document.getElementById('gemini-response-text').classList.add('hidden');
            document.getElementById('gemini-loading').classList.remove('hidden');

            // Construir Prompt Profesional en base al Inventario y Tipo de Campaña
            let prompt = `Eres un redactor creativo experto en marketing digital y conversión (copywriter) para pequeños comercios y e-commerce. Genera un copy altamente atractivo y profesional en español utilizando los siguientes datos específicos del inventario:
            - Nombre de Producto: ${p.name}
            - Categoría: ${p.category}
            - Existencia en Almacén: ${p.stock} unidades de stock actual.
            - Precio Normal de Venta: $${p.price.toFixed(2)} (Menciónalo de forma atractiva o con descuento implícito según el tipo).
            `;

            if (copyType === 'instagram_post') {
                prompt += `\nGenera una publicación excelente para Instagram o Facebook. Incluye ganchos de atención llamativos, descripciones que resalten beneficios clave del producto, llamadas a la acción (CTA) directas, emojis estratégicos y hashtags populares.`;
            } else if (copyType === 'newsletter') {
                prompt += `\nGenera una Newsletter atractiva y directa (Asunto del correo con gancho y Cuerpo del correo). Que se sienta personalizado, exclusivo y ofrezca valor de manera inmediata destacando por qué deben adquirirlo hoy.`;
            } else if (copyType === 'excess_liquidation') {
                prompt += `\nGenera un copy enfocado en urgencia o liquidación masiva. Ya que tenemos un stock importante de ${p.stock} unidades, inventa una oferta especial imbatible para vaciar el almacén rápido. El tono debe incitar a la escasez.`;
            } else if (copyType === 'funny_hook') {
                prompt += `\nGenera una redacción divertida, fresca o con un chiste simpático relacionado al producto o categoría para viralizar en redes, manteniendo la intención de venta elegante pero carismática.`;
            }

            prompt += `\nTono de comunicación requerido: ${copyTone}.`;
            if (customInstructions) {
                prompt += `\nInstrucciones especiales adicionales solicitadas por el comerciante: ${customInstructions}`;
            }

            prompt += `\nPor favor, responde directamente con la propuesta de redacción terminada. No saludes, no expliques las razones de tu estructura y no pongas introducciones adicionales.`;

            // API Call setup
            const apiKey = ""; // Runtime environment handles the key
            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
            const systemPrompt = "Eres un asistente de redacción de marketing para tiendas comerciales experto en elevar ventas y rotación de stock.";

            const payload = {
                contents: [{ parts: [{ text: prompt }] }],
                systemInstruction: { parts: [{ text: systemPrompt }] }
            };

            // Implementación de backoff exponencial
            let retries = 5;
            let delay = 1000;
            let success = false;
            let resultText = "";

            while (retries > 0 && !success) {
                try {
                    const response = await fetch(url, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (!response.ok) {
                        throw new Error(`API Error: Status ${response.status}`);
                    }

                    const result = await response.json();
                    resultText = result.candidates?.[0]?.content?.parts?.[0]?.text;
                    if (resultText) {
                        success = true;
                    } else {
                        throw new Error("Formato de respuesta de API inválido.");
                    }
                } catch (error) {
                    retries--;
                    if (retries === 0) {
                        document.getElementById('gemini-loading').classList.add('hidden');
                        document.getElementById('gemini-response-text').innerText = "Lo sentimos, el servicio de Inteligencia Artificial de Gemini no se pudo conectar temporalmente después de varios intentos. Verifica tu conexión e inténtalo nuevamente.";
                        document.getElementById('gemini-response-text').classList.remove('hidden');
                        showToast("Error de Conexión", "No se pudo conectar con el motor de IA de Gemini.", "error");
                        return;
                    }
                    await new Promise(res => setTimeout(res, delay));
                    delay *= 2; // Incrementar retraso de backoff
                }
            }

            // Mostrar resultado exitoso
            document.getElementById('gemini-loading').classList.add('hidden');
            const responseContainer = document.getElementById('gemini-response-text');
            responseContainer.innerText = resultText;
            responseContainer.classList.remove('hidden');
            showToast("Copy Redactado", "Tu copy promocional ha sido generado exitosamente por Gemini.", "success");
        }
    </script>
</body>
</html>

```
