<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Catatan Dompet Rumah Tangga</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Google Font Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">
    <style>
        :root {
            --side-padding: 1rem;
            --font-scale: 100%;
        }
        body {
            font-family: 'Inter', sans-serif;
            font-size: var(--font-scale);
            background-color: #0f172a;
        }
        /* Custom scrollbar */
        ::-webkit-scrollbar {
            width: 4px;
        }
        ::-webkit-scrollbar-track {
            background: transparent;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
        .dynamic-padding {
            padding-left: var(--side-padding);
            padding-right: var(--side-padding);
        }
    </style>
</head>
<body class="min-h-screen flex items-center justify-center sm:py-6 text-slate-800">

    <!-- Container Frame Mobile/Desktop -->
    <div id="appContainer" class="w-full max-w-md bg-slate-50 h-screen sm:h-[844px] sm:rounded-[40px] shadow-2xl flex flex-col overflow-hidden relative border-4 border-slate-900/10">

        <!-- HEADER FIXED (Navigasi Atas & Info Saldo) -->
        <header id="headerTheme" class="bg-gradient-to-r from-emerald-600 to-teal-700 text-white shrink-0 pt-5 pb-5 transition-colors duration-300 shadow-md">
            <div class="dynamic-padding space-y-4">
                <!-- Row 1: Title, Month Selector & Settings -->
                <div class="flex items-center justify-between">
                    <div>
                        <h1 id="walletTitleDisplay" class="text-xs font-semibold text-emerald-100/90 tracking-wide">Dompet Rumah Tangga</h1>
                        <div class="flex items-center space-x-1 mt-0.5">
                            <input type="month" id="monthPicker" onchange="changeMonth(this.value)" class="bg-white/10 text-white font-bold text-xs rounded-lg px-2 py-1 outline-none border border-white/20 cursor-pointer hover:bg-white/20 transition">
                            <span id="archiveBadge" class="hidden text-[10px] bg-amber-400 text-amber-950 px-2 py-0.5 rounded-full font-bold flex items-center gap-1">
                                <i data-lucide="lock" class="w-2.5 h-2.5"></i> Terarsip
                            </span>
                        </div>
                    </div>
                    <div class="flex items-center space-x-1">
                        <button onclick="openArchiveModal()" title="Riwayat Arsip" class="p-2 bg-white/10 hover:bg-white/20 rounded-full transition text-white">
                            <i data-lucide="archive" class="w-4 h-4"></i>
                        </button>
                        <button onclick="toggleArchiveStatus()" id="btnArchiveMonth" title="Kunci/Arsip Bulan Ini" class="p-2 bg-white/10 hover:bg-white/20 rounded-full transition text-white">
                            <i data-lucide="lock-keyhole" class="w-4 h-4"></i>
                        </button>
                        <button onclick="openSettingsModal()" title="Pengaturan" class="p-2 bg-white/10 hover:bg-white/20 rounded-full transition text-white">
                            <i data-lucide="settings" class="w-4 h-4"></i>
                        </button>
                    </div>
                </div>

                <!-- Row 2: Sisa Saldo utama -->
                <div class="bg-white/10 backdrop-blur-md rounded-2xl p-4 border border-white/20 text-center relative overflow-hidden">
                    <p class="text-[11px] font-medium text-emerald-100 uppercase tracking-wider">Sisa Saldo Bulan Ini</p>
                    <h2 id="totalBalance" class="text-3xl font-extrabold tracking-tight my-1">Rp 0</h2>
                    <p id="balanceSubtitle" class="text-[10px] text-emerald-200">Total Pemasukan − Total Pengeluaran</p>
                </div>

                <!-- Row 3: Ringkasan Pemasukan & Pengeluaran (Clickable Filter) -->
                <div class="grid grid-cols-2 gap-3 pt-1">
                    <!-- Card Pemasukan -->
                    <div onclick="filterByCardType('income')" class="bg-white/15 backdrop-blur-sm p-2.5 rounded-xl border border-white/10 cursor-pointer hover:bg-white/25 active:scale-95 transition flex items-center space-x-2.5 group">
                        <div class="w-7 h-7 rounded-lg bg-emerald-400/20 flex items-center justify-center shrink-0">
                            <i data-lucide="arrow-down-left" class="w-3.5 h-3.5 text-emerald-200"></i>
                        </div>
                        <div class="min-w-0">
                            <p class="text-[10px] font-medium text-emerald-100 truncate">Pemasukan</p>
                            <p id="totalIncome" class="text-xs font-bold text-white truncate">Rp 0</p>
                        </div>
                    </div>

                    <!-- Card Pengeluaran -->
                    <div onclick="filterByCardType('expense')" class="bg-white/15 backdrop-blur-sm p-2.5 rounded-xl border border-white/10 cursor-pointer hover:bg-white/25 active:scale-95 transition flex items-center space-x-2.5 group">
                        <div class="w-7 h-7 rounded-lg bg-rose-400/20 flex items-center justify-center shrink-0">
                            <i data-lucide="arrow-up-right" class="w-3.5 h-3.5 text-rose-200"></i>
                        </div>
                        <div class="min-w-0">
                            <p class="text-[10px] font-medium text-emerald-100 truncate">Pengeluaran</p>
                            <p id="totalExpense" class="text-xs font-bold text-white truncate">Rp 0</p>
                        </div>
                    </div>
                </div>
            </div>
        </header>

        <!-- AREA UTAMA DENGAN SCROLLING -->
        <main class="flex-1 overflow-y-auto dynamic-padding py-4 space-y-5">
            
            <!-- Analisis Pengeluaran (Grafik & Persentase) -->
            <section class="bg-white p-4 rounded-2xl shadow-sm border border-slate-100 space-y-3">
                <div class="flex items-center justify-between">
                    <h3 class="text-xs font-bold text-slate-700 flex items-center gap-1.5">
                        <i data-lucide="pie-chart" class="w-4 h-4 text-emerald-600"></i> Analisis Pengeluaran
                    </h3>
                    <span id="chartMonthLabel" class="text-[10px] text-slate-400 font-medium">Bulan Ini</span>
                </div>
                
                <div class="relative h-44 flex items-center justify-center">
                    <canvas id="expenseChart"></canvas>
                </div>

                <!-- Detail Rincian Persentase Kategori -->
                <div id="expensePercentDetails" class="space-y-2 pt-2 border-t border-slate-100">
                    <!-- Dynamically populated -->
                </div>
            </section>

            <!-- Daftar Transaksi -->
            <section id="transactionSection" class="space-y-3">
                <div class="flex items-center justify-between">
                    <h3 class="text-xs font-bold text-slate-700 flex items-center gap-1.5">
                        <i data-lucide="receipt" class="w-4 h-4 text-emerald-600"></i> Riwayat Transaksi
                    </h3>
                    <select id="filterCategory" onchange="renderTransactions()" class="text-xs bg-slate-100 text-slate-600 rounded-lg px-2 py-1 outline-none font-medium border border-slate-200">
                        <option value="all">Semua Jenis</option>
                        <option value="income">Khusus Pemasukan</option>
                        <option value="expense">Khusus Pengeluaran</option>
                        <option value="food">Makanan & Belanja</option>
                        <option value="bills">Tagihan & Listrik</option>
                        <option value="transport">Transportasi</option>
                        <option value="health">Kesehatan</option>
                        <option value="other">Lain-lain</option>
                    </select>
                </div>

                <div id="transactionList" class="space-y-2 pb-4">
                    <!-- Transaksi akan dimuat di sini -->
                </div>
            </section>
        </main>

        <!-- NAVIGASI BAWAH TERKUNCI (FIXED BOTTOM NAVBAR) -->
        <nav class="bg-white border-t border-slate-200 shrink-0 px-4 py-2 z-20">
            <div class="flex items-center justify-around relative">
                <!-- Home -->
                <button onclick="scrollToTop()" class="flex flex-col items-center text-emerald-600 font-semibold text-[10px] space-y-0.5">
                    <i data-lucide="home" class="w-5 h-5"></i>
                    <span>Beranda</span>
                </button>

                <!-- Riwayat / Arsip -->
                <button onclick="openArchiveModal()" class="flex flex-col items-center text-slate-400 hover:text-slate-600 font-semibold text-[10px] space-y-0.5">
                    <i data-lucide="history" class="w-5 h-5"></i>
                    <span>Arsip</span>
                </button>

                <!-- Floating Center Button (+) Tambah Transaksi -->
                <div class="relative -top-5">
                    <button onclick="openTransactionModal()" id="btnFloatingAdd" class="w-12 h-12 bg-gradient-to-tr from-emerald-600 to-teal-500 text-white rounded-full flex items-center justify-center shadow-lg hover:scale-105 active:scale-95 transition">
                        <i data-lucide="plus" class="w-6 h-6 stroke-[2.5]"></i>
                    </button>
                </div>

                <!-- Ekspor -->
                <button onclick="openExportModal()" class="flex flex-col items-center text-slate-400 hover:text-slate-600 font-semibold text-[10px] space-y-0.5">
                    <i data-lucide="download" class="w-5 h-5"></i>
                    <span>Ekspor</span>
                </button>

                <!-- Pengaturan -->
                <button onclick="openSettingsModal()" class="flex flex-col items-center text-slate-400 hover:text-slate-600 font-semibold text-[10px] space-y-0.5">
                    <i data-lucide="settings" class="w-5 h-5"></i>
                    <span>Pengaturan</span>
                </button>
            </div>
        </nav>
    </div>

    <!-- MODAL TAMBAH / EDIT TRANSAKSI -->
    <div id="transactionModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-white w-full max-w-sm rounded-3xl p-5 shadow-xl space-y-4">
            <div class="flex justify-between items-center border-b pb-3">
                <h3 id="modalTitle" class="text-sm font-bold text-slate-800">Tambah Transaksi Baru</h3>
                <button onclick="closeModal('transactionModal')" class="text-slate-400 hover:text-slate-600">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>

            <form id="transactionForm" onsubmit="saveTransaction(event)" class="space-y-3 text-xs">
                <input type="hidden" id="editTxId">
                <div>
                    <label class="font-medium text-slate-600 block mb-1">Tipe Transaksi</label>
                    <select id="txType" class="w-full bg-slate-100 p-2.5 rounded-xl outline-none font-bold text-slate-700" onchange="toggleCategorySelect()">
                        <option value="expense">Pengeluaran (-)</option>
                        <option value="income">Pemasukan (+)</option>
                    </select>
                </div>

                <div>
                    <label class="font-medium text-slate-600 block mb-1">Jumlah (Rp)</label>
                    <input type="number" id="txAmount" required placeholder="Contoh: 50000" class="w-full bg-slate-100 p-2.5 rounded-xl outline-none font-bold text-slate-800 text-sm">
                </div>

                <div id="categoryContainer">
                    <label class="font-medium text-slate-600 block mb-1">Kategori</label>
                    <select id="txCategory" class="w-full bg-slate-100 p-2.5 rounded-xl outline-none font-medium text-slate-700">
                        <option value="food">Makanan & Belanja</option>
                        <option value="bills">Tagihan & Listrik</option>
                        <option value="transport">Transportasi</option>
                        <option value="health">Kesehatan</option>
                        <option value="other">Lain-lain</option>
                    </select>
                </div>

                <div>
                    <label class="font-medium text-slate-600 block mb-1">Metode Pembayaran</label>
                    <select id="txMethod" class="w-full bg-slate-100 p-2.5 rounded-xl outline-none font-medium text-slate-700">
                        <option value="Tunai">Tunai</option>
                        <option value="Transfer Bank">Transfer Bank</option>
                        <option value="E-Wallet">E-Wallet</option>
                    </select>
                </div>

                <div>
                    <label class="font-medium text-slate-600 block mb-1">Tanggal</label>
                    <input type="date" id="txDate" required class="w-full bg-slate-100 p-2 rounded-xl outline-none text-slate-700">
                </div>

                <div>
                    <label class="font-medium text-slate-600 block mb-1">Catatan / Keterangan</label>
                    <input type="text" id="txNote" placeholder="Catatan singkat..." class="w-full bg-slate-100 p-2.5 rounded-xl outline-none text-slate-700">
                </div>

                <div class="pt-2 flex space-x-2">
                    <button type="button" onclick="closeModal('transactionModal')" class="w-1/2 py-2.5 bg-slate-100 text-slate-600 rounded-xl font-semibold">Batal</button>
                    <button type="submit" class="w-1/2 py-2.5 bg-emerald-600 text-white rounded-xl font-semibold hover:bg-emerald-700">Simpan</button>
                </div>
            </form>
        </div>
    </div>

    <!-- MODAL ARSIP & RIWAYAT BULANAN -->
    <div id="archiveModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-white w-full max-w-sm rounded-3xl p-5 shadow-xl space-y-4">
            <div class="flex justify-between items-center border-b pb-3">
                <h3 class="text-sm font-bold text-slate-800 flex items-center gap-1.5">
                    <i data-lucide="archive" class="w-4 h-4 text-emerald-600"></i> Riwayat Arsip Bulanan
                </h3>
                <button onclick="closeModal('archiveModal')" class="text-slate-400 hover:text-slate-600">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>
            <div id="archiveList" class="space-y-2 max-h-60 overflow-y-auto text-xs">
                <!-- Archived months list -->
            </div>
        </div>
    </div>

    <!-- MODAL PENGATURAN -->
    <div id="settingsModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-white w-full max-w-sm rounded-3xl p-5 shadow-xl space-y-4 max-h-[90vh] overflow-y-auto">
            <div class="flex justify-between items-center border-b pb-3">
                <h3 class="text-sm font-bold text-slate-800 flex items-center gap-1.5">
                    <i data-lucide="settings" class="w-4 h-4 text-emerald-600"></i> Pengaturan Aplikasi
                </h3>
                <button onclick="closeModal('settingsModal')" class="text-slate-400 hover:text-slate-600">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>

            <div class="space-y-4 text-xs">
                <!-- 1. Nama Dompet -->
                <div>
                    <label class="font-bold text-slate-700 block mb-1">Nama Judul Dompet</label>
                    <input type="text" id="settingWalletTitle" oninput="updateWalletTitle(this.value)" class="w-full bg-slate-100 p-2.5 rounded-xl outline-none font-medium text-slate-800">
                </div>

                <!-- 2. Tema Warna -->
                <div>
                    <label class="font-bold text-slate-700 block mb-1.5">Tema Warna Header</label>
                    <div class="grid grid-cols-5 gap-2">
                        <button onclick="setTheme('emerald')" class="h-8 rounded-xl bg-gradient-to-r from-emerald-600 to-teal-700 border-2 border-slate-300"></button>
                        <button onclick="setTheme('indigo')" class="h-8 rounded-xl bg-gradient-to-r from-indigo-600 to-blue-700 border-2 border-slate-300"></button>
                        <button onclick="setTheme('violet')" class="h-8 rounded-xl bg-gradient-to-r from-violet-600 to-purple-700 border-2 border-slate-300"></button>
                        <button onclick="setTheme('rose')" class="h-8 rounded-xl bg-gradient-to-r from-rose-600 to-pink-700 border-2 border-slate-300"></button>
                        <button onclick="setTheme('amber')" class="h-8 rounded-xl bg-gradient-to-r from-amber-600 to-orange-700 border-2 border-slate-300"></button>
                    </div>
                </div>

                <!-- 3. Margin Kiri Kanan -->
                <div>
                    <div class="flex justify-between mb-1">
                        <label class="font-bold text-slate-700">Margin Samping (Kanan-Kiri)</label>
                        <span id="sidePaddingVal" class="text-slate-400">16px</span>
                    </div>
                    <input type="range" min="8" max="32" value="16" oninput="updateSidePadding(this.value)" class="w-full accent-emerald-600">
                </div>

                <!-- 4. Ukuran Font -->
                <div>
                    <div class="flex justify-between mb-1">
                        <label class="font-bold text-slate-700">Ukuran Font Teks</label>
                        <span id="fontSizeVal" class="text-slate-400">100%</span>
                    </div>
                    <input type="range" min="85" max="115" value="100" oninput="updateFontSize(this.value)" class="w-full accent-emerald-600">
                </div>

                <!-- 5. Reset Data -->
                <div class="pt-2 border-t border-slate-100">
                    <button onclick="resetAllData()" class="w-full py-2.5 bg-rose-50 text-rose-600 border border-rose-200 rounded-xl font-bold hover:bg-rose-100 transition">
                        Reset / Hapus Semua Data
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- MODAL EKSPOR -->
    <div id="exportModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-white w-full max-w-sm rounded-3xl p-5 shadow-xl space-y-4">
            <div class="flex justify-between items-center border-b pb-3">
                <h3 class="text-sm font-bold text-slate-800 flex items-center gap-1.5">
                    <i data-lucide="download" class="w-4 h-4 text-emerald-600"></i> Ekspor Data
                </h3>
                <button onclick="closeModal('exportModal')" class="text-slate-400 hover:text-slate-600">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>
            <div class="space-y-2 text-xs">
                <button onclick="exportCSV()" class="w-full p-3 bg-slate-50 border border-slate-200 rounded-2xl flex items-center space-x-3 hover:bg-slate-100 transition">
                    <div class="p-2 bg-emerald-100 text-emerald-700 rounded-xl">
                        <i data-lucide="file-spreadsheet" class="w-5 h-5"></i>
                    </div>
                    <div class="text-left">
                        <h4 class="font-bold text-slate-700">Unduh Format CSV (Excel)</h4>
                        <p class="text-[10px] text-slate-400">Rekap transaksi bulan aktif dalam file spreadsheet</p>
                    </div>
                </button>
                <button onclick="exportJSON()" class="w-full p-3 bg-slate-50 border border-slate-200 rounded-2xl flex items-center space-x-3 hover:bg-slate-100 transition">
                    <div class="p-2 bg-blue-100 text-blue-700 rounded-xl">
                        <i data-lucide="file-json" class="w-5 h-5"></i>
                    </div>
                    <div class="text-left">
                        <h4 class="font-bold text-slate-700">Backup Seluruh Data (JSON)</h4>
                        <p class="text-[10px] text-slate-400">Unduh seluruh riwayat transaksi & status arsip</p>
                    </div>
                </button>
            </div>
        </div>
    </div>

    <!-- TOAST NOTIFIKASI -->
    <div id="toast" class="fixed bottom-16 left-1/2 -translate-x-1/2 bg-slate-800 text-white text-xs px-4 py-2 rounded-full shadow-lg opacity-0 pointer-events-none transition-all duration-300 z-50">
        Notifikasi
    </div>

    <!-- JAVASCRIPT LOGIC -->
    <script>
        // State Global
        let transactions = JSON.parse(localStorage.getItem('hh_transactions')) || [];
        let archivedMonths = JSON.parse(localStorage.getItem('hh_archives')) || {};
        let settings = JSON.parse(localStorage.getItem('hh_settings')) || {
            walletTitle: 'Dompet Rumah Tangga',
            theme: 'emerald',
            sidePadding: 16,
            fontSize: 100
        };

        let currentMonth = new Date().toISOString().slice(0, 7); // Format: YYYY-MM
        let chartInstance = null;

        const CATEGORIES = {
            income: { icon: 'arrow-down-left', label: 'Pemasukan', color: 'bg-emerald-100 text-emerald-700' },
            food: { icon: 'shopping-bag', label: 'Makanan & Belanja', color: 'bg-amber-100 text-amber-700' },
            bills: { icon: 'zap', label: 'Tagihan & Listrik', color: 'bg-blue-100 text-blue-700' },
            transport: { icon: 'car', label: 'Transportasi', color: 'bg-pink-100 text-pink-700' },
            health: { icon: 'activity', label: 'Kesehatan', color: 'bg-emerald-100 text-emerald-700' },
            other: { icon: 'more-horizontal', label: 'Lain-lain', color: 'bg-purple-100 text-purple-700' }
        };

        const THEMES = {
            emerald: 'from-emerald-600 to-teal-700',
            indigo: 'from-indigo-600 to-blue-700',
            violet: 'from-violet-600 to-purple-700',
            rose: 'from-rose-600 to-pink-700',
            amber: 'from-amber-600 to-orange-700'
        };

        // Init App pada Load
        window.onload = function() {
            document.getElementById('monthPicker').value = currentMonth;
            applySettings();
            updateAppView();
            lucide.createIcons();
        };

        function escapeHtml(text) {
            if (!text) return '';
            return text.replace(/[&<>"']/g, function(m) {
                return { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#039;' }[m];
            });
        }

        function formatRupiah(amount) {
            return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', maximumFractionDigits: 0 }).format(amount);
        }

        function formatDate(dateStr) {
            if (!dateStr) return '';
            const d = new Date(dateStr);
            return d.toLocaleDateString('id-ID', { day: 'numeric', month: 'short' });
        }

        function showToast(msg) {
            const toast = document.getElementById('toast');
            toast.innerText = msg;
            toast.classList.remove('opacity-0', 'pointer-events-none');
            setTimeout(() => {
                toast.classList.add('opacity-0', 'pointer-events-none');
            }, 2500);
        }

        function scrollToTop() {
            document.querySelector('main').scrollTo({ top: 0, behavior: 'smooth' });
        }

        // Filter dari klik Card Pemasukan / Pengeluaran
        function filterByCardType(type) {
            const select = document.getElementById('filterCategory');
            select.value = type;
            renderTransactions();
            const section = document.getElementById('transactionSection');
            section.scrollIntoView({ behavior: 'smooth' });
        }

        function changeMonth(monthVal) {
            currentMonth = monthVal;
            updateAppView();
        }

        function isCurrentMonthArchived() {
            return !!archivedMonths[currentMonth];
        }

        function getFilteredTransactionsBySelectedMonth() {
            return transactions.filter(t => t.date && t.date.startsWith(currentMonth));
        }

        // Update seluruh tampilan sesuai data bulan terpilih
        function updateAppView() {
            const filteredTx = getFilteredTransactionsBySelectedMonth();

            let totalIncome = 0;
            let totalExpense = 0;
            const expenseCategoryMap = { food: 0, bills: 0, transport: 0, health: 0, other: 0 };

            filteredTx.forEach(t => {
                const amt = parseFloat(t.amount) || 0;
                if (t.type === 'income') {
                    totalIncome += amt;
                } else {
                    totalExpense += amt;
                    if (expenseCategoryMap[t.category] !== undefined) {
                        expenseCategoryMap[t.category] += amt;
                    } else {
                        expenseCategoryMap.other += amt;
                    }
                }
            });

            const totalBalance = totalIncome - totalExpense;

            // Update UI Nominal
            document.getElementById('totalBalance').innerText = formatRupiah(totalBalance);
            document.getElementById('totalIncome').innerText = formatRupiah(totalIncome);
            document.getElementById('totalExpense').innerText = formatRupiah(totalExpense);

            // Update UI Status Arsip
            const isArchived = isCurrentMonthArchived();
            const archiveBadge = document.getElementById('archiveBadge');
            const btnFloatingAdd = document.getElementById('btnFloatingAdd');

            if (isArchived) {
                archiveBadge.classList.remove('hidden');
                if (btnFloatingAdd) btnFloatingAdd.classList.add('opacity-50', 'pointer-events-none');
            } else {
                archiveBadge.classList.add('hidden');
                if (btnFloatingAdd) btnFloatingAdd.classList.remove('opacity-50', 'pointer-events-none');
            }

            // Render komponen child
            renderTransactions();
            renderChart(expenseCategoryMap, totalExpense);
            lucide.createIcons();
        }

        // Render List Transaksi
        function renderTransactions() {
            const listEl = document.getElementById('transactionList');
            if (!listEl) return;
            
            const categoryFilter = document.getElementById('filterCategory').value;
            const monthFiltered = getFilteredTransactionsBySelectedMonth();

            const finalFiltered = monthFiltered.filter(t => {
                if (categoryFilter === 'all') return true;
                if (categoryFilter === 'income') return t.type === 'income';
                if (categoryFilter === 'expense') return t.type === 'expense';
                return t.category === categoryFilter;
            });

            if (finalFiltered.length === 0) {
                listEl.innerHTML = `
                    <div class="text-center py-8 bg-white/70 rounded-2xl border border-dashed border-slate-200">
                        <i data-lucide="inbox" class="w-8 h-8 mx-auto text-slate-300 mb-2"></i>
                        <p class="text-xs text-slate-400 font-medium">Belum ada transaksi di bulan ini</p>
                    </div>
                `;
                lucide.createIcons();
                return;
            }

            listEl.innerHTML = finalFiltered.map(t => {
                const isIncome = t.type === 'income';
                const catKey = isIncome ? 'income' : t.category;
                const catInfo = CATEGORIES[catKey] || { icon: 'tag', label: t.category, color: 'bg-slate-100 text-slate-600' };
                const isLocked = isCurrentMonthArchived();

                return `
                    <div class="bg-white p-3 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between hover:border-slate-200 transition">
                        <div class="flex items-center space-x-3">
                            <div class="p-2 rounded-xl ${catInfo.color}">
                                <i data-lucide="${catInfo.icon}" class="w-4 h-4"></i>
                            </div>
                            <div>
                                <h4 class="text-xs font-bold text-slate-700">${escapeHtml(t.note || catInfo.label)}</h4>
                                <div class="flex items-center space-x-2 text-[10px] text-slate-400">
                                    <span>${formatDate(t.date)}</span>
                                    <span>•</span>
                                    <span class="capitalize">${t.method || 'Tunai'}</span>
                                </div>
                            </div>
                        </div>
                        <div class="flex items-center space-x-2">
                            <span class="text-xs font-extrabold ${isIncome ? 'text-emerald-600' : 'text-rose-600'}">
                                ${isIncome ? '+' : '-'}${formatRupiah(t.amount)}
                            </span>
                            ${!isLocked ? `
                                <div class="flex items-center space-x-1">
                                    <button onclick="openTransactionModal('${t.id}')" class="p-1 text-slate-300 hover:text-slate-600 transition">
                                        <i data-lucide="pencil" class="w-3.5 h-3.5"></i>
                                    </button>
                                    <button onclick="deleteTransaction('${t.id}')" class="p-1 text-slate-300 hover:text-rose-500 transition">
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

        // Render Donut Chart & Percentages
        function renderChart(expenseCategoryMap, totalExpense) {
            const ctx = document.getElementById('expenseChart');
            if (!ctx) return;
            
            const detailContainer = document.getElementById('expensePercentDetails');

            try {
                if (chartInstance) {
                    chartInstance.destroy();
                    chartInstance = null;
                }
                const existingChart = Chart.getChart(ctx);
                if (existingChart) existingChart.destroy();
            } catch(e) {
                console.warn("Chart destroy warning:", e);
            }

            const categories = ['food', 'bills', 'transport', 'health', 'other'];
            const dataValues = categories.map(cat => expenseCategoryMap[cat] || 0);
            const labels = categories.map(cat => CATEGORIES[cat].label);
            const colors = ['#f59e0b', '#3b82f6', '#ec4899', '#10b981', '#8b5cf6'];

            if (totalExpense === 0) {
                ctx.parentElement.classList.add('hidden');
                if(detailContainer) detailContainer.innerHTML = '<p class="text-center text-[10px] text-slate-400 py-2">Belum ada pengeluaran di bulan ini.</p>';
                return;
            }

            ctx.parentElement.classList.remove('hidden');

            chartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [{
                        data: dataValues,
                        backgroundColor: colors,
                        borderWidth: 2,
                        borderColor: '#ffffff'
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
                                font: { size: 10, family: 'Inter' },
                                generateLabels: function(chart) {
                                    const data = chart.data;
                                    if (data.labels.length && data.datasets.length) {
                                        return data.labels.map(function(label, i) {
                                            const meta = chart.getDatasetMeta(0);
                                            const style = meta.controller.getStyle(i);
                                            const value = data.datasets[0].data[i];
                                            const percentage = totalExpense > 0 ? ((value / totalExpense) * 100).toFixed(1) : 0;

                                            return {
                                                text: `${label} (${percentage}%)`,
                                                fillStyle: style.backgroundColor,
                                                strokeStyle: style.borderColor,
                                                lineWidth: style.borderWidth,
                                                hidden: isNaN(data.datasets[0].data[i]) || meta.data[i].hidden,
                                                index: i
                                            };
                                        });
                                    }
                                    return [];
                                }
                            }
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    const value = context.raw || 0;
                                    const percentage = totalExpense > 0 ? ((value / totalExpense) * 100).toFixed(1) : 0;
                                    return ` ${context.label}: ${formatRupiah(value)} (${percentage}%)`;
                                }
                            }
                        }
                    },
                    cutout: '70%'
                }
            });

            if (detailContainer) {
                detailContainer.innerHTML = categories.map((catKey, idx) => {
                    const amount = expenseCategoryMap[catKey] || 0;
                    if (amount === 0) return '';
                    const percentage = totalExpense > 0 ? ((amount / totalExpense) * 100).toFixed(1) : 0;
                    const catInfo = CATEGORIES[catKey];
                    const barColor = colors[idx];

                    return `
                        <div class="space-y-1">
                            <div class="flex justify-between items-center text-[11px]">
                                <span class="font-medium text-slate-600">${catInfo.label}</span>
                                <span class="font-bold text-slate-700">${formatRupiah(amount)} <span class="text-slate-400 font-normal">(${percentage}%)</span></span>
                            </div>
                            <div class="w-full bg-slate-100 rounded-full h-1.5 overflow-hidden">
                                <div class="h-full rounded-full transition-all duration-500" style="width: ${percentage}%; background-color: ${barColor}"></div>
                            </div>
                        </div>
                    `;
                }).join('');
            }
        }

        // Modal Handlers
        function openTransactionModal(editId = null) {
            if (isCurrentMonthArchived()) {
                showToast("Bulan ini telah terarsip/dikunci. Tidak dapat menambah data.");
                return;
            }

            const modal = document.getElementById('transactionModal');
            const form = document.getElementById('transactionForm');
            const title = document.getElementById('modalTitle');

            form.reset();
            document.getElementById('editTxId').value = '';
            document.getElementById('txDate').valueAsDate = new Date();

            if (editId) {
                const tx = transactions.find(t => t.id === editId);
                if (tx) {
                    title.innerText = 'Edit Transaksi';
                    document.getElementById('editTxId').value = tx.id;
                    document.getElementById('txType').value = tx.type;
                    document.getElementById('txCategory').value = tx.category || 'food';
                    document.getElementById('txAmount').value = tx.amount;
                    document.getElementById('txMethod').value = tx.method || 'Tunai';
                    document.getElementById('txDate').value = tx.date;
                    document.getElementById('txNote').value = tx.note || '';
                }
            } else {
                title.innerText = 'Tambah Transaksi Baru';
            }

            toggleCategorySelect();
            modal.classList.remove('hidden');
        }

        function toggleCategorySelect() {
            const type = document.getElementById('txType').value;
            const container = document.getElementById('categoryContainer');
            if (type === 'income') {
                container.classList.add('hidden');
            } else {
                container.classList.remove('hidden');
            }
        }

        function closeModal(modalId) {
            document.getElementById(modalId).classList.add('hidden');
        }

        // Simpan Transaksi
        function saveTransaction(e) {
            e.preventDefault();
            const editId = document.getElementById('editTxId').value;
            const type = document.getElementById('txType').value;
            const amount = parseFloat(document.getElementById('txAmount').value) || 0;
            const category = type === 'income' ? 'income' : document.getElementById('txCategory').value;
            const method = document.getElementById('txMethod').value;
            const date = document.getElementById('txDate').value;
            const note = document.getElementById('txNote').value.trim();

            if (editId) {
                transactions = transactions.map(t => t.id === editId ? { id: editId, type, amount, category, method, date, note } : t);
                showToast("Transaksi berhasil diperbarui");
            } else {
                const newTx = {
                    id: Date.now().toString(),
                    type,
                    amount,
                    category,
                    method,
                    date,
                    note
                };
                transactions.unshift(newTx);
                showToast("Transaksi berhasil ditambahkan");
            }

            localStorage.setItem('hh_transactions', JSON.stringify(transactions));
            closeModal('transactionModal');
            updateAppView();
        }

        function deleteTransaction(id) {
            if (isCurrentMonthArchived()) {
                showToast("Bulan ini telah terarsip.");
                return;
            }
            transactions = transactions.filter(t => t.id !== id);
            localStorage.setItem('hh_transactions', JSON.stringify(transactions));
            showToast("Transaksi berhasil dihapus");
            updateAppView();
        }

        // Toggle Kunci Bulan Ini
        function toggleArchiveStatus() {
            if (archivedMonths[currentMonth]) {
                delete archivedMonths[currentMonth];
                showToast(`Status arsip bulan ${currentMonth} dibuka`);
            } else {
                archivedMonths[currentMonth] = true;
                showToast(`Bulan ${currentMonth} berhasil dikunci/diarsip`);
            }
            localStorage.setItem('hh_archives', JSON.stringify(archivedMonths));
            updateAppView();
        }

        function openArchiveModal() {
            const modal = document.getElementById('archiveModal');
            const list = document.getElementById('archiveList');

            const months = Object.keys(archivedMonths).sort().reverse();
            if (months.length === 0) {
                list.innerHTML = '<p class="text-center text-slate-400 py-4">Belum ada bulan yang diarsip.</p>';
            } else {
                list.innerHTML = months.map(m => `
                    <div class="p-3 bg-slate-50 border border-slate-200 rounded-xl flex justify-between items-center">
                        <span class="font-bold text-slate-700">${m}</span>
                        <button onclick="changeMonth('${m}'); closeModal('archiveModal');" class="text-emerald-600 font-semibold text-xs">
                            Buka Bulan ↗
                        </button>
                    </div>
                `).join('');
            }

            modal.classList.remove('hidden');
        }

        // Pengaturan Handler
        function openSettingsModal() {
            document.getElementById('settingWalletTitle').value = settings.walletTitle;
            document.getElementById('settingsModal').classList.remove('hidden');
        }

        function updateWalletTitle(val) {
            settings.walletTitle = val || 'Dompet Rumah Tangga';
            document.getElementById('walletTitleDisplay').innerText = settings.walletTitle;
            saveSettings();
        }

        function setTheme(themeKey) {
            settings.theme = themeKey;
            const header = document.getElementById('headerTheme');
            header.className = `bg-gradient-to-r ${THEMES[themeKey]} text-white shrink-0 pt-5 pb-5 transition-colors duration-300 shadow-md`;
            saveSettings();
        }

        function updateSidePadding(val) {
            settings.sidePadding = val;
            document.documentElement.style.setProperty('--side-padding', `${val}px`);
            document.getElementById('sidePaddingVal').innerText = `${val}px`;
            saveSettings();
        }

        function updateFontSize(val) {
            settings.fontSize = val;
            document.documentElement.style.setProperty('--font-scale', `${val}%`);
            document.getElementById('fontSizeVal').innerText = `${val}%`;
            saveSettings();
        }

        function applySettings() {
            document.getElementById('walletTitleDisplay').innerText = settings.walletTitle;
            setTheme(settings.theme || 'emerald');
            document.documentElement.style.setProperty('--side-padding', `${settings.sidePadding || 16}px`);
            document.documentElement.style.setProperty('--font-scale', `${settings.fontSize || 100}%`);
        }

        function saveSettings() {
            localStorage.setItem('hh_settings', JSON.stringify(settings));
        }

        function resetAllData() {
            if (confirm("Apakah Anda yakin ingin menghapus seluruh data transaksi dan pengaturan?")) {
                localStorage.clear();
                location.reload();
            }
        }

        // Ekspor Handler
        function openExportModal() {
            document.getElementById('exportModal').classList.remove('hidden');
        }

        function exportCSV() {
            const filtered = getFilteredTransactionsBySelectedMonth();
            if (filtered.length === 0) {
                showToast("Tidak ada data untuk diekspor");
                return;
            }

            let csvContent = "data:text/csv;charset=utf-8,ID,Tanggal,Tipe,Kategori,Jumlah,Metode,Catatan\n";
            filtered.forEach(t => {
                csvContent += `"${t.id}","${t.date}","${t.type}","${t.category}","${t.amount}","${t.method || ''}","${t.note || ''}"\n`;
            });

            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", `Pengeluaran_${currentMonth}.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            closeModal('exportModal');
        }

        function exportJSON() {
            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify({
                transactions,
                archivedMonths,
                settings
            }, null, 2));

            const link = document.createElement("a");
            link.setAttribute("href", dataStr);
            link.setAttribute("download", `Backup_Dompet_${currentMonth}.json`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            closeModal('exportModal');
        }
    </script>
</body>
</html>
