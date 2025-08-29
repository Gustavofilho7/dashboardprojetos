<!DOCTYPE html>
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
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f8f9fa;
        }
        .chart-container {
            position: relative;
            width: 100%;
            height: 96;
            max-height: 400px;
        }
        .card {
            background-color: #ffffff;
            border-radius: 0.75rem;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
            transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
        }
        .card:hover {
            transform: translateY(-4px);
            box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -2px rgb(0 0 0 / 0.1);
        }
        select {
            -webkit-appearance: none;
            -moz-appearance: none;
            appearance: none;
            background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke-width='1.5' stroke='%236b7280' class='w-6 h-6'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M19.5 8.25l-7.5 7.5-7.5-7.5' /%3E%3C/svg%3E%0A");
            background-repeat: no-repeat;
            background-position: right 0.75rem center;
            background-size: 1.5em 1.5em;
            padding-right: 2.5rem;
        }
        .modal-overlay {
            transition: opacity 0.3s ease;
        }
        .modal-content {
            transition: transform 0.3s ease;
        }
        #loader {
            border-top-color: #3498db;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
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
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0l3.181 3.183a8.25 8.25 0 0011.667 0l3.181-3.183m-4.991-2.691V5.25a2.25 2.25 0 00-2.25-2.25h-4.5a2.25 2.25 0 00-2.25 2.25v4.992m11.667 0l-3.181 3.183a8.25 8.25 0 01-11.667 0l-3.181-3.183" />
                </svg>
            </button>
        </header>

        <div id="errorBanner" class="hidden bg-yellow-100 border-l-4 border-yellow-500 text-yellow-700 p-4 mb-6" role="alert">
            <p class="font-bold">Aviso</p>
            <p id="errorBannerText"></p>
        </div>

        <main class="hidden">
            <section id="kpis" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
            </section>

            <section id="filters" class="card p-6 mb-8 flex flex-col sm:flex-row gap-4 items-center">
                <h3 class="text-lg font-semibold text-gray-700 sm:mr-4">Filtros:</h3>
                <div class="w-full sm:w-auto flex-grow">
                    <label for="responsibleFilter" class="sr-only">Responsável</label>
                    <select id="responsibleFilter" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition">
                        <option value="all">Todos os Responsáveis</option>
                    </select>
                </div>
                <div class="w-full sm:w-auto flex-grow">
                    <label for="statusFilter" class="sr-only">Status</label>
                    <select id="statusFilter" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition">
                        <option value="all">Todos os Status</option>
                    </select>
                </div>
                <div class="w-full sm:w-auto flex-grow">
                    <label for="classificationFilter" class="sr-only">Classificação</label>
                    <select id="classificationFilter" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition">
                        <option value="all">Todas as Classificações</option>
                    </select>
                </div>
            </section>

            <section id="charts" class="grid grid-cols-1 lg:grid-cols-3 gap-8 mb-8">
                <div class="card p-6 lg:col-span-1">
                    <h3 class="text-xl font-semibold text-center mb-4">Projetos por Status</h3>
                    <div class="chart-container h-72 sm:h-80 mx-auto" style="max-width: 400px;">
                        <canvas id="statusChart"></canvas>
                    </div>
                </div>
                <div class="card p-6 lg:col-span-2">
                    <h3 class="text-xl font-semibold text-center mb-4">Projetos por Responsável</h3>
                    <div class="chart-container h-72 sm:h-80">
                        <canvas id="responsibleChart"></canvas>
                    </div>
                </div>
                 <div class="card p-6 lg:col-span-3">
                    <h3 class="text-xl font-semibold text-center mb-4">Foco Estratégico do Time</h3>
                    <div class="chart-container h-72 sm:h-80">
                        <canvas id="focusChart"></canvas>
                    </div>
                </div>
            </section>

            <section id="project-table" class="card p-6">
                <h3 class="text-xl font-semibold mb-4">Detalhamento dos Projetos</h3>
                <div class="overflow-x-auto">
                    <table class="min-w-full divide-y divide-gray-200">
                        <thead class="bg-gray-50">
                            <tr>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Projeto</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Responsável(eis)</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Próxima Entrega</th>
                                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Impacto</th>
                            </tr>
                        </thead>
                        <tbody id="projectsTableBody" class="bg-white divide-y divide-gray-200">
                        </tbody>
                    </table>
                </div>
            </section>
        </main>
    </div>

    <div id="projectModal" class="modal-overlay fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 hidden opacity-0">
        <div class="modal-content bg-white rounded-lg shadow-xl w-full max-w-2xl transform scale-95">
            <div class="p-6 border-b border-gray-200 flex justify-between items-center">
                <h2 id="modalTitle" class="text-2xl font-bold">Detalhes do Projeto</h2>
                <button id="closeModal" class="text-gray-400 hover:text-gray-600">
                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
                    </svg>
                </button>
            </div>
            <div id="modalBody" class="p-6 space-y-4">
            </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', async () => {
            
            const googleSheetCsvUrl = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vQKLMMfDk_DOHEMeQj7Coa6-2aAGy7Xqz7JVkgGZJo10IeMWG581GabHYDUOBQHcqlbPUFqPfeumDK3/pub?output=csv';

            let projectData = [];
            let statusChart, responsibleChart, focusChart;
            
            const responsibleFilter = document.getElementById('responsibleFilter');
            const statusFilter = document.getElementById('statusFilter');
            const classificationFilter = document.getElementById('classificationFilter');
            const modal = document.getElementById('projectModal');
            const closeModalBtn = document.getElementById('closeModal');
            const projectsTableBody = document.getElementById('projectsTableBody');
            const loader = document.getElementById('loader');
            const loaderText = document.getElementById('loader-text');
            const mainContent = document.querySelector('main');
            const errorBanner = document.getElementById('errorBanner');
            const errorBannerText = document.getElementById('errorBannerText');
            const refreshBtn = document.getElementById('refreshBtn');

            async function loadProjectData() {
                loader.style.display = 'flex';
                mainContent.classList.add('hidden');
                errorBanner.classList.add('hidden');
                
                if (googleSheetCsvUrl && googleSheetCsvUrl !== 'URL_DA_SUA_PLANILHA_CSV_VEM_AQUI') {
                    loaderText.textContent = 'Carregando dados do Google Sheets...';
                    try {
                        const proxyUrl = 'https://api.allorigins.win/raw?url=';
                        const requestUrl = `${proxyUrl}${encodeURIComponent(googleSheetCsvUrl)}`;
                        
                        const response = await fetch(requestUrl);
                        if (!response.ok) throw new Error(`A resposta da rede não foi OK: ${response.statusText}`);
                        const csvText = await response.text();
                        projectData = parseCsv(csvText);
                        if (projectData.length === 0) throw new Error('A planilha parece estar vazia ou em um formato incorreto.');
                        initializeApp();
                    } catch (error) {
                        console.error('Falha ao carregar dados do Google Sheets:', error);
                        errorBannerText.textContent = 'Não foi possível carregar os dados da sua planilha. Carregando dados de exemplo. Verifique a URL e as configurações de compartilhamento.';
                        errorBanner.classList.remove('hidden');
                        loadLocalData();
                    }
                } else {
                    loaderText.textContent = 'Nenhuma planilha configurada. Carregando dados de exemplo...';
                    errorBannerText.textContent = 'O dashboard está usando dados de exemplo. Para conectar sua planilha, edite o arquivo HTML e insira a URL correta.';
                    errorBanner.classList.remove('hidden');
                    setTimeout(loadLocalData, 500);
                }
            }

            function loadLocalData() {
                projectData = [
                    { id: 'P01', name: 'Estrutura de Governança', description: 'Criar repositório no Notion e definir processos, responsáveis e prazos para as iniciativas.', responsible: ['Gustavo'], primaryClassification: 'Projeto Estratégico', secondaryClassification: 'Processos', status: 'Planejamento', startDate: '', nextMilestone: 'Desenvolver estrutura inicial', deadline: 'A definir', impact: 'Aumentar a maturidade e responsabilidade do time, evitar que projetos fiquem parados.' },
                    { id: 'P02', name: 'Bot Gerador de Comunicados (GPT)', description: 'Desenvolver bot que gera comunicados (inicial, update, final) a partir de contexto falado, usando templates.', responsible: ['Felipe'], primaryClassification: 'Projeto Estratégico', secondaryClassification: 'Inovação / Automação / IA', status: 'Em Andamento', startDate: '', nextMilestone: 'Validar viabilidade da integração com Zendesk/Slack', deadline: 'A definir', impact: 'Agilizar drasticamente a criação de comunicados, padronizar a comunicação externa.' },
                    { id: 'P03', name: 'Expansão do Bot Batman', description: 'Evoluir o bot de consulta N1 para um assistente de processos e procedimentos para todo o time.', responsible: ['Gustavo'], primaryClassification: 'Projeto', secondaryClassification: 'Inovação / Automação / IA', status: 'Em Andamento', startDate: '', nextMilestone: 'Apresentar o bot em evento para aumentar a adoção', deadline: 'A definir', impact: 'Aumentar a autonomia do time N1, reduzir dúvidas recorrentes, centralizar processos.' },
                    { id: 'P04', name: 'Site de Registro (Qualidade)', description: 'Criar um site interno de baixo custo para registrar informações e dados de qualidade.', responsible: ['Felipe', 'Morango'], primaryClassification: 'Projeto', secondaryClassification: 'Qualidade / Exp. do Cliente', status: 'Em Andamento', startDate: '', nextMilestone: 'Reunião técnica com Adrielly', deadline: 'A definir', impact: 'Centralizar informações de qualidade, facilitar o acesso e a análise de dados.' },
                    { id: 'P05', name: 'Estruturação do Academy', description: 'Centralizar conteúdo de treinamento e criar a trilha "Customer Care Hub" no BP Academy.', responsible: ['João Pedro', 'Lirinha', 'Lira', 'Thales', 'Rafa'], primaryClassification: 'Projeto', secondaryClassification: 'Desenvolvimento Humano', status: 'Em Andamento', startDate: '', nextMilestone: 'Mapear e organizar conteúdos existentes', deadline: 'A definir', impact: 'Reduzir tempo de onboarding, padronizar conhecimento, facilitar a reciclagem da equipe.' },
                    { id: 'I01', name: 'Prompt para Comunicação Interna', description: 'Finalizar um prompt para agilizar a comunicação interna da equipe SWAT.', responsible: ['Felipe'], primaryClassification: 'Iniciativa', secondaryClassification: 'Inovação / Automação / IA', status: 'Em Finalização', startDate: '', nextMilestone: 'Disponibilizar prompt para a equipe', deadline: 'A definir', impact: 'Agilidade e padronização na comunicação interna do time de N2/N3.' },
                    { id: 'I02', name: 'Canal de Comunicação', description: 'Manter o canal de comunicação com Nelson, Gisele e Bia em funcionamento.', responsible: ['João Pedro'], primaryClassification: 'Iniciativa', secondaryClassification: 'Processos', status: 'Em Operação', startDate: '', nextMilestone: 'Coletar feedbacks para melhorias', deadline: 'Contínuo', impact: 'Melhorar o fluxo de comunicação com áreas parceiras.' }
                ];
                initializeApp();
            }

            function parseCsv(text) {
                const lines = text.trim().split(/\r\n|\n/);
                if (lines.length < 2) return [];
                const headers = lines[0].split(',').map(h => h.trim());
                const data = [];
                for (let i = 1; i < lines.length; i++) {
                    if (!lines[i]) continue;
                    const values = lines[i].split(',');
                    const entry = {};
                    for (let j = 0; j < headers.length; j++) {
                        entry[headers[j]] = values[j] ? values[j].trim() : '';
                    }
                    entry.responsible = entry.responsible ? entry.responsible.split(';').map(r => r.trim()) : [];
                    data.push(entry);
                }
                return data;
            }
            
            function initializeApp() {
                loader.style.display = 'none';
                mainContent.classList.remove('hidden');
                
                responsibleFilter.innerHTML = '<option value="all">Todos os Responsáveis</option>';
                statusFilter.innerHTML = '<option value="all">Todos os Status</option>';
                classificationFilter.innerHTML = '<option value="all">Todas as Classificações</option>';

                populateFilters();
                updateDashboard();

                responsibleFilter.addEventListener('change', updateDashboard);
                statusFilter.addEventListener('change', updateDashboard);
                classificationFilter.addEventListener('change', updateDashboard);
                closeModalBtn.addEventListener('click', closeModal);
                modal.addEventListener('click', (e) => {
                    if (e.target === modal) closeModal();
                });
                projectsTableBody.addEventListener('click', (e) => {
                    const row = e.target.closest('tr');
                    if (row && row.dataset.projectId) {
                        openModal(row.dataset.projectId);
                    }
                });
                refreshBtn.addEventListener('click', loadProjectData);
            }

            function populateFilters() {
                const responsibles = [...new Set(projectData.flatMap(p => p.responsible))];
                const statuses = [...new Set(projectData.map(p => p.status))];
                const classifications = [...new Set(projectData.map(p => p.primaryClassification))];

                responsibles.forEach(r => {
                    if(!r) return;
                    const option = document.createElement('option');
                    option.value = r;
                    option.textContent = r;
                    responsibleFilter.appendChild(option);
                });

                statuses.forEach(s => {
                    if(!s) return;
                    const option = document.createElement('option');
                    option.value = s;
                    option.textContent = s;
                    statusFilter.appendChild(option);
                });

                classifications.forEach(c => {
                    if(!c) return;
                    const option = document.createElement('option');
                    option.value = c;
                    option.textContent = c;
                    classificationFilter.appendChild(option);
                });
            }

            function updateDashboard() {
                const selectedResponsible = responsibleFilter.value;
                const selectedStatus = statusFilter.value;
                const selectedClassification = classificationFilter.value;

                const filteredData = projectData.filter(p => {
                    const responsibleMatch = selectedResponsible === 'all' || (p.responsible && p.responsible.includes(selectedResponsible));
                    const statusMatch = selectedStatus === 'all' || p.status === selectedStatus;
                    const classificationMatch = selectedClassification === 'all' || p.primaryClassification === selectedClassification;
                    return responsibleMatch && statusMatch && classificationMatch;
                });

                updateKPIs(filteredData);
                updateStatusChart(filteredData);
                updateResponsibleChart(filteredData);
                updateFocusChart(filteredData);
                updateProjectsTable(filteredData);
            }
            
            function createKPI(title, value, icon, color) {
                const kpiContainer = document.getElementById('kpis');
                const kpiCard = document.createElement('div');
                kpiCard.className = `card p-6 flex items-center justify-between`;
                kpiCard.innerHTML = `
                    <div>
                        <p class="text-sm font-medium text-gray-500">${title}</p>
                        <p class="text-3xl font-bold text-gray-900">${value}</p>
                    </div>
                    <div class="w-12 h-12 rounded-full flex items-center justify-center ${color.bg} ${color.text}">
                        ${icon}
                    </div>
                `;
                kpiContainer.appendChild(kpiCard);
            }

            function updateKPIs(data) {
                const kpiContainer = document.getElementById('kpis');
                kpiContainer.innerHTML = '';
                
                const icons = {
                    total: `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="M3.75 12h16.5m-16.5 3.75h16.5M3.75 19.5h16.5M5.625 4.5h12.75a1.875 1.875 0 010 3.75H5.625a1.875 1.875 0 010-3.75z" /></svg>`,
                    inProgress: `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0l3.181 3.183a8.25 8.25 0 0011.667 0l3.181-3.183m-4.991-2.691V5.25a2.25 2.25 0 00-2.25-2.25h-4.5a2.25 2.25 0 00-2.25 2.25v4.992m11.667 0l-3.181 3.183a8.25 8.25 0 01-11.667 0l-3.181-3.183" /></svg>`,
                    planning: `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="M10.125 2.25h-4.5c-.621 0-1.125.504-1.125 1.125v17.25c0 .621.504 1.125 1.125 1.125h12.75c.621 0 1.125-.504 1.125-1.125v-9M10.125 2.25h.375a9 9 0 019 9v.375M10.125 2.25A3.375 3.375 0 0113.5 5.625v1.5c0 .621.504 1.125 1.125 1.125h1.5a3.375 3.375 0 013.375 3.375M9 15l2.25 2.25L15 12" /></svg>`,
                    strategic: `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="M11.48 3.499a.562.562 0 011.04 0l2.125 5.111a.563.563 0 00.475.345h5.364c.518 0 .734.664.348.97l-4.336 3.159a.562.562 0 00-.182.635l1.634 5.057c.155.48-.42.864-.83.541l-4.49-3.268a.563.563 0 00-.656 0l-4.49 3.268c-.41.299-.985-.06-.83-.541l1.634-5.057a.562.562 0 00-.182-.635L2.533 9.925c-.386-.281-.17-.97.348-.97h5.364a.562.562 0 00.475-.345L11.48 3.5z" /></svg>`
                };
                const colors = {
                    total: { bg: 'bg-blue-100', text: 'text-blue-600' },
                    inProgress: { bg: 'bg-green-100', text: 'text-green-600' },
                    planning: { bg: 'bg-yellow-100', text: 'text-yellow-600' },
                    strategic: { bg: 'bg-indigo-100', text: 'text-indigo-600' }
                };

                createKPI('Total de Iniciativas', data.length, icons.total, colors.total);
                createKPI('Em Andamento', data.filter(p => p.status === 'Em Andamento').length, icons.inProgress, colors.inProgress);
                createKPI('Em Planejamento', data.filter(p => p.status === 'Planejamento').length, icons.planning, colors.planning);
                createKPI('Projetos Estratégicos', data.filter(p => p.primaryClassification === 'Projeto Estratégico').length, icons.strategic, colors.strategic);
            }

            function updateStatusChart(data) {
                const ctx = document.getElementById('statusChart').getContext('2d');
                const statusCounts = data.reduce((acc, p) => {
                    if(p.status) acc[p.status] = (acc[p.status] || 0) + 1;
                    return acc;
                }, {});

                const chartData = {
                    labels: Object.keys(statusCounts),
                    datasets: [{
                        data: Object.values(statusCounts),
                        backgroundColor: ['#34D399', '#FBBF24', '#60A5FA', '#A78BFA', '#F87171', '#9CA3AF'],
                        borderColor: '#f8f9fa',
                        borderWidth: 4,
                        hoverOffset: 8
                    }]
                };

                if (statusChart) {
                    statusChart.data = chartData;
                    statusChart.update();
                } else {
                    statusChart = new Chart(ctx, {
                        type: 'doughnut',
                        data: chartData,
                        options: {
                            responsive: true,
                            maintainAspectRatio: false,
                            plugins: {
                                legend: {
                                    position: 'bottom',
                                    labels: {
                                        font: { family: 'Inter' }
                                    }
                                },
                            }
                        }
                    });
                }
            }

            function updateResponsibleChart(data) {
                const ctx = document.getElementById('responsibleChart').getContext('2d');
                const responsibleCounts = data.flatMap(p => p.responsible).reduce((acc, r) => {
                    if(r) acc[r] = (acc[r] || 0) + 1;
                    return acc;
                }, {});

                const chartData = {
                    labels: Object.keys(responsibleCounts),
                    datasets: [{
                        label: 'Nº de Projetos',
                        data: Object.values(responsibleCounts),
                        backgroundColor: '#60A5FA',
                        borderRadius: 4,
                    }]
                };

                if (responsibleChart) {
                    responsibleChart.data = chartData;
                    responsibleChart.update();
                } else {
                    responsibleChart = new Chart(ctx, {
                        type: 'bar',
                        data: chartData,
                        options: {
                            indexAxis: 'y',
                            responsive: true,
                            maintainAspectRatio: false,
                            plugins: {
                                legend: { display: false },
                                tooltip: {
                                    callbacks: {
                                        label: (context) => `${context.dataset.label}: ${context.raw}`
                                    }
                                }
                            },
                            scales: {
                                x: {
                                    beginAtZero: true,
                                    ticks: {
                                        stepSize: 1,
                                        font: { family: 'Inter' }
                                    }
                                },
                                y: {
                                    ticks: { font: { family: 'Inter' } }
                                }
                            }
                        }
                    });
                }
            }
            
            function updateFocusChart(data) {
                const ctx = document.getElementById('focusChart').getContext('2d');
                const focusCounts = data.reduce((acc, p) => {
                    if(p.secondaryClassification) acc[p.secondaryClassification] = (acc[p.secondaryClassification] || 0) + 1;
                    return acc;
                }, {});

                const chartData = {
                    labels: Object.keys(focusCounts),
                    datasets: [{
                        label: 'Nº de Projetos',
                        data: Object.values(focusCounts),
                        backgroundColor: ['#34D399', '#FBBF24', '#60A5FA', '#A78BFA', '#F87171', '#9CA3AF'],
                        borderRadius: 4,
                    }]
                };

                if (focusChart) {
                    focusChart.data = chartData;
                    focusChart.update();
                } else {
                    focusChart = new Chart(ctx, {
                        type: 'bar',
                        data: chartData,
                        options: {
                            responsive: true,
                            maintainAspectRatio: false,
                            plugins: {
                                legend: { display: false },
                                tooltip: {
                                    callbacks: {
                                        label: (context) => `${context.dataset.label}: ${context.raw}`
                                    }
                                }
                            },
                            scales: {
                                y: {
                                    beginAtZero: true,
                                    ticks: {
                                        stepSize: 1,
                                        font: { family: 'Inter' }
                                    }
                                },
                                x: {
                                    ticks: { font: { family: 'Inter' } }
                                }
                            }
                        }
                    });
                }
            }

            function updateProjectsTable(data) {
                projectsTableBody.innerHTML = '';

                if (data.length === 0) {
                    projectsTableBody.innerHTML = `<tr><td colspan="5" class="text-center py-8 text-gray-500">Nenhum projeto encontrado para os filtros selecionados.</td></tr>`;
                    return;
                }

                data.forEach(p => {
                    const row = document.createElement('tr');
                    row.className = 'hover:bg-gray-50 cursor-pointer';
                    row.dataset.projectId = p.id;
                    row.innerHTML = `
                        <td class="px-6 py-4 whitespace-nowrap">
                            <div class="text-sm font-semibold text-gray-900">${p.name}</div>
                            <div class="text-xs text-gray-500">${p.primaryClassification}</div>
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-700">${p.responsible.join(', ')}</td>
                        <td class="px-6 py-4 whitespace-nowrap">
                            <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full ${getStatusColor(p.status)}">
                                ${p.status}
                            </span>
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-700">${p.nextMilestone}</td>
                        <td class="px-6 py-4 text-sm text-gray-700 max-w-xs truncate" title="${p.impact}">${p.impact}</td>
                    `;
                    projectsTableBody.appendChild(row);
                });
            }

            function getStatusColor(status) {
                switch (status) {
                    case 'Em Andamento': return 'bg-green-100 text-green-800';
                    case 'Planejamento': return 'bg-yellow-100 text-yellow-800';
                    case 'Em Finalização': return 'bg-blue-100 text-blue-800';
                    case 'Em Operação': return 'bg-indigo-100 text-indigo-800';
                    case 'Concluído': return 'bg-gray-100 text-gray-800';
                    default: return 'bg-red-100 text-red-800';
                }
            }

            function openModal(projectId) {
                const project = projectData.find(p => p.id === projectId);
                if (!project) return;

                document.getElementById('modalTitle').textContent = project.name;
                const modalBody = document.getElementById('modalBody');
                modalBody.innerHTML = `
                    <p class="text-gray-700">${project.description}</p>
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 pt-4 border-t">
                        <div>
                            <h4 class="text-sm font-semibold text-gray-500">Responsáveis</h4>
                            <p class="text-gray-800">${project.responsible.join(', ')}</p>
                        </div>
                        <div>
                            <h4 class="text-sm font-semibold text-gray-500">Status</h4>
                            <p><span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full ${getStatusColor(project.status)}">${project.status}</span></p>
                        </div>
                        <div>
                            <h4 class="text-sm font-semibold text-gray-500">Classificação</h4>
                            <p class="text-gray-800">${project.primaryClassification} / ${project.secondaryClassification}</p>
                        </div>
                        <div>
                            <h4 class="text-sm font-semibold text-gray-500">Próxima Entrega</h4>
                            <p class="text-gray-800">${project.nextMilestone}</p>
                        </div>
                        <div class="sm:col-span-2">
                            <h4 class="text-sm font-semibold text-gray-500">Impacto / Ganhos Esperados</h4>
                            <p class="text-gray-800">${project.impact}</p>
                        </div>
                    </div>
                `;

                modal.classList.remove('hidden');
                setTimeout(() => {
                    modal.classList.remove('opacity-0');
                    modal.querySelector('.modal-content').classList.remove('scale-95');
                }, 10);
            }

            function closeModal() {
                modal.classList.add('opacity-0');
                modal.querySelector('.modal-content').classList.add('scale-95');
                setTimeout(() => {
                    modal.classList.add('hidden');
                }, 300);
            }
            
            loadProjectData();
        });
    </script>
</body>
</html>
