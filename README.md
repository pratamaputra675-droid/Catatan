<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Pengeluaran Rumah Tangga</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons CDN -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Chart.js CDN -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700;800&display=swap" rel="stylesheet">

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Plus Jakarta Sans', 'sans-serif'],
                    },
                    colors: {
                        brand: {
                            50: '#f0fdf4',
                            100: '#dcfce7',
                            500: '#22c55e',
                            600: '#16a34a',
                            700: '#15803d',
                            900: '#14532d',
                        }
                    }
                }
            }
        }
    </script>
    <style>
        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
            -webkit-tap-highlight-color: transparent;
        }
        .no-scrollbar::-webkit-scrollbar {
            display: none;
        }
        .no-scrollbar {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
    </style>
</head>
<body class="bg-slate-100 min-h-screen text-slate-800 flex justify-center items-center sm:py-6">

    <!-- Mobile Frame Container -->
    <div class="w-full max-w-[430px] min-h-screen sm:min-h-[844px] sm:max-h-[900px] bg-slate-50 sm:rounded-[40px] shadow-2xl flex flex-col justify-between overflow-hidden relative border-slate-200/60 sm:border-[8px]">
        
        <!-- Header -->
        <header class="bg-gradient-to-r from-emerald-600 to-teal-700 text-white px-5 pt-6 pb-5 rounded-b-[28px] shadow-lg shrink-0">
            <!-- Top Controls: Month Selector & Action Icons -->
            <div class="flex justify-between items-center mb-4">
                <div class="flex items-center gap-1.5 bg-white/15 backdrop-blur-md px-3 py-1.5 rounded-full text-white border border-white/20">
                    <button onclick="changeMonth(-1)" class="p-0.5 hover:bg-white/20 rounded-full transition" title="Bulan Sebelumnya">
                        <i data-lucide="chevron-left" class="w-4 h-4"></i>
                    </button>
                    <input type="month" id="monthPicker" onchange="onMonthPickerChange(this.value)" class="bg-transparent text-xs font-bold text-white focus:outline-none cursor-pointer [color-scheme:dark]">
                    <button onclick="changeMonth(1)" class="p-0.5 hover:bg-white/20 rounded-full transition" title="Bulan Berikutnya">
                        <i data-lucide="chevron-right" class="w-4 h-4"></i>
                    </button>
                </div>
                
                <div class="flex items-center gap-2">
                    <button onclick="openHistoryModal()" class="bg-white/20 hover:bg-white/30 backdrop-blur-md p-2 rounded-full text-white transition" title="Riwayat & Arsip Bulanan">
                        <i data-lucide="history" class="w-4.5 h-4.5"></i>
                    </button>
                    <button onclick="openExportModal()" class="bg-white/20 hover:bg-white/30 backdrop-blur-md p-2 rounded-full text-white transition" title="Ekspor Data">
                        <i data-lucide="download" class="w-4.5 h-4.5"></i>
                    </button>
                </div>
            </div>

            <!-- Balance Card -->
            <div class="bg-white/10 backdrop-blur-md border border-white/20 rounded-2xl p-4 text-white shadow-inner relative overflow-hidden">
                <div class="flex justify-between items-start">
                    <div>
                        <span class="text-[11px] text-emerald-100 font-semibold tracking-wide uppercase">Sisa Saldo Bulan Ini</span>
                        <h2 class="text-2xl font-extrabold tracking-tight mt-0.5" id="remainingBalanceDisplay">Rp 0</h2>
                    </div>
                    <span id="archiveBadge" class="hidden text-[10px] bg-amber-400 text-slate-900 font-bold px-2.5 py-0.5 rounded-full flex items-center gap-1 shadow-sm">
                        <i data-lucide="lock" class="w-3 h-3"></i> Terarsip
                    </span>
                </div>
            </div>
        </header>

        <!-- Main Content Area -->
        <main class="flex-1 overflow-y-auto px-5 py-4 space-y-4 no-scrollbar">

            <!-- Save / Archive Month Banner -->
            <div id="archiveActionCard" class="bg-gradient-to-r from-emerald-50 to-teal-50 border border-emerald-200/70 p-3.5 rounded-2xl flex items-center justify-between gap-3 shadow-sm">
                <div class="flex items-center gap-2.5">
                    <div class="w-8 h-8 rounded-xl bg-emerald-600 text-white flex items-center justify-center shrink-0">
                        <i data-lucide="archive" class="w-4 h-4"></i>
                    </div>
                    <div>
                        <h4 class="text-xs font-bold text-slate-800" id="archiveBannerTitle">Rekap & Simpan Bulan Ini</h4>
                        <p class="text-[10px] text-slate-500 font-medium">Kunci data pengeluaran bulan ini ke arsip</p>
                    </div>
                </div>
                <button id="btnArchiveToggle" onclick="toggleArchiveCurrentMonth()" class="bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-bold px-3 py-1.5 rounded-xl shadow-sm transition shrink-0">
                    Simpan
                </button>
            </div>

            <!-- Summary Chips (Income vs Expense) -->
            <div class="grid grid-cols-2 gap-3">
                <div class="bg-white p-3 rounded-2xl shadow-sm border border-slate-100 flex items-center gap-3">
                    <div class="w-10 h-10 rounded-xl bg-emerald-100 text-emerald-600 flex items-center justify-center shrink-0">
                        <i data-lucide="arrow-down-left" class="w-5 h-5"></i>
                    </div>
                    <div class="overflow-hidden">
                        <p class="text-[11px] text-slate-400 font-medium">Pemasukan</p>
                        <p class="text-sm font-bold text-slate-800 truncate" id="totalIncomeDisplay">Rp 0</p>
                    </div>
                </div>

                <div class="bg-white p-3 rounded-2xl shadow-sm border border-slate-100 flex items-center gap-3">
                    <div class="w-10 h-10 rounded-xl bg-rose-100 text-rose-600 flex items-center justify-center shrink-0">
                        <i data-lucide="arrow-up-right" class="w-5 h-5"></i>
                    </div>
                    <div class="overflow-hidden">
                        <p class="text-[11px] text-slate-400 font-medium">Pengeluaran</p>
                        <p class="text-sm font-bold text-slate-800 truncate" id="subTotalExpenseDisplay">Rp 0</p>
                    </div>
                </div>
            </div>

            <!-- Categories Breakdown Chart -->
            <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-100">
                <div class="flex justify-between items-center mb-2">
                    <h3 class="text-sm font-bold text-slate-700">Analisis Pengeluaran</h3>
                    <span class="text-[10px] bg-slate-100 text-slate-500 font-semibold px-2 py-0.5 rounded-md" id="chartMonthLabel">Bulan Ini</span>
                </div>
                <div class="relative w-full h-40 flex items-center justify-center">
                    <canvas id="categoryChart"></canvas>
                    <div id="noChartData" class="hidden absolute inset-0 flex flex-col items-center justify-center text-slate-400 bg-white/90">
                        <i data-lucide="pie-chart" class="w-8 h-8 mb-1 stroke-1"></i>
                        <p class="text-xs">Belum ada data pengeluaran</p>
                    </div>
                </div>
            </div>

            <!-- Transactions Section -->
            <div class="space-y-3">
                <div class="flex justify-between items-center">
                    <h3 class="text-sm font-bold text-slate-700">Riwayat Transaksi</h3>
                    <select id="filterCategory" onchange="renderTransactions()" class="text-xs bg-white border border-slate-200 rounded-lg px-2.5 py-1 text-slate-600 focus:outline-none focus:ring-2 focus:ring-emerald-500">
                        <option value="ALL">Semua Kategori</option>
                        <option value="Makanan/Belanja">Makanan & Belanja</option>
                        <option value="Listrik/Air/Internet">Tagihan (Listrik/Air)</option>
                        <option value="Transportasi">Transportasi</option>
                        <option value="Kesehatan">Kesehatan</option>
                        <option value="Cicilan/Pendidikan">Cicilan & Edukasi</option>
                        <option value="Lain-lain">Lain-lain</option>
                        <option value="Pemasukan">Pemasukan</option>
                    </select>
                </div>

                <!-- Transaction List Container -->
                <div id="transactionList" class="space-y-2.5 pb-20">
                    <!-- Dynamic rendering via JS -->
                </div>
            </div>
        </main>

        <!-- Floating Action Button / Add Button -->
        <div class="absolute bottom-5 inset-x-0 flex justify-center px-5 pointer-events-none">
            <button id="btnAddTransaction" onclick="openTransactionModal()" class="pointer-events-auto bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-700 hover:to-teal-700 text-white font-semibold py-3.5 px-6 rounded-full shadow-xl flex items-center gap-2 active:scale-95 transition transform">
                <i data-lucide="plus" class="w-5 h-5"></i>
                <span>Tambah Transaksi</span>
            </button>
        </div>

    </div>

    <!-- Modal: Add / Edit Transaction -->
    <div id="transactionModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex justify-center items-end sm:items-center hidden transition-opacity duration-300">
        <div class="bg-white w-full max-w-[430px] rounded-t-3xl sm:rounded-3xl p-6 shadow-2xl space-y-4 max-h-[90vh] overflow-y-auto">
            <div class="flex justify-between items-center border-b pb-3">
                <h3 class="text-lg font-bold text-slate-800" id="modalTitle">Tambah Transaksi</h3>
                <button onclick="closeTransactionModal()" class="text-slate-400 hover:text-slate-600 p-1">
                    <i data-lucide="x" class="w-6 h-6"></i>
                </button>
            </div>

            <form id="transactionForm" onsubmit="saveTransaction(event)" class="space-y-4">
                <input type="hidden" id="editTxId">

                <!-- Type Selector (Pengeluaran vs Pemasukan) -->
                <div class="grid grid-cols-2 gap-2 p-1 bg-slate-100 rounded-xl">
                    <button type="button" id="btnTypeExpense" onclick="setType('EXPENSE')" class="py-2 text-xs font-bold rounded-lg transition bg-white text-rose-600 shadow-sm">Pengeluaran</button>
                    <button type="button" id="btnTypeIncome" onclick="setType('INCOME')" class="py-2 text-xs font-bold rounded-lg transition text-slate-500">Pemasukan</button>
                </div>
                <input type="hidden" id="txType" value="EXPENSE">

                <!-- Amount Input -->
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">Jumlah (Rp)</label>
                    <div class="relative">
                        <span class="absolute left-3 top-3 text-slate-400 font-bold text-sm">Rp</span>
                        <input type="number" id="txAmount" required placeholder="0" class="w-full pl-10 pr-4 py-2.5 bg-slate-50 border border-slate-200 rounded-xl font-bold text-slate-800 text-lg focus:outline-none focus:ring-2 focus:ring-emerald-500">
                    </div>
                </div>

                <!-- Category -->
                <div id="categoryContainer">
                    <label class="block text-xs font-semibold text-slate-600 mb-1">Kategori</label>
                    <select id="txCategory" class="w-full p-2.5 bg-slate-50 border border-slate-200 rounded-xl text-slate-700 font-medium text-xs focus:outline-none focus:ring-2 focus:ring-emerald-500">
                        <option value="Makanan/Belanja">🍔 Makanan & Belanja</option>
                        <option value="Listrik/Air/Internet">⚡ Listrik, Air & Internet</option>
                        <option value="Transportasi">🚗 Transportasi</option>
                        <option value="Kesehatan">🏥 Kesehatan</option>
                        <option value="Cicilan/Pendidikan">🎓 Cicilan & Pendidikan</option>
                        <option value="Lain-lain">📦 Lain-lain</option>
                    </select>
                </div>

                <!-- Date & Method Grid -->
                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">Tanggal</label>
                        <input type="date" id="txDate" required class="w-full p-2.5 bg-slate-50 border border-slate-200 rounded-xl text-slate-700 text-xs font-medium focus:outline-none focus:ring-2 focus:ring-emerald-500">
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">Pembayaran</label>
                        <select id="txMethod" class="w-full p-2.5 bg-slate-50 border border-slate-200 rounded-xl text-slate-700 text-xs font-medium focus:outline-none focus:ring-2 focus:ring-emerald-500">
                            <option value="Tunai">💵 Tunai</option>
                            <option value="Transfer/Bank">💳 Transfer / Bank</option>
                            <option value="E-Wallet">📱 E-Wallet</option>
                        </select>
                    </div>
                </div>

                <!-- Note -->
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">Catatan (Opsional)</label>
                    <input type="text" id="txNote" placeholder="Contoh: Beli beras 10kg" class="w-full p-2.5 bg-slate-50 border border-slate-200 rounded-xl text-slate-700 text-xs focus:outline-none focus:ring-2 focus:ring-emerald-500">
                </div>

                <!-- Submit Button -->
                <button type="submit" class="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-bold py-3 rounded-xl transition shadow-lg shadow-emerald-600/30 text-xs">
                    Simpan Transaksi
                </button>
            </form>
        </div>
    </div>

    <!-- Modal: Monthly History & Archives -->
    <div id="historyModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex justify-center items-end sm:items-center hidden transition-opacity duration-300">
        <div class="bg-white w-full max-w-[430px] rounded-t-3xl sm:rounded-3xl p-6 shadow-2xl space-y-4 max-h-[85vh] flex flex-col">
            <div class="flex justify-between items-center border-b pb-3 shrink-0">
                <div class="flex items-center gap-2">
                    <i data-lucide="archive" class="w-5 h-5 text-emerald-600"></i>
                    <h3 class="text-base font-bold text-slate-800">Arsip & Rekap Bulanan</h3>
                </div>
                <button onclick="closeHistoryModal()" class="text-slate-400 hover:text-slate-600 p-1">
                    <i data-lucide="x" class="w-6 h-6"></i>
                </button>
            </div>
            
            <p class="text-xs text-slate-500 shrink-0">Daftar rekapitulasi pengeluaran bulanan yang tersimpan:</p>

            <div id="historyList" class="flex-1 overflow-y-auto space-y-2.5 pr-1 no-scrollbar">
                <!-- Dynamic render -->
            </div>
        </div>
    </div>

    <!-- Modal: Export Data -->
    <div id="exportModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex justify-center items-end sm:items-center hidden transition-opacity duration-300">
        <div class="bg-white w-full max-w-[430px] rounded-t-3xl sm:rounded-3xl p-6 shadow-2xl space-y-4">
            <div class="flex justify-between items-center border-b pb-3">
                <div class="flex items-center gap-2">
                    <i data-lucide="download" class="w-5 h-5 text-emerald-600"></i>
                    <h3 class="text-base font-bold text-slate-800">Ekspor Data</h3>
                </div>
                <button onclick="closeExportModal()" class="text-slate-400 hover:text-slate-600 p-1">
                    <i data-lucide="x" class="w-6 h-6"></i>
                </button>
            </div>

            <p class="text-xs text-slate-500">Pilih format untuk mengunduh rekap transaksi bulan yang dipilih (<span id="exportMonthName" class="font-bold text-slate-700"></span>):</p>

            <div class="space-y-2.5 pt-2">
                <button onclick="exportData('csv')" class="w-full bg-slate-50 border border-slate-200 hover:bg-emerald-50 hover:border-emerald-300 p-3 rounded-2xl flex items-center justify-between transition group">
                    <div class="flex items-center gap-3">
                        <div class="w-9 h-9 rounded-xl bg-emerald-100 text-emerald-600 flex items-center justify-center font-bold text-xs">CSV</div>
                        <div class="text-left">
                            <h4 class="text-xs font-bold text-slate-800 group-hover:text-emerald-700">Format Excel / CSV</h4>
                            <p class="text-[10px] text-slate-400">Cocok dibuka di Microsoft Excel atau Spreadsheet</p>
                        </div>
                    </div>
                    <i data-lucide="chevron-right" class="w-4 h-4 text-slate-400 group-hover:text-emerald-600"></i>
                </button>

                <button onclick="exportData('json')" class="w-full bg-slate-50 border border-slate-200 hover:bg-emerald-50 hover:border-emerald-300 p-3 rounded-2xl flex items-center justify-between transition group">
                    <div class="flex items-center gap-3">
                        <div class="w-9 h-9 rounded-xl bg-blue-100 text-blue-600 flex items-center justify-center font-bold text-xs">JSON</div>
                        <div class="text-left">
                            <h4 class="text-xs font-bold text-slate-800 group-hover:text-emerald-700">Format Backup JSON</h4>
                            <p class="text-[10px] text-slate-400">Seluruh data mentah transaksi & arsip</p>
                        </div>
                    </div>
                    <i data-lucide="chevron-right" class="w-4 h-4 text-slate-400 group-hover:text-emerald-600"></i>
                </button>
            </div>
        </div>
    </div>

    <!-- Notification Toast -->
    <div id="toast" class="fixed top-5 left-1/2 -translate-x-1/2 bg-slate-800 text-white text-xs px-4 py-2.5 rounded-full shadow-lg z-50 opacity-0 transition-opacity pointer-events-none duration-300 flex items-center gap-2">
        <i data-lucide="check-circle" class="w-4 h-4 text-emerald-400"></i>
        <span id="toastMsg">Pesan Notifikasi</span>
    </div>

    <script>
        // Global Application State
        let appData = {
            transactions: [],
            archivedMonths: {} // Key: 'YYYY-MM', Value: true/false
        };

        let currentSelectedMonth = getCurrentYearMonthStr();
        let chartInstance = null;

        // Category Configurations
        const CATEGORY_CONFIG = {
            'Makanan/Belanja': { color: '#f59e0b', bg: 'bg-amber-50', text: 'text-amber-600', icon: 'utensils' },
            'Listrik/Air/Internet': { color: '#06b6d4', bg: 'bg-cyan-50', text: 'text-cyan-600', icon: 'zap' },
            'Transportasi': { color: '#3b82f6', bg: 'bg-blue-50', text: 'text-blue-600', icon: 'car' },
            'Kesehatan': { color: '#ec4899', bg: 'bg-pink-50', text: 'text-pink-600', icon: 'activity' },
            'Cicilan/Pendidikan': { color: '#8b5cf6', bg: 'bg-purple-50', text: 'text-purple-600', icon: 'graduation-cap' },
            'Lain-lain': { color: '#64748b', bg: 'bg-slate-100', text: 'text-slate-600', icon: 'more-horizontal' },
            'Pemasukan': { color: '#10b981', bg: 'bg-emerald-50', text: 'text-emerald-600', icon: 'arrow-down-left' }
        };

        // Window Load
        window.onload = function() {
            loadFromLocalStorage();
            document.getElementById('monthPicker').value = currentSelectedMonth;
            renderAll();
            lucide.createIcons();
        };

        function getCurrentYearMonthStr() {
            const now = new Date();
            const yyyy = now.getFullYear();
            const mm = String(now.getMonth() + 1).padStart(2, '0');
            return `${yyyy}-${mm}`;
        }

        function formatIDR(amount) {
            return new Intl.NumberFormat('id-ID', {
                style: 'currency',
                currency: 'IDR',
                maximumFractionDigits: 0
            }).format(amount);
        }

        function formatMonthLabel(yearMonthStr) {
            const [y, m] = yearMonthStr.split('-');
            const date = new Date(parseInt(y), parseInt(m) - 1, 1);
            return date.toLocaleDateString('id-ID', { month: 'long', year: 'numeric' });
        }

        // Local Storage
        function loadFromLocalStorage() {
            const saved = localStorage.getItem('dompet_rumah_tangga');
            if (saved) {
                try {
                    const parsed = JSON.parse(saved);
                    appData.transactions = parsed.transactions || [];
                    appData.archivedMonths = parsed.archivedMonths || {};
                } catch(e) {
                    console.error("Gagal membaca storage", e);
                }
            } else {
                // Seed Initial Data for current month
                const ym = getCurrentYearMonthStr();
                appData.transactions = [
                    { id: '1', type: 'INCOME', amount: 8500000, category: 'Pemasukan', method: 'Transfer/Bank', date: `${ym}-01`, note: 'Gaji Bulanan' },
                    { id: '2', type: 'EXPENSE', amount: 650000, category: 'Makanan/Belanja', method: 'Tunai', date: `${ym}-05`, note: 'Belanja Dapur & Sayur' },
                    { id: '3', type: 'EXPENSE', amount: 450000, category: 'Listrik/Air/Internet', method: 'E-Wallet', date: `${ym}-08`, note: 'Listrik & Wifi Rumah' },
                    { id: '4', type: 'EXPENSE', amount: 200000, category: 'Transportasi', method: 'E-Wallet', date: `${ym}-10`, note: 'Bensin & Tol' },
                    { id: '5', type: 'EXPENSE', amount: 150000, category: 'Kesehatan', method: 'Tunai', date: `${ym}-12`, note: 'Vitamin & Obat' }
                ];
                saveToLocalStorage();
            }
        }

        function saveToLocalStorage() {
            localStorage.setItem('dompet_rumah_tangga', JSON.stringify(appData));
        }

        // Month Navigation
        function changeMonth(delta) {
            const [y, m] = currentSelectedMonth.split('-').map(Number);
            const date = new Date(y, m - 1 + delta, 1);
            const newY = date.getFullYear();
            const newM = String(date.getMonth() + 1).padStart(2, '0');
            currentSelectedMonth = `${newY}-${newM}`;
            document.getElementById('monthPicker').value = currentSelectedMonth;
            renderAll();
        }

        function onMonthPickerChange(val) {
            if (!val) return;
            currentSelectedMonth = val;
            renderAll();
        }

        // Master Render Function
        function renderAll() {
            const isArchived = !!appData.archivedMonths[currentSelectedMonth];

            // Render Header Status & Badges
            const archiveBadge = document.getElementById('archiveBadge');
            const btnAddTx = document.getElementById('btnAddTransaction');
            const archiveBannerTitle = document.getElementById('archiveBannerTitle');
            const btnArchiveToggle = document.getElementById('btnArchiveToggle');

            if (isArchived) {
                archiveBadge.classList.remove('hidden');
                btnAddTx.classList.add('opacity-40', 'pointer-events-none');
                archiveBannerTitle.innerText = "Bulan Ini Terkunci (Terarsip)";
                btnArchiveToggle.innerText = "Buka Arsip";
                btnArchiveToggle.className = "bg-amber-600 hover:bg-amber-700 text-white text-xs font-bold px-3 py-1.5 rounded-xl shadow-sm transition shrink-0";
            } else {
                archiveBadge.classList.add('hidden');
                btnAddTx.classList.remove('opacity-40', 'pointer-events-none');
                archiveBannerTitle.innerText = "Simpan & Kunci Bulan Ini";
                btnArchiveToggle.innerText = "Simpan";
                btnArchiveToggle.className = "bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-bold px-3 py-1.5 rounded-xl shadow-sm transition shrink-0";
            }

            document.getElementById('chartMonthLabel').innerText = formatMonthLabel(currentSelectedMonth);

            renderDashboard();
            renderTransactions();
        }

        // Toggle Archive for current month
        function toggleArchiveCurrentMonth() {
            const isArchived = !!appData.archivedMonths[currentSelectedMonth];
            if (isArchived) {
                delete appData.archivedMonths[currentSelectedMonth];
                showToast(`Arsip bulan ${formatMonthLabel(currentSelectedMonth)} dibuka`);
            } else {
                appData.archivedMonths[currentSelectedMonth] = true;
                showToast(`Bulan ${formatMonthLabel(currentSelectedMonth)} berhasil diarsip`);
            }
            saveToLocalStorage();
            renderAll();
        }

        // Dashboard & Analytics
        function renderDashboard() {
            const monthlyTxs = appData.transactions.filter(t => t.date.startsWith(currentSelectedMonth));

            let totalIncome = 0;
            let totalExpense = 0;
            const categoryTotals = {
                'Makanan/Belanja': 0,
                'Listrik/Air/Internet': 0,
                'Transportasi': 0,
                'Kesehatan': 0,
                'Cicilan/Pendidikan': 0,
                'Lain-lain': 0
            };

            monthlyTxs.forEach(t => {
                const amt = parseFloat(t.amount) || 0;
                if (t.type === 'INCOME') {
                    totalIncome += amt;
                } else {
                    totalExpense += amt;
                    if (categoryTotals.hasOwnProperty(t.category)) {
                        categoryTotals[t.category] += amt;
                    } else {
                        categoryTotals['Lain-lain'] += amt;
                    }
                }
            });

            const remainingBalance = totalIncome - totalExpense;

            document.getElementById('remainingBalanceDisplay').innerText = formatIDR(remainingBalance);
            document.getElementById('totalIncomeDisplay').innerText = formatIDR(totalIncome);
            document.getElementById('subTotalExpenseDisplay').innerText = formatIDR(totalExpense);

            renderChart(categoryTotals, totalExpense);
        }

        function renderChart(categoryTotals, totalExpense) {
            const ctx = document.getElementById('categoryChart').getContext('2d');
            const noDataEl = document.getElementById('noChartData');

            if (totalExpense === 0) {
                noDataEl.classList.remove('hidden');
                if (chartInstance) chartInstance.destroy();
                return;
            } else {
                noDataEl.classList.add('hidden');
            }

            // Filter out 0 categories
            const labels = [];
            const data = [];
            const backgroundColors = [];

            Object.keys(categoryTotals).forEach(cat => {
                if (categoryTotals[cat] > 0) {
                    labels.push(cat);
                    data.push(categoryTotals[cat]);
                    backgroundColors.push(CATEGORY_CONFIG[cat]?.color || '#cbd5e1');
                }
            });

            if (chartInstance) {
                chartInstance.destroy();
            }

            chartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [{
                        data: data,
                        backgroundColor: backgroundColors,
                        borderWidth: 2,
                        borderColor: '#ffffff',
                        hoverOffset: 4
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            position: 'right',
                            labels: {
                                boxWidth: 10,
                                font: { size: 10, family: 'Plus Jakarta Sans' },
                                padding: 6
                            }
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    return ` ${context.label}: ${formatIDR(context.raw)}`;
                                }
                            }
                        }
                    },
                    cutout: '68%'
                }
            });
        }

        // Transaction List Rendering
        function renderTransactions() {
            const container = document.getElementById('transactionList');
            const filter = document.getElementById('filterCategory').value;
            const isArchived = !!appData.archivedMonths[currentSelectedMonth];

            let monthlyTxs = appData.transactions.filter(t => t.date.startsWith(currentSelectedMonth));

            if (filter !== 'ALL') {
                if (filter === 'Pemasukan') {
                    monthlyTxs = monthlyTxs.filter(t => t.type === 'INCOME');
                } else {
                    monthlyTxs = monthlyTxs.filter(t => t.type === 'EXPENSE' && t.category === filter);
                }
            }

            monthlyTxs.sort((a, b) => new Date(b.date) - new Date(a.date));

            if (monthlyTxs.length === 0) {
                container.innerHTML = `
                    <div class="text-center py-8 text-slate-400 bg-white rounded-2xl border border-slate-100">
                        <i data-lucide="receipt" class="w-10 h-10 mx-auto mb-2 stroke-1 opacity-60"></i>
                        <p class="text-xs font-medium">Belum ada transaksi di bulan ini</p>
                    </div>
                `;
                lucide.createIcons();
                return;
            }

            container.innerHTML = monthlyTxs.map(item => {
                const isIncome = item.type === 'INCOME';
                const config = isIncome ? CATEGORY_CONFIG['Pemasukan'] : (CATEGORY_CONFIG[item.category] || CATEGORY_CONFIG['Lain-lain']);
                const formattedAmt = (isIncome ? '+ ' : '- ') + formatIDR(item.amount);
                const amtColor = isIncome ? 'text-emerald-600' : 'text-slate-800';

                return `
                    <div class="bg-white p-3.5 rounded-2xl border border-slate-100 shadow-sm flex items-center justify-between gap-3 hover:border-slate-200 transition">
                        <div class="flex items-center gap-3 overflow-hidden">
                            <div class="w-10 h-10 rounded-xl ${config.bg} ${config.text} flex items-center justify-center shrink-0">
                                <i data-lucide="${config.icon}" class="w-5 h-5"></i>
                            </div>
                            <div class="overflow-hidden">
                                <h4 class="text-xs font-bold text-slate-800 truncate">${item.note || item.category}</h4>
                                <div class="flex items-center gap-2 mt-0.5 text-[10px] text-slate-400 font-medium">
                                    <span>${formatDateDisplay(item.date)}</span>
                                    <span>•</span>
                                    <span class="bg-slate-100 px-1.5 py-0.5 rounded text-slate-500">${item.method || 'Tunai'}</span>
                                </div>
                            </div>
                        </div>

                        <div class="text-right shrink-0">
                            <p class="text-xs font-extrabold ${amtColor}">${formattedAmt}</p>
                            ${!isArchived ? `
                            <div class="flex items-center justify-end gap-1 mt-1">
                                <button onclick="editTransaction('${item.id}')" class="text-slate-400 hover:text-blue-600 p-1">
                                    <i data-lucide="pencil" class="w-3.5 h-3.5"></i>
                                </button>
                                <button onclick="deleteTransaction('${item.id}')" class="text-slate-400 hover:text-rose-600 p-1">
                                    <i data-lucide="trash-2" class="w-3.5 h-3.5"></i>
                                </button>
                            </div>
                            ` : ''}
                        </div>
                    </div>
                `;
            }).join('');

            lucide.createIcons();
        }

        function formatDateDisplay(dateStr) {
            if (!dateStr) return '';
            const d = new Date(dateStr);
            return d.toLocaleDateString('id-ID', { day: 'numeric', month: 'short' });
        }

        // Form Type Selector
        function setType(type) {
            document.getElementById('txType').value = type;
            const btnExpense = document.getElementById('btnTypeExpense');
            const btnIncome = document.getElementById('btnTypeIncome');
            const categoryContainer = document.getElementById('categoryContainer');

            if (type === 'EXPENSE') {
                btnExpense.className = "py-2 text-xs font-bold rounded-lg transition bg-white text-rose-600 shadow-sm";
                btnIncome.className = "py-2 text-xs font-bold rounded-lg transition text-slate-500";
                categoryContainer.classList.remove('hidden');
            } else {
                btnIncome.className = "py-2 text-xs font-bold rounded-lg transition bg-white text-emerald-600 shadow-sm";
                btnExpense.className = "py-2 text-xs font-bold rounded-lg transition text-slate-500";
                categoryContainer.classList.add('hidden');
            }
        }

        // Transaction CRUD
        function saveTransaction(e) {
            e.preventDefault();
            if (appData.archivedMonths[currentSelectedMonth]) {
                showToast("Bulan ini sedang terarsip & terkunci!");
                return;
            }

            const id = document.getElementById('editTxId').value;
            const type = document.getElementById('txType').value;
            const amount = parseFloat(document.getElementById('txAmount').value);
            const category = type === 'INCOME' ? 'Pemasukan' : document.getElementById('txCategory').value;
            const date = document.getElementById('txDate').value;
            const method = document.getElementById('txMethod').value;
            const note = document.getElementById('txNote').value.trim();

            if (!amount || amount <= 0) {
                showToast("Masukkan jumlah nominal yang valid!");
                return;
            }

            if (id) {
                const idx = appData.transactions.findIndex(t => t.id === id);
                if (idx !== -1) {
                    appData.transactions[idx] = { id, type, amount, category, date, method, note };
                    showToast("Transaksi diperbarui");
                }
            } else {
                const newTx = {
                    id: Date.now().toString(),
                    type,
                    amount,
                    category,
                    date,
                    method,
                    note
                };
                appData.transactions.push(newTx);
                showToast("Transaksi ditambahkan");
            }

            saveToLocalStorage();
            renderAll();
            closeTransactionModal();
        }

        function editTransaction(id) {
            const tx = appData.transactions.find(t => t.id === id);
            if (!tx) return;

            document.getElementById('editTxId').value = tx.id;
            setType(tx.type);
            document.getElementById('txAmount').value = tx.amount;
            if (tx.type === 'EXPENSE') {
                document.getElementById('txCategory').value = tx.category;
            }
            document.getElementById('txDate').value = tx.date;
            document.getElementById('txMethod').value = tx.method || 'Tunai';
            document.getElementById('txNote').value = tx.note || '';

            document.getElementById('modalTitle').innerText = "Edit Transaksi";
            openTransactionModal();
        }

        function deleteTransaction(id) {
            if (appData.archivedMonths[currentSelectedMonth]) {
                showToast("Bulan ini terkunci!");
                return;
            }
            appData.transactions = appData.transactions.filter(t => t.id !== id);
            saveToLocalStorage();
            renderAll();
            showToast("Transaksi dihapus");
        }

        // Modals Management
        function openTransactionModal() {
            if (appData.archivedMonths[currentSelectedMonth]) {
                showToast("Bulan ini terarsip! Buka arsip untuk menambah data.");
                return;
            }
            document.getElementById('transactionModal').classList.remove('hidden');
            // Set default date to match selected month
            const today = new Date().toISOString().split('T')[0];
            if (today.startsWith(currentSelectedMonth)) {
                document.getElementById('txDate').value = today;
            } else {
                document.getElementById('txDate').value = `${currentSelectedMonth}-01`;
            }
        }

        function closeTransactionModal() {
            document.getElementById('transactionModal').classList.add('hidden');
            document.getElementById('transactionForm').reset();
            document.getElementById('editTxId').value = '';
            document.getElementById('modalTitle').innerText = "Tambah Transaksi";
            setType('EXPENSE');
        }

        // History / Archives Modal
        function openHistoryModal() {
            const listEl = document.getElementById('historyList');
            const monthMap = {};

            // Group transactions by YYYY-MM
            appData.transactions.forEach(t => {
                const ym = t.date.substring(0, 7);
                if (!monthMap[ym]) {
                    monthMap[ym] = { income: 0, expense: 0, count: 0 };
                }
                const amt = parseFloat(t.amount) || 0;
                if (t.type === 'INCOME') monthMap[ym].income += amt;
                else monthMap[ym].expense += amt;
                monthMap[ym].count++;
            });

            // Ensure current selected month is present in map
            if (!monthMap[currentSelectedMonth]) {
                monthMap[currentSelectedMonth] = { income: 0, expense: 0, count: 0 };
            }

            const sortedMonths = Object.keys(monthMap).sort().reverse();

            if (sortedMonths.length === 0) {
                listEl.innerHTML = `<p class="text-xs text-slate-400 py-4 text-center">Belum ada riwayat arsip</p>`;
            } else {
                listEl.innerHTML = sortedMonths.map(ym => {
                    const isArchived = !!appData.archivedMonths[ym];
                    const data = monthMap[ym];
                    const balance = data.income - data.expense;
                    const isSelected = ym === currentSelectedMonth;

                    return `
                        <div onclick="selectMonthFromHistory('${ym}')" class="p-3.5 rounded-2xl border ${isSelected ? 'border-emerald-500 bg-emerald-50/40' : 'border-slate-100 bg-slate-50'} hover:border-emerald-300 transition cursor-pointer flex items-center justify-between">
                            <div>
                                <div class="flex items-center gap-2">
                                    <h4 class="text-xs font-bold text-slate-800">${formatMonthLabel(ym)}</h4>
                                    ${isArchived ? `<span class="text-[9px] bg-amber-100 text-amber-700 font-bold px-1.5 py-0.5 rounded flex items-center gap-0.5"><i data-lucide="lock" class="w-2.5 h-2.5"></i> Arsip</span>` : ''}
                                </div>
                                <p class="text-[10px] text-slate-400 mt-0.5">${data.count} Transaksi</p>
                            </div>
                            <div class="text-right">
                                <p class="text-xs font-extrabold ${balance >= 0 ? 'text-emerald-600' : 'text-rose-600'}">${formatIDR(balance)}</p>
                                <span class="text-[10px] text-slate-400">Pengeluaran: ${formatIDR(data.expense)}</span>
                            </div>
                        </div>
                    `;
                }).join('');
            }

            lucide.createIcons();
            document.getElementById('historyModal').classList.remove('hidden');
        }

        function selectMonthFromHistory(ym) {
            currentSelectedMonth = ym;
            document.getElementById('monthPicker').value = currentSelectedMonth;
            renderAll();
            closeHistoryModal();
            showToast(`Bulan diubah ke ${formatMonthLabel(ym)}`);
        }

        function closeHistoryModal() {
            document.getElementById('historyModal').classList.add('hidden');
        }

        // Export Data Modal & Functions
        function openExportModal() {
            document.getElementById('exportMonthName').innerText = formatMonthLabel(currentSelectedMonth);
            document.getElementById('exportModal').classList.remove('hidden');
        }

        function closeExportModal() {
            document.getElementById('exportModal').classList.add('hidden');
        }

        function exportData(format) {
            const monthlyTxs = appData.transactions.filter(t => t.date.startsWith(currentSelectedMonth));

            if (format === 'csv') {
                if (monthlyTxs.length === 0) {
                    showToast("Tidak ada data transaksi bulan ini untuk diekspor");
                    return;
                }

                let csv = 'ID,Tanggal,Tipe,Kategori,Metode Pembayaran,Jumlah (Rp),Catatan\n';
                monthlyTxs.forEach(t => {
                    const cleanNote = (t.note || '').replace(/"/g, '""');
                    csv += `"${t.id}","${t.date}","${t.type}","${t.category}","${t.method || 'Tunai'}","${t.amount}","${cleanNote}"\n`;
                });

                downloadFile(csv, `Pengeluaran_${currentSelectedMonth}.csv`, 'text/csv');
                showToast("File CSV berhasil diunduh");
            } else if (format === 'json') {
                const jsonStr = JSON.stringify(appData, null, 2);
                downloadFile(jsonStr, `Backup_DompetRumahTangga_${currentSelectedMonth}.json`, 'application/json');
                showToast("File Backup JSON berhasil diunduh");
            }

            closeExportModal();
        }

        function downloadFile(content, fileName, contentType) {
            const blob = new Blob([content], { type: contentType });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = fileName;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        }

        // Toast Helper
        function showToast(msg) {
            const toast = document.getElementById('toast');
            document.getElementById('toastMsg').innerText = msg;
            toast.classList.remove('opacity-0');
            setTimeout(() => {
                toast.classList.add('opacity-0');
            }, 2500);
        }
    </script>
</body>
</html>
