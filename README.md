<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard de Governança de Projetos | Suporte N2/N3</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f8f9fa; }
        .chart-container { position: relative; height: 300px; width: 100%; }
        .card { background-color: #ffffff; border-radius: 0.75rem; box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1); }
        .modal-overlay { transition: opacity 0.3s ease; }
        .modal-content { transition: transform 0.3s ease; }
        #loader { border-top-color: #3498db; animation: spin 1s linear infinite; }
        @keyframes spin { to { transform: rotate(360deg); } }
    </style>
</head>
<body class="text-gray-800">

    <div id="loader" class="fixed inset-0 bg-white z-50 flex flex-col items-center justify-center text-center p-4">
        <div class="w-16 h-16 border-4 border-gray-200 border-t-blue-500 rounded-full animate-spin"></div>
        <p class="mt-4 text-gray-600 font-medium">Carregando dados...</p>
    </div>

    <div id="error-container" class="hidden container mx-auto p-4 sm:p-6 lg:p-8">
         <div class="bg-red-100 border-l-4 border-red-500 text-red-700 p-4" role="alert">
            <p id="error-message" class="text-sm"></p>
        </div>
    </div>

    <div id="dashboard-container" class="hidden container mx-auto p-4 sm:p-6 lg:p-8">
        <header class="mb-8 flex justify-between items-center">
            <div>
                <h1 class="text-3xl font-bold text-gray-900">Dashboard de Governança de Projetos</h1>
                <p class="text-md text-gray-600 mt-1">Acompanhamento de iniciativas do time.</p>
            </div>
            <button id="refreshBtn" title="Atualizar Dados da Planilha" class="text-gray-500 hover:text-gray-800 p-2 rounded-full hover:bg-gray-200 transition">
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0l3.181 3.183a8.25 8.25 0 0011.667 0l3.181-3.183m-4.991-2.691V5.25a2.25 2.25 0 00-2.25-2.25h-4.5a2.25 2.25 0 00-2.25 2.25v4.992m11.667 0l-3.181 3.183a8.25 8.25 0 01-11.667 0l-3.181-3.183" /></svg>
            </button>
        </header>

        <main>
            <section id="kpis" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-8"></section>
            
            <section id="charts" class="grid grid-cols-1 lg:grid-cols-2 gap-8 mb-8">
                <div class="card p-6">
                    <h3 class="text-xl font-semibold text-center mb-4">Projetos por Status</h3>
                    <div class="chart-container"><canvas id="statusChart"></canvas></div>
                </div>
                <div class="card p-6">
                    <h3 class="text-xl font-semibold text-center mb-4">Projetos por Responsável</h3>
                    <div class="chart-container"><canvas id="responsibleChart"></canvas></div>
                </div>
            </section>

            <section class="card p-6">
                <h3 class="text-xl font-semibold mb-4">Detalhamento dos Projetos</h3>
                <div class="overflow-x-auto">
                    <table class="min-w-full divide-y divide-gray-200">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Projeto</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Responsável</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
                            </tr>
                        </thead>
                        <tbody id="projects-table-body" class="bg-white divide-y divide-gray-200"></tbody>
                    </table>
                </div>
            </section>
        </main>
    </div>
    
    <div id="project-modal" class="modal-overlay fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 hidden opacity-0">
        <div class="modal-content bg-white rounded-lg shadow-xl w-full max-w-2xl transform scale-95">
            <div class="p-6 border-b flex justify-between items-center">
                <h2 id="modal-title" class="text-2xl font-bold"></h2>
                <button id="modal-close-btn" class="text-gray-400 hover:text-gray-600">&times;</button>
            </div>
            <div id="modal-body" class="p-6"></div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // ---> INSTRUÇÃO: Cole o link da sua planilha (publicada como CSV) aqui <---
            const GOOGLE_SHEET_URL = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vQKLMMfDk_DOHEMeQj7Coa6-2aAGy7Xqz7JVkgGZJo10IeMWG581GabHYDUOBQHcqlbPUFqPfeumDK3/pub?output=csv';

            let allProjectData = [];
            const charts = {};
            const dom = {
                loader: document.getElementById('loader'),
                errorContainer: document.getElementById('error-container'),
                errorMessage: document.getElementById('error-message'),
                dashboardContainer: document.getElementById('dashboard-container'),
                kpis: document.getElementById('kpis'),
                tableBody: document.getElementById('projects-table-body'),
                modal: document.getElementById('project-modal'),
                modalTitle: document.getElementById('modal-title'),
                modalBody: document.getElementById('modal-body'),
                modalCloseBtn: document.getElementById('modal-close-btn'),
                refreshBtn: document.getElementById('refreshBtn'),
            };

            const STATUS_TRANSLATIONS = {
                'Not Started': 'Não Iniciado', 'Planning': 'Planejamento', 'In Progress': 'Em Andamento',
                'Completed': 'Concluído', 'Closed': 'Encerrado', 'At Risk': 'Em Risco'
            };

            const COLUMN_MAPPING = {
                'id': 'id', 'nome do projeto': 'name', 'responsável': 'responsible', 'status': 'status',
                'descrição': 'description', 'classificação primária': 'classification1'
            };

            const parseCSV = (text) => {
                const lines = text.trim().split(/\r\n|\n/);
                const headers = lines[0].split(',').map(h => (h || '').trim().toLowerCase());
                
                const requiredHeaders = ['id', 'nome do projeto', 'responsável', 'status'];
                const missingHeaders = requiredHeaders.filter(rh => !headers.includes(rh));
                if (missingHeaders.length > 0) {
                    throw new Error(`COLUNAS FALTANDO: A planilha precisa ter as seguintes colunas: ${missingHeaders.join(', ')}.`);
                }
                
                return lines.slice(1).map(line => {
                    const values = line.split(/,(?=(?:(?:[^"]*"){2})*[^"]*$)/);
                    const entry = {};
                    headers.forEach((header, index) => {
                        const internalKey = COLUMN_MAPPING[header];
                        if (internalKey) {
                            entry[internalKey] = (values[index] || '').trim().replace(/^"|"$/g, '');
                        }
                    });
                    return entry;
                }).filter(entry => entry.id && entry.name);
            };

            const fetchData = async () => {
                dom.loader.style.display = 'flex';
                dom.dashboardContainer.classList.add('hidden');
                
                try {
                    const response = await fetch(GOOGLE_SHEET_URL);
                    if (!response.ok) throw new Error('Falha na conexão com a planilha.');
                    const csvText = await response.text();
                    
                    if (csvText.trim().toLowerCase().includes('<!doctype html')) {
                         throw new Error('PERMISSÃO NECESSÁRIA: A planilha não está pública. Em "Compartilhar", defina o "Acesso geral" para "Qualquer pessoa com o link".');
                    }

                    return parseCSV(csvText);
                } catch (error) {
                    console.error("Erro ao carregar dados:", error);
                    showError(error.message);
                    return getLocalData(); // Retorna dados locais como fallback
                }
            };

            const render = (data) => {
                renderKPIs(data);
                renderStatusChart(data);
                renderResponsibleChart(data);
                renderTable(data);
                dom.loader.style.display = 'none';
                dom.dashboardContainer.classList.remove('hidden');
            };

            const renderKPIs = (data) => {
                const total = data.length;
                const inProgress = data.filter(p => p.status === 'In Progress').length;
                const planning = data.filter(p => p.status === 'Planning').length;
                const completed = data.filter(p => ['Completed', 'Closed'].includes(p.status)).length;
                
                dom.kpis.innerHTML = `
                    <div class="card p-6"><p class="text-sm text-gray-500">Total de Projetos</p><p class="text-3xl font-bold">${total}</p></div>
                    <div class="card p-6"><p class="text-sm text-gray-500">Em Andamento</p><p class="text-3xl font-bold">${inProgress}</p></div>
                    <div class="card p-6"><p class="text-sm text-gray-500">Em Planejamento</p><p class="text-3xl font-bold">${planning}</p></div>
                    <div class="card p-6"><p class="text-sm text-gray-500">Concluídos</p><p class="text-3xl font-bold">${completed}</p></div>
                `;
            };
            
            const createOrUpdateChart = (id, type, data, options) => {
                const ctx = document.getElementById(id).getContext('2d');
                if (charts[id]) charts[id].destroy();
                charts[id] = new Chart(ctx, { type, data, options });
            };

            const renderStatusChart = (data) => {
                const counts = data.reduce((acc, p) => {
                    const status = STATUS_TRANSLATIONS[p.status] || p.status;
                    if(status) acc[status] = (acc[status] || 0) + 1;
                    return acc;
                }, {});
                createOrUpdateChart('statusChart', 'doughnut', {
                    labels: Object.keys(counts),
                    datasets: [{ data: Object.values(counts), backgroundColor: ['#34D399', '#FBBF24', '#60A5FA', '#A78BFA', '#F87171', '#9CA3AF'], borderWidth: 0 }]
                }, { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'bottom' } } });
            };

            const renderResponsibleChart = (data) => {
                const counts = data.reduce((acc, p) => {
                    if(p.responsible) acc[p.responsible] = (acc[p.responsible] || 0) + 1;
                    return acc;
                }, {});
                createOrUpdateChart('responsibleChart', 'bar', {
                    labels: Object.keys(counts),
                    datasets: [{ label: 'Nº de Projetos', data: Object.values(counts), backgroundColor: '#60A5FA' }]
                }, { indexAxis: 'y', responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { x: { ticks: { stepSize: 1 } } } });
            };

            const renderTable = (data) => {
                dom.tableBody.innerHTML = '';
                data.forEach(p => {
                    const row = document.createElement('tr');
                    row.className = 'hover:bg-gray-50 cursor-pointer';
                    row.dataset.id = p.id;
                    row.innerHTML = `
                        <td class="px-6 py-4 font-semibold">${p.name}</td>
                        <td class="px-6 py-4">${p.responsible}</td>
                        <td class="px-6 py-4">${STATUS_TRANSLATIONS[p.status] || p.status}</td>`;
                    dom.tableBody.appendChild(row);
                });
            };

            const openModal = (project) => {
                if (!project) return;
                dom.modalTitle.textContent = project.name;
                dom.modalBody.innerHTML = `
                    <p class="text-base text-gray-700">${project.description || '<i>Sem descrição disponível.</i>'}</p>
                    <div class="mt-4 pt-4 border-t grid grid-cols-2 gap-4">
                        <div><h4 class="font-semibold text-sm text-gray-500">Responsável</h4><p>${project.responsible}</p></div>
                        <div><h4 class="font-semibold text-sm text-gray-500">Status</h4><p>${STATUS_TRANSLATIONS[project.status] || project.status}</p></div>
                        <div><h4 class="font-semibold text-sm text-gray-500">Classificação</h4><p>${project.classification1 || 'N/A'}</p></div>
                    </div>`;
                dom.modal.classList.remove('hidden');
                setTimeout(() => dom.modal.classList.remove('opacity-0'), 10);
            };

            const closeModal = () => {
                dom.modal.classList.add('opacity-0');
                setTimeout(() => dom.modal.classList.add('hidden'), 300);
            };

            const showError = (message) => {
                dom.loader.style.display = 'none';
                dom.errorMessage.innerHTML = message.replace(/\n/g, '<br>');
                dom.errorContainer.classList.remove('hidden');
            };
            
            const getLocalData = () => [
                {id: 'EX1', name: 'Projeto de Exemplo 1', responsible: 'Equipe', status: 'In Progress', description: 'Este é um projeto de exemplo carregado localmente.'},
                {id: 'EX2', name: 'Projeto de Exemplo 2', responsible: 'Equipe', status: 'Planning', description: 'Os dados da sua planilha não puderam ser carregados.'}
            ];
            
            dom.refreshBtn.addEventListener('click', () => fetchData().then(data => data && render(data)));
            dom.tableBody.addEventListener('click', e => {
                const row = e.target.closest('tr');
                if (row && row.dataset.id) openModal(allProjectData.find(p => p.id === row.dataset.id));
            });
            dom.modalCloseBtn.addEventListener('click', closeModal);
            dom.modal.addEventListener('click', e => { if (e.target === dom.modal) closeModal(); });

            fetchData().then(data => {
                allProjectData = data;
                if (data) render(data);
            });
        });
    </script>
</body>
</html>
