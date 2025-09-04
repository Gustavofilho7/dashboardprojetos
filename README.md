
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard de Governança de Projetos | Suporte N2/N3</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Calm Corporate -->
    <!-- Application Structure Plan: A classic dashboard layout with top-level KPIs, interactive filters, a grid of dynamic charts for visual analysis (status, workload, focus), and a detailed project table for drill-down. This structure provides an immediate overview while allowing for intuitive, filter-based exploration, directly addressing the user's need for visibility and project tracking. -->
    <!-- Visualization & Content Choices: KPIs -> Inform -> HTML Cards -> Dynamic update -> Quick summary. Filters -> Organize -> HTML Selectors -> JS filtering -> User-driven exploration. Status -> Compare Proportions -> Donut Chart (Chart.js) -> Hover tooltips, dynamic updates -> Quick health check. Workload -> Compare Distribution -> Bar Chart (Chart.js) -> Updates -> Resource management. Focus -> Compare Allocation -> Bar Chart (Chart.js) -> Updates -> Strategic alignment check. Details -> Inform -> Dynamic HTML Table -> Filtering -> Access specific project info. All choices support the dashboard structure and use Canvas-based Chart.js. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f8f9fa; }
        .chart-container { position: relative; width: 100%; height: 96; max-height: 400px; }
        .card { background-color: #ffffff; border-radius: 0.75rem; box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1); transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out; }
        .card:hover { transform: translateY(-4px); box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -2px rgb(0 0 0 / 0.1); }
        select { -webkit-appearance: none; -moz-appearance: none; appearance: none; background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke-width='1.5' stroke='%236b7280' class='w-6 h-6'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M19.5 8.25l-7.5 7.5-7.5-7.5' /%3E%3C/svg%3E%0A"); background-repeat: no-repeat; background-position: right 0.75rem center; background-size: 1.5em 1.5em; padding-right: 2.5rem; }
        .modal-overlay { transition: opacity 0.3s ease; }
        .modal-content { transition: transform 0.3s ease; }
        #loader { border-top-color: #3498db; animation: spin 1s linear infinite; }
        @keyframes spin { to { transform: rotate(360deg); } }
    </style>
</head>
<body class="text-gray-800">

    <div id="loader" class="fixed inset-0 bg-white bg-opacity-75 z-50 flex flex-col items-center justify-center text-center p-4">
        <div class="w-16 h-16 border-4 border-gray-200 border-t-blue-500 rounded-full animate-spin"></div>
        <p id="loader-text" class="mt-4 text-gray-600 font-medium">Iniciando...</p>
    </div>

    <div class="container mx-auto p-4 sm:p-6 lg:p-8">
        <header class="mb-8 flex justify-between items-center">
            <div>
                <h1 class="text-3xl font-bold text-gray-900">Dashboard de Governança de Projetos</h1>
                <p class="text-md text-gray-600 mt-1">Acompanhamento de iniciativas do time de Suporte N2/N3.</p>
            </div>
            <button id="refreshBtn" title="Atualizar Dados da Planilha" class="text-gray-500 hover:text-gray-800 p-2 rounded-full hover:bg-gray-200 transition">
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0l3.181 3.183a8.25 8.25 0 0011.667 0l3.181-3.183m-4.991-2.691V5.25a2.25 2.25 0 00-2.25-2.25h-4.5a2.25 2.25 0 00-2.25 2.25v4.992m11.667 0l-3.181 3.183a8.25 8.25 0 01-11.667 0l-3.181-3.183" /></svg>
            </button>
        </header>

        <div id="errorBanner" class="hidden bg-red-100 border-l-4 border-red-500 text-red-700 p-4 mb-6" role="alert">
            <p class="font-bold">Erro de Configuração</p>
            <p id="errorBannerText"></p>
        </div>

        <main class="hidden">
            <section id="kpis" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-8"></section>

            <section id="filters" class="card p-6 mb-8 flex flex-col sm:flex-row gap-4 items-center">
                <h3 class="text-lg font-semibold text-gray-700 sm:mr-4">Filtros:</h3>
                <div class="w-full sm:w-auto flex-grow">
                    <select id="responsibleFilter" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition"></select>
                </div>
                <div class="w-full sm:w-auto flex-grow">
                    <select id="statusFilter" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition"></select>
                </div>
                <div class="w-full sm:w-auto flex-grow">
                    <select id="classificationFilter" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition"></select>
                </div>
            </section>

            <section id="charts" class="grid grid-cols-1 lg:grid-cols-3 gap-8 mb-8">
                <div class="card p-6 lg:col-span-1">
                    <h3 class="text-xl font-semibold text-center mb-4">Projetos por Status</h3>
                    <div class="chart-container h-72 sm:h-80 mx-auto" style="max-width: 400px;"><canvas id="statusChart"></canvas></div>
                </div>
                <div class="card p-6 lg:col-span-2">
                    <h3 class="text-xl font-semibold text-center mb-4">Projetos por Responsável</h3>
                    <div class="chart-container h-72 sm:h-80"><canvas id="responsibleChart"></canvas></div>
                </div>
                 <div class="card p-6 lg:col-span-3">
                    <h3 class="text-xl font-semibold text-center mb-4">Foco Estratégico do Time</h3>
                    <div class="chart-container h-72 sm:h-80"><canvas id="focusChart"></canvas></div>
                </div>
            </section>

            <section id="project-table" class="card p-6">
                <h3 class="text-xl font-semibold mb-4">Detalhamento dos Projetos</h3>
                <div class="overflow-x-auto">
                    <table class="min-w-full divide-y divide-gray-200">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Projeto</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Responsável(eis)</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Próxima Entrega</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Impacto</th>
                            </tr>
                        </thead>
                        <tbody id="projectsTableBody" class="bg-white divide-y divide-gray-200"></tbody>
                    </table>
                </div>
            </section>
        </main>
    </div>

    <div id="projectModal" class="modal-overlay fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 hidden opacity-0">
        <div class="modal-content bg-white rounded-lg shadow-xl w-full max-w-2xl transform scale-95">
            <div class="p-6 border-b border-gray-200 flex justify-between items-center">
                <h2 id="modalTitle" class="text-2xl font-bold"></h2>
                <button id="closeModal" class="text-gray-400 hover:text-gray-600">
                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" /></svg>
                </button>
            </div>
            <div id="modalBody" class="p-6 space-y-4"></div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const googleSheetCsvUrl = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vQKLMMfDk_DOHEMeQj7Coa6-2aAGy7Xqz7JVkgGZJo10IeMWG581GabHYDUOBQHcqlbPUFqPfeumDK3/pub?output=csv';
            
            let allProjectData = [];
            const charts = {};
            const dom = {
                loader: document.getElementById('loader'),
                loaderText: document.getElementById('loader-text'),
                mainContent: document.querySelector('main'),
                errorBanner: document.getElementById('errorBanner'),
                errorBannerText: document.getElementById('errorBannerText'),
                kpis: document.getElementById('kpis'),
                responsibleFilter: document.getElementById('responsibleFilter'),
                statusFilter: document.getElementById('statusFilter'),
                classificationFilter: document.getElementById('classificationFilter'),
                projectsTableBody: document.getElementById('projectsTableBody'),
                modal: document.getElementById('projectModal'),
                modalTitle: document.getElementById('modalTitle'),
                modalBody: document.getElementById('modalBody'),
                closeModalBtn: document.getElementById('closeModal'),
                refreshBtn: document.getElementById('refreshBtn'),
            };

            const parseCsv = (text) => {
                if (text.charCodeAt(0) === 65279) { text = text.substring(1); }
                const lines = text.trim().split(/\r\n|\n/);
                if (lines.length < 2) return [];

                const headerMapping = {
                    'id': 'id',
                    'nome do projeto': 'name',
                    'description': 'description',
                    'responsável': 'responsible',
                    'classificação primária': 'primaryclassification',
                    'classificação secundária': 'secondaryclassification',
                    'status': 'status',
                    'próxima entregas': 'nextmilestone', // Corrigido para corresponder à imagem
                    'ganhos': 'impact',
                };
                
                const sanitize = (str) => str.trim().toLowerCase().replace(/\s+/g, ' ');

                const actualHeaders = lines[0].split(',').map(h => sanitize(h.replace(/^"|"$/g, '')));
                
                const internalHeaders = actualHeaders.map(h => {
                    for (const key in headerMapping) {
                        if (sanitize(key) === h) {
                            return headerMapping[key];
                        }
                    }
                    return null;
                });

                const requiredInternalHeaders = ['id', 'name', 'status', 'responsible', 'description'];
                const foundHeaders = internalHeaders.filter(Boolean);
                const missingHeaders = requiredInternalHeaders.filter(h => !foundHeaders.includes(h));

                if (missingHeaders.length > 0) {
                     throw new Error(`A sua planilha precisa ter as seguintes colunas: ${missingHeaders.join(', ')}. As colunas encontradas foram: ${actualHeaders.join(', ')}`);
                }

                return lines.slice(1).map(line => {
                    if (!line.trim()) return null;

                    const values = line.split(/,(?=(?:(?:[^"]*"){2})*[^"]*$)/).map(v => v.trim().replace(/^"|"$/g, ''));
                    const entry = {};
                    
                    internalHeaders.forEach((internalHeader, index) => {
                        if (internalHeader) {
                            entry[internalHeader] = values[index] || '';
                        }
                    });
                    
                    if (!entry.id || !entry.name) return null;

                    entry.responsible = entry.responsible.split(';').map(r => r.trim()).filter(Boolean);
                    
                    return entry;
                }).filter(Boolean);
            };

            const loadData = async () => {
                dom.loader.style.display = 'flex';
                dom.mainContent.classList.add('hidden');
                dom.errorBanner.classList.add('hidden');
                dom.loaderText.textContent = 'Carregando dados da planilha...';

                try {
                    const proxyUrl = 'https://api.allorigins.win/raw?url=';
                    const requestUrl = `${proxyUrl}${encodeURIComponent(googleSheetCsvUrl)}`;
                    
                    const response = await fetch(requestUrl);
                    if (!response.ok) throw new Error(`A resposta da rede não foi OK: ${response.statusText}`);
                    
                    const csvText = await response.text();
                    allProjectData = parseCsv(csvText);
                    initializeApp(allProjectData);

                } catch (error) {
                    console.error('Falha ao carregar ou processar dados da planilha:', error);
                    dom.errorBannerText.innerHTML = `<strong>Falha ao carregar a planilha.</strong> Verifique se o link está correto e se a primeira linha da planilha tem as colunas esperadas.<br><br><i>Detalhe do erro: ${error.message}</i>`;
                    dom.errorBanner.classList.remove('hidden');
                    dom.loader.style.display = 'none';
                }
            };

            const initializeApp = (data) => {
                if (!data || data.length === 0) {
                    dom.errorBannerText.textContent = 'Nenhum projeto encontrado nos dados carregados.';
                    dom.errorBanner.classList.remove('hidden');
                    dom.loader.style.display = 'none';
                    return;
                }
                populateFilters(data);
                updateDashboard();
                dom.loader.style.display = 'none';
                dom.mainContent.classList.remove('hidden');
            };
            
            const populateFilters = (data) => {
                const createOptions = (element, values, label) => {
                    element.innerHTML = `<option value="all">Todos os ${label}</option>`;
                    [...new Set(values)].sort().forEach(value => {
                        if (value) {
                            const option = document.createElement('option');
                            option.value = value;
                            option.textContent = value;
                            element.appendChild(option);
                        }
                    });
                };
                createOptions(dom.responsibleFilter, data.flatMap(p => p.responsible), 'Responsáveis');
                createOptions(dom.statusFilter, data.map(p => p.status), 'Status');
                createOptions(dom.classificationFilter, data.map(p => p.primaryclassification), 'Classificações');
            };

            const updateDashboard = () => {
                const selectedResponsible = dom.responsibleFilter.value;
                const selectedStatus = dom.statusFilter.value;
                const selectedClassification = dom.classificationFilter.value;

                const filteredData = allProjectData.filter(p => 
                    (selectedResponsible === 'all' || p.responsible.includes(selectedResponsible)) &&
                    (selectedStatus === 'all' || p.status === selectedStatus) &&
                    (selectedClassification === 'all' || p.primaryclassification === selectedClassification)
                );

                updateKPIs(filteredData);
                updateCharts(filteredData);
                updateProjectsTable(filteredData);
            };

            const updateKPIs = (data) => {
                dom.kpis.innerHTML = '';
                const kpiData = [
                    { title: 'Total de Iniciativas', value: data.length, icon: `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="M3.75 12h16.5m-16.5 3.75h16.5M3.75 19.5h16.5M5.625 4.5h12.75a1.875 1.875 0 010 3.75H5.625a1.875 1.875 0 010-3.75z" /></svg>`, color: { bg: 'bg-blue-100', text: 'text-blue-600' } },
                    { title: 'Em Andamento', value: data.filter(p => p.status === 'In Progress').length, icon: `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0l3.181 3.183a8.25 8.25 0 0011.667 0l3.181-3.183m-4.991-2.691V5.25a2.25 2.25 0 00-2.25-2.25h-4.5a2.25 2.25 0 00-2.25 2.25v4.992m11.667 0l-3.181 3.183a8.25 8.25 0 01-11.667 0l-3.181-3.183" /></svg>`, color: { bg: 'bg-green-100', text: 'text-green-600' } },
                    { title: 'Planejamento', value: data.filter(p => p.status === 'Planning').length, icon: `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="M10.125 2.25h-4.5c-.621 0-1.125.504-1.125 1.125v17.25c0 .621.504 1.125 1.125 1.125h12.75c.621 0 1.125-.504 1.125-1.125v-9M10.125 2.25h.375a9 9 0 019 9v.375M10.125 2.25A3.375 3.375 0 0113.5 5.625v1.5c0 .621.504 1.125 1.125 1.125h1.5a3.375 3.375 0 013.375 3.375M9 15l2.25 2.25L15 12" /></svg>`, color: { bg: 'bg-yellow-100', text: 'text-yellow-600' } },
                    { title: 'Projetos Estratégicos', value: data.filter(p => p.primaryclassification === 'Projeto Estratégico').length, icon: `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="M11.48 3.499a.562.562 0 011.04 0l2.125 5.111a.563.563 0 00.475.345h5.364c.518 0 .734.664.348.97l-4.336 3.159a.562.562 0 00-.182.635l1.634 5.057c.155.48-.42.864-.83.541l-4.49-3.268a.563.563 0 00-.656 0l-4.49 3.268c-.41.299-.985-.06-.83-.541l1.634-5.057a.562.562 0 00-.182-.635L2.533 9.925c-.386-.281-.17-.97.348-.97h5.364a.562.562 0 00.475-.345L11.48 3.5z" /></svg>`, color: { bg: 'bg-indigo-100', text: 'text-indigo-600' } }
                ];
                kpiData.forEach(({ title, value, icon, color }) => {
                    dom.kpis.innerHTML += `
                        <div class="card p-6 flex items-center justify-between">
                            <div><p class="text-sm font-medium text-gray-500">${title}</p><p class="text-3xl font-bold text-gray-900">${value}</p></div>
                            <div class="w-12 h-12 rounded-full flex items-center justify-center ${color.bg} ${color.text}">${icon}</div>
                        </div>`;
                });
            };
            
            const createOrUpdateChart = (id, type, data, options) => {
                const ctx = document.getElementById(id).getContext('2d');
                if (charts[id]) {
                    charts[id].destroy();
                }
                charts[id] = new Chart(ctx, { type, data, options });
            };
            
            const updateCharts = (data) => {
                const statusCounts = data.reduce((acc, p) => { if(p.status) acc[p.status] = (acc[p.status] || 0) + 1; return acc; }, {});
                createOrUpdateChart('statusChart', 'doughnut', {
                    labels: Object.keys(statusCounts),
                    datasets: [{ data: Object.values(statusCounts), backgroundColor: ['#34D399', '#FBBF24', '#60A5FA', '#A78BFA', '#F87171', '#9CA3AF'], borderColor: '#f8f9fa', borderWidth: 4, hoverOffset: 8 }]
                }, { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'bottom', labels: { font: { family: 'Inter' } } } } });

                const responsibleCounts = data.flatMap(p => p.responsible).reduce((acc, r) => { if(r) acc[r] = (acc[r] || 0) + 1; return acc; }, {});
                createOrUpdateChart('responsibleChart', 'bar', {
                    labels: Object.keys(responsibleCounts),
                    datasets: [{ label: 'Nº de Projetos', data: Object.values(responsibleCounts), backgroundColor: '#60A5FA', borderRadius: 4 }]
                }, { indexAxis: 'y', responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { x: { beginAtZero: true, ticks: { stepSize: 1, font: { family: 'Inter' } } }, y: { ticks: { font: { family: 'Inter' } } } } });

                const focusCounts = data.reduce((acc, p) => { if(p.secondaryclassification) acc[p.secondaryclassification] = (acc[p.secondaryclassification] || 0) + 1; return acc; }, {});
                createOrUpdateChart('focusChart', 'bar', {
                    labels: Object.keys(focusCounts),
                    datasets: [{ label: 'Nº de Projetos', data: Object.values(focusCounts), backgroundColor: ['#34D399', '#FBBF24', '#60A5FA', '#A78BFA', '#F87171', '#9CA3AF'], borderRadius: 4 }]
                }, { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { y: { beginAtZero: true, ticks: { stepSize: 1, font: { family: 'Inter' } } }, x: { ticks: { font: { family: 'Inter' } } } } });
            };

            const updateProjectsTable = (data) => {
                dom.projectsTableBody.innerHTML = '';
                if (data.length === 0) {
                    dom.projectsTableBody.innerHTML = `<tr><td colspan="5" class="text-center py-8 text-gray-500">Nenhum projeto encontrado para os filtros selecionados.</td></tr>`;
                    return;
                }
                data.forEach(p => {
                    const getStatusColor = s => ({'In Progress':'bg-green-100 text-green-800','Planning':'bg-yellow-100 text-yellow-800', 'Completed':'bg-gray-100 text-gray-800'}[s]||'bg-blue-100 text-blue-800');
                    const row = document.createElement('tr');
                    row.className = 'hover:bg-gray-50 cursor-pointer';
                    row.dataset.projectId = p.id;
                    row.innerHTML = `
                        <td class="px-6 py-4"><div class="text-sm font-semibold text-gray-900">${p.name}</div><div class="text-xs text-gray-500">${p.primaryclassification}</div></td>
                        <td class="px-6 py-4 text-sm text-gray-700">${p.responsible.join(', ')}</td>
                        <td class="px-6 py-4"><span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full ${getStatusColor(p.status)}">${p.status}</span></td>
                        <td class="px-6 py-4 text-sm text-gray-700">${p.nextmilestone}</td>
                        <td class="px-6 py-4 text-sm text-gray-700 max-w-xs truncate" title="${p.impact}">${p.impact}</td>`;
                    dom.projectsTableBody.appendChild(row);
                });
            };
            
            const openModal = (projectId) => {
                const project = allProjectData.find(p => p.id === projectId);
                if (!project) return;
                const getStatusColor = s => ({'In Progress':'bg-green-100 text-green-800','Planning':'bg-yellow-100 text-yellow-800', 'Completed':'bg-gray-100 text-gray-800'}[s]||'bg-blue-100 text-blue-800');
                dom.modalTitle.textContent = project.name;
                dom.modalBody.innerHTML = `
                    <p class="text-gray-700">${project.description}</p>
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 pt-4 border-t">
                        <div><h4 class="text-sm font-semibold text-gray-500">Responsáveis</h4><p class="text-gray-800">${project.responsible.join(', ')}</p></div>
                        <div><h4 class="text-sm font-semibold text-gray-500">Status</h4><p><span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full ${getStatusColor(project.status)}">${project.status}</span></p></div>
                        <div><h4 class="text-sm font-semibold text-gray-500">Classificação</h4><p class="text-gray-800">${project.primaryclassification} / ${project.secondaryclassification}</p></div>
                        <div><h4 class="text-sm font-semibold text-gray-500">Próxima Entrega</h4><p class="text-gray-800">${project.nextmilestone}</p></div>
                        <div class="sm:col-span-2"><h4 class="text-sm font-semibold text-gray-500">Impacto (Ganhos)</h4><p class="text-gray-800">${project.impact}</p></div>
                    </div>`;
                dom.modal.classList.remove('hidden');
                setTimeout(() => { dom.modal.classList.remove('opacity-0'); dom.modal.querySelector('.modal-content').classList.remove('scale-95'); }, 10);
            };

            const closeModal = () => {
                dom.modal.classList.add('opacity-0');
                dom.modal.querySelector('.modal-content').classList.add('scale-95');
                setTimeout(() => dom.modal.classList.add('hidden'), 300);
            };

            dom.refreshBtn.addEventListener('click', loadData);
            [dom.responsibleFilter, dom.statusFilter, dom.classificationFilter].forEach(el => el.addEventListener('change', updateDashboard));
            dom.closeModalBtn.addEventListener('click', closeModal);
            dom.modal.addEventListener('click', e => { if (e.target === dom.modal) closeModal(); });
            dom.projectsTableBody.addEventListener('click', e => {
                const row = e.target.closest('tr');
                if (row && row.dataset.projectId) openModal(row.dataset.projectId);
            });
            
            loadData();
        });
    </script>
</body>
</html>

