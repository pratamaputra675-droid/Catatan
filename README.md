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
        :root {
            --side-padding: 20px;
            --font-scale: 100%;
        }
        html {
            font-size: var(--font-scale);
        }
        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
            -webkit-tap-highlight-color: transparent;
        }
        .dynamic-padding {
            padding-left: var(--side-padding);
            padding-right: var(--side-padding);
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
                    <button onclick="openSettingsModal()" class="bg-white/20 hover:bg-white/30 backdrop-blur-md p-2 rounded-full text-white transition" title="Pengaturan">
                        <i data-lucide="settings" class="w-4.5 h-4.5"></i>
                    </button>
                </div>
            </div>

            <!-- Balance Card -->
            <div class="bg-white/10 backdrop-blur-md border border-white/20 rounded-2xl p-4 text-white shadow-inner relative overflow-hidden">
                <div class="flex justify-between items-start">
                    <div>
                        <span class="text-[11px] text-emerald-100 font-semibold tracking-wide uppercase" id="appTitleDisplay">Sisa Saldo Bulan Ini</span>
                        <h2 class="text-2xl font-extrabold tracking-tight mt-0.5" id="remainingBalanceDisplay">Rp 0</h2>
                    </div>
                    <span id="archiveBadge" class="hidden text-[10px] bg-amber-400 text-slate-900 font-bold px-2.5 py-0.5 rounded-full flex items-center gap-1 shadow-sm">
                        <i data-lucide="lock" class="w-3 h-3"></i> Terarsip
                    </span>
                </div>
            </div>
        </header>

        <!-- Main Content Area -->
        <main id="mainContainer" class="flex-1 overflow-y-auto dynamic-padding py-4 space-y-4 no-scrollbar">
            <!-- Monthly Lock Banner -->
            <div class="bg-white p-3.5 rounded-2xl shadow-sm border border-slate-100 flex items-center justify-between">
                <div class="flex items-center gap-2.5">
                    <div class="w-8 h-8 rounded-xl bg-emerald-600 text-white flex items-center justify-center shrink-0" id="archiveBannerIcon">
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
                <div onclick="filterSummaryType('INCOME')" class="bg-white p-2.5 rounded-2xl shadow-sm border border-slate-100 flex items-center gap-2.5 cursor-pointer hover:border-emerald-300 hover:shadow-md transition active:scale-95 group" title="Klik untuk lihat riwayat Pemasukan">
                    <div class="w-7 h-7 rounded-lg bg-emerald-100 text-emerald-600 flex items-center justify-center shrink-0 group-hover:bg-emerald-600 group-hover:text-white transition">
                        <i data-lucide="arrow-down-left" class="w-3.5 h-3.5"></i>
                    </div>
                    <div class="overflow-hidden">
                        <p class="text-[10px] text-slate-400 font-medium leading-tight group-hover:text-emerald-600 transition">Pemasukan</p>
                        <p class="text-xs font-bold text-slate-800 truncate" id="totalIncomeDisplay">Rp 0</p>
                    </div>
                </div>

                <div onclick="filterSummaryType('EXPENSE')" class="bg-white p-2.5 rounded-2xl shadow-sm border border-slate-100 flex items-center gap-2.5 cursor-pointer hover:border-rose-300 hover:shadow-md transition active:scale-95 group" title="Klik untuk lihat riwayat Pengeluaran">
                    <div class="w-7 h-7 rounded-lg bg-rose-100 text-rose-600 flex items-center justify-center shrink-0 group-hover:bg-rose-600 group-hover:text-white transition">
                        <i data-lucide="arrow-up-right" class="w-3.5 h-3.5"></i>
                    </div>
                    <div class="overflow-hidden">
                        <p class="text-[10px] text-slate-400 font-medium leading-tight group-hover:text-rose-600 transition">Pengeluaran</p>
                        <p class="text-xs font-bold text-slate-800 truncate" id="subTotalExpenseDisplay">Rp 0</p>
                    </div>
                </div>
            </div>

            <!-- Categories Breakdown Chart -->
            <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-100">
                <div class="flex justify-between items-center mb-2">
                    <h3 class="text-sm font-bold text-slate-700">Analisis Pengeluaran</h3>
                    <span class="text-[10px] bg-slate-100 text-slate-500 font-semibold px-2 py-0.5 rounded-md" id="chartMonthLabel">Bulan Ini</span>
                </div>
                <div class="relative w-full h-44 flex items-center justify-center">
                    <canvas id="categoryChart"></canvas>
                    <div id="noChartData" class="hidden absolute inset-0 flex flex-col items-center justify-center text-slate-400 bg-white/90">
                        <i data-lucide="pie-chart" class="w-8 h-8 mb-1 stroke-1"></i>
                        <p class="text-xs">Belum ada data pengeluaran</p>
                    </div>
                </div>
                <!-- Percentage Breakdown List -->
                <div id="categoryPercentageList" class="mt-3 pt-3 border-t border-slate-100 space-y-2">
                    <!-- Dynamic percentage list items -->
                </div>
            </div>

            <!-- Transactions Section -->
            <div class="space-y-3 pb-24">
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
                <div id="transactionList" class="space-y-2.5">
                    <!-- Dynamic rendering via JS -->
                </div>
            </div>
        </main>

        <!-- Bottom Navigation Bar (Fixed / Sticky Positioned) -->
        <nav class="shrink-0 bg-white border-t border-slate-200/80 px-4 py-2 z-40 sticky bottom-0 left-0 right-0 shadow-lg flex justify-around items-center">
            <!-- Menu 1: Home / Dashboard -->
            <button onclick="scrollToTop()" class="flex flex-col items-center justify-center text-emerald-600 space-y-0.5">
                <i data-lucide="home" class="w-5 h-5"></i>
                <span class="text-[10px] font-bold">Beranda</span>
            </button>

            <!-- Menu 2: History & Archives -->
            <button onclick="openHistoryModal()" class="flex flex-col items-center justify-center text-slate-400 hover:text-slate-600 transition space-y-0.5">
                <i data-lucide="history" class="w-5 h-5"></i>
                <span class="text-[10px] font-medium">Riwayat</span>
            </button>

            <!-- Floating Center Button: Add Transaction -->
            <div class="relative -top-5">
                <button id="btnAddTransaction" onclick="openTransactionModal()" class="bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-700 hover:to-teal-700 text-white p-3.5 rounded-full shadow-lg hover:shadow-xl active:scale-95 transition flex items-center justify-center border-4 border-slate-50" title="Tambah Transaksi">
                    <i data-lucide="plus" class="w-6 h-6 stroke-[2.5]"></i>
                </button>
            </div>

            <!-- Menu 3: Export Data -->
            <button onclick="openExportModal()" class="flex flex-col items-center justify-center text-slate-400 hover:text-slate-600 transition space-y-0.5">
                <i data-lucide="download" class="w-5 h-5"></i>
                <span class="text-[10px] font-medium">Ekspor</span>
            </button>

            <!-- Menu 4: Settings -->
            <button onclick="openSettingsModal()" class="flex flex-col items-center justify-center text-slate-400 hover:text-slate-600 transition space-y-0.5">
                <i data-lucide="settings" class="w-5 h-5"></i>
                <span class="text-[10px] font-medium">Pengaturan</span>
            </button>
        </nav>

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

            <p class="text-xs text-slate-500">Unduh data transaksi untuk bulan <span id="exportMonthName" class="font-bold text-slate-700"></span>:</p>

            <div class="grid grid-cols-2 gap-3 pt-2">
                <button onclick="exportData('csv')" class="p-4 bg-slate-50 hover:bg-emerald-50 border border-slate-200 hover:border-emerald-300 rounded-2xl flex flex-col items-center justify-center gap-2 transition group">
                    <div class="w-10 h-10 rounded-full bg-emerald-100 text-emerald-600 flex items-center justify-center group-hover:scale-110 transition">
                        <i data-lucide="file-spreadsheet" class="w-5 h-5"></i>
                    </div>
                    <span class="text-xs font-bold text-slate-700">Format CSV</span>
                    <span class="text-[10px] text-slate-400">Untuk Excel / Spreadsheet</span>
                </button>

                <button onclick="exportData('json')" class="p-4 bg-slate-50 hover:bg-emerald-50 border border-slate-200 hover:border-emerald-300 rounded-2xl flex flex-col items-center justify-center gap-2 transition group">
                    <div class="w-10 h-10 rounded-full bg-blue-100 text-blue-600 flex items-center justify-center group-hover:scale-110 transition">
                        <i data-lucide="file-json" class="w-5 h-5"></i>
                    </div>
                    <span class="text-xs font-bold text-slate-700">Format JSON</span>
                    <span class="text-[10px] text-slate-400">Backup Data Mentah</span>
                </button>
            </div>
        </div>
    </div>

    <!-- Modal: Settings -->
    <div id="settingsModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex justify-center items-end sm:items-center hidden transition-opacity duration-300">
        <div class="bg-white w-full max-w-[430px] rounded-t-3xl sm:rounded-3xl p-6 shadow-2xl space-y-4 max-h-[85vh] flex flex-col">
            <div class="flex justify-between items-center border-b pb-3 shrink-0">
                <div class="flex items-center gap-2">
                    <i data-lucide="settings" class="w-5 h-5 text-emerald-600" id="settingsIcon"></i>
                    <h3 class="text-base font-bold text-slate-800">Pengaturan Aplikasi</h3>
                </div>
                <button onclick="closeSettingsModal()" class="text-slate-400 hover:text-slate-600 p-1">
                    <i data-lucide="x" class="w-6 h-6"></i>
                </button>
            </div>

            <div class="flex-1 overflow-y-auto space-y-5 pr-1 no-scrollbar">
                <!-- Custom App Title -->
                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-1">Nama / Judul Dompet</label>
                    <input type="text" id="settingAppTitle" oninput="updateSettingsFromForm()" placeholder="Sisa Saldo Bulan Ini" class="w-full p-2.5 bg-slate-50 border border-slate-200 rounded-xl text-slate-800 text-xs font-semibold focus:outline-none focus:ring-2 focus:ring-emerald-500">
                </div>

                <!-- Horizontal Spacing (Margin / Padding) -->
                <div>
                    <div class="flex justify-between items-center mb-1">
                        <label class="text-xs font-semibold text-slate-700">Margin Kiri - Kanan Layout</label>
                        <span id="paddingValDisplay" class="text-xs font-bold text-slate-500">20px</span>
                    </div>
                    <input type="range" id="settingPadding" min="8" max="36" value="20" oninput="updateSettingsFromForm()" class="w-full accent-emerald-600 cursor-pointer">
                    <div class="flex justify-between text-[10px] text-slate-400 mt-0.5">
                        <span>Rapat (8px)</span>
                        <span>Longgar (36px)</span>
                    </div>
                </div>

                <!-- Font Size Scaling -->
                <div>
                    <div class="flex justify-between items-center mb-1">
                        <label class="text-xs font-semibold text-slate-700">Ukuran Font</label>
                        <span id="fontSizeValDisplay" class="text-xs font-bold text-slate-500">100%</span>
                    </div>
                    <input type="range" id="settingFontSize" min="85" max="115" step="5" value="100" oninput="updateSettingsFromForm()" class="w-full accent-emerald-600 cursor-pointer">
                    <div class="flex justify-between text-[10px] text-slate-400 mt-0.5">
                        <span>Kecil (85%)</span>
                        <span>Normal</span>
                        <span>Besar (115%)</span>
                    </div>
                </div>

                <!-- Color Theme Selector -->
                <div>
                    <label class="block text-xs font-semibold text-slate-700 mb-2">Tema Warna Utama</label>
                    <div class="grid grid-cols-5 gap-2">
                        <button onclick="setTheme('emerald')" class="h-9 rounded-xl bg-gradient-to-r from-emerald-600 to-teal-600 flex items-center justify-center text-white font-bold border-2 transition" id="theme-btn-emerald">✓</button>
                        <button onclick="setTheme('indigo')" class="h-9 rounded-xl bg-gradient-to-r from-indigo-600 to-blue-600 flex items-center justify-center text-white font-bold border-2 border-transparent transition" id="theme-btn-indigo"></button>
                        <button onclick="setTheme('violet')" class="h-9 rounded-xl bg-gradient-to-r from-purple-600 to-pink-600 flex items-center justify-center text-white font-bold border-2 border-transparent transition" id="theme-btn-violet"></button>
                        <button onclick="setTheme('rose')" class="h-9 rounded-xl bg-gradient-to-r from-rose-600 to-amber-600 flex items-center justify-center text-white font-bold border-2 border-transparent transition" id="theme-btn-rose"></button>
                        <button onclick="setTheme('warm')" class="h-9 rounded-xl bg-gradient-to-r from-stone-700 to-amber-800 flex items-center justify-center text-white font-bold border-2 border-transparent transition" id="theme-btn-warm"></button>
                    </div>
                </div>

                <!-- Import Backup File -->
                <div class="border-t pt-4">
                    <label class="block text-xs font-semibold text-slate-700 mb-1.5">Pulihkan Data (Import JSON)</label>
                    <input type="file" id="importFileInput" accept=".json" onchange="importData(event)" class="block w-full text-xs text-slate-500 file:mr-3 file:py-2 file:px-4 file:rounded-xl file:border-0 file:text-xs file:font-semibold file:bg-slate-100 file:text-slate-700 hover:file:bg-slate-200">
                </div>

                <!-- Reset Data Button -->
                <div class="border-t pt-4">
                    <button onclick="resetAllData()" class="w-full bg-rose-50 hover:bg-rose-100 border border-rose-200 text-rose-600 font-bold py-2.5 rounded-xl text-xs flex items-center justify-center gap-2 transition">
                        <i data-lucide="trash-2" class="w-4 h-4"></i>
                        <span>Hapus & Reset Semua Data</span>
                    </button>
                </div>
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
            archivedMonths: {}, // Key: 'YYYY-MM', Value: true/false
            settings: {
                appTitle: 'Sisa Saldo Bulan Ini',
                padding: 20,
                fontSize: 100,
                theme: 'emerald'
            }
        };

        const THEMES = {
            emerald: {
                header: 'from-emerald-600 to-teal-700',
                btn: 'from-emerald-600 to-teal-600 hover:from-emerald-700 hover:to-teal-700 shadow-emerald-600/30',
                accent: 'emerald',
                hex: '#16a34a'
            },
            indigo: {
                header: 'from-indigo-600 to-blue-700',
                btn: 'from-indigo-600 to-blue-600 hover:from-indigo-700 hover:to-blue-700 shadow-indigo-600/30',
                accent: 'indigo',
                hex: '#4f46e5'
            },
            violet: {
                header: 'from-purple-600 to-pink-700',
                btn: 'from-purple-600 to-pink-600 hover:from-purple-700 hover:to-pink-700 shadow-purple-600/30',
                accent: 'purple',
                hex: '#9333ea'
            },
            rose: {
                header: 'from-rose-600 to-amber-700',
                btn: 'from-rose-600 to-amber-600 hover:from-rose-700 hover:to-amber-700 shadow-rose-600/30',
                accent: 'rose',
                hex: '#e11d48'
            },
            warm: {
                header: 'from-stone-700 to-amber-900',
                btn: 'from-stone-700 to-amber-800 hover:from-stone-800 hover:to-amber-900 shadow-stone-700/30',
                accent: 'stone',
                hex: '#44403c'
            }
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
            const val = isNaN(amount) ? 0 : amount;
            return new Intl.NumberFormat('id-ID', {
                style: 'currency',
                currency: 'IDR',
                maximumFractionDigits: 0
            }).format(val);
        }

        function formatMonthLabel(yearMonthStr) {
            if (!yearMonthStr) return '';
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
                    appData.settings = Object.assign({
                        appTitle: 'Sisa Saldo Bulan Ini',
                        padding: 20,
                        fontSize: 100,
                        theme: 'emerald'
                    }, parsed.settings || {});
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
            applySettingsToUI();
        }

        function saveToLocalStorage() {
            localStorage.setItem('dompet_rumah_tangga', JSON.stringify(appData));
        }

        // Apply Settings To Real UI using CSS Variables & Classes
        function applySettingsToUI() {
            const s = appData.settings;
            
            // 1. App Title
            document.getElementById('appTitleDisplay').innerText = s.appTitle || 'Sisa Saldo Bulan Ini';
            document.getElementById('settingAppTitle').value = s.appTitle || 'Sisa Saldo Bulan Ini';

            // 2. Padding Horizontal via CSS Variable
            document.documentElement.style.setProperty('--side-padding', `${s.padding}px`);
            document.getElementById('settingPadding').value = s.padding;
            document.getElementById('paddingValDisplay').innerText = `${s.padding}px`;

            // 3. Font Scale via CSS Variable
            document.documentElement.style.setProperty('--font-scale', `${s.fontSize}%`);
            document.getElementById('settingFontSize').value = s.fontSize;
            document.getElementById('fontSizeValDisplay').innerText = `${s.fontSize}%`;

            // 4. Color Theme
            applyThemeUI(s.theme || 'emerald');
        }

        function applyThemeUI(themeKey) {
            const theme = THEMES[themeKey] || THEMES.emerald;
            const headerEl = document.querySelector('header');
            const btnAddTx = document.getElementById('btnAddTransaction');

            // Reset header gradient classes
            headerEl.className = `bg-gradient-to-r ${theme.header} text-white px-5 pt-6 pb-5 rounded-b-[28px] shadow-lg shrink-0`;

            // Reset Add button gradient classes
            btnAddTx.className = `bg-gradient-to-r ${theme.btn} text-white p-3.5 rounded-full shadow-lg hover:shadow-xl active:scale-95 transition flex items-center justify-center border-4 border-slate-50`;

            // Highlight active theme button in settings
            Object.keys(THEMES).forEach(k => {
                const btn = document.getElementById(`theme-btn-${k}`);
                if (btn) {
                    if (k === themeKey) {
                        btn.innerText = '✓';
                        btn.classList.remove('border-transparent');
                        btn.classList.add('border-white');
                    } else {
                        btn.innerText = '';
                        btn.classList.add('border-transparent');
                        btn.classList.remove('border-white');
                    }
                }
            });
        }

        function updateSettingsFromForm() {
            const appTitle = document.getElementById('settingAppTitle').value || 'Sisa Saldo Bulan Ini';
            const padding = parseInt(document.getElementById('settingPadding').value, 10) || 20;
            const fontSize = parseInt(document.getElementById('settingFontSize').value, 10) || 100;

            appData.settings.appTitle = appTitle;
            appData.settings.padding = padding;
            appData.settings.fontSize = fontSize;

            saveToLocalStorage();
            applySettingsToUI();
        }

        function setTheme(themeKey) {
            appData.settings.theme = themeKey;
            saveToLocalStorage();
            applySettingsToUI();
            showToast(`Tema warna diubah`);
        }

        function openSettingsModal() {
            applySettingsToUI();
            document.getElementById('settingsModal').classList.remove('hidden');
        }

        function closeSettingsModal() {
            document.getElementById('settingsModal').classList.add('hidden');
        }

        function importData(event) {
            const file = event.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const parsed = JSON.parse(e.target.result);
                    if (parsed && Array.isArray(parsed.transactions)) {
                        appData.transactions = parsed.transactions;
                        appData.archivedMonths = parsed.archivedMonths || {};
                        if (parsed.settings) appData.settings = parsed.settings;
                        saveToLocalStorage();
                        renderAll();
                        applySettingsToUI();
                        closeSettingsModal();
                        showToast("Data berhasil dipulihkan!");
                    } else {
                        showToast("Format file JSON tidak valid!");
                    }
                } catch(err) {
                    showToast("Gagal membaca file backup!");
                }
            };
            reader.readAsText(file);
        }

        function resetAllData() {
            if (confirm("Apakah Anda yakin ingin menghapus SELURUH data transaksi & arsip? Tindakan ini tidak dapat dibatalkan!")) {
                appData.transactions = [];
                appData.archivedMonths = {};
                saveToLocalStorage();
                renderAll();
                closeSettingsModal();
                showToast("Seluruh data berhasil dihapus");
            }
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
            const monthlyTxs = appData.transactions.filter(t => t.date && t.date.startsWith(currentSelectedMonth));

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
            const canvas = document.getElementById('categoryChart');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            const noDataEl = document.getElementById('noChartData');

            // Safely destroy existing chart instance
            const existingChart = Chart.getChart(canvas);
            if (existingChart) {
                existingChart.destroy();
            }
            if (chartInstance) {
                chartInstance.destroy();
                chartInstance = null;
            }

            if (totalExpense === 0) {
                noDataEl.classList.remove('hidden');
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

            chartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: labels.map((cat, i) => {
                        const pct = ((data[i] / totalExpense) * 100).toFixed(1);
                        return `${cat} (${pct}%)`;
                    }),
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
                                    const val = context.raw || 0;
                                    const pct = ((val / totalExpense) * 100).toFixed(1);
                                    return ` ${context.label}: ${formatIDR(val)} (${pct}%)`;
                                }
                            }
                        }
                    },
                    cutout: '68%'
                }
            });

            renderCategoryPercentageList(categoryTotals, totalExpense);
        }

        function renderCategoryPercentageList(categoryTotals, totalExpense) {
            const container = document.getElementById('categoryPercentageList');
            if (!container) return;

            if (totalExpense === 0) {
                container.innerHTML = '';
                return;
            }

            const items = [];
            Object.keys(categoryTotals).forEach(cat => {
                const amount = categoryTotals[cat];
                if (amount > 0) {
                    const pct = ((amount / totalExpense) * 100).toFixed(1);
                    const config = CATEGORY_CONFIG[cat] || CATEGORY_CONFIG['Lain-lain'];
                    items.push({ cat, amount, pct: parseFloat(pct), config });
                }
            });

            // Urutkan dari persentase terbesar ke terkecil
            items.sort((a, b) => b.amount - a.amount);

            container.innerHTML = items.map(item => `
                <div class="space-y-1">
                    <div class="flex justify-between items-center text-[11px]">
                        <span class="font-semibold text-slate-700 flex items-center gap-1.5">
                            <span class="w-2 h-2 rounded-full inline-block" style="background-color: ${item.config.color}"></span>
                            ${item.cat}
                        </span>
                        <div class="text-right">
                            <span class="font-bold text-slate-800">${formatIDR(item.amount)}</span>
                            <span class="text-slate-400 font-medium ml-1">(${item.pct}%)</span>
                        </div>
                    </div>
                    <div class="w-full h-1.5 bg-slate-100 rounded-full overflow-hidden">
                        <div class="h-full rounded-full transition-all duration-500" style="width: ${item.pct}%; background-color: ${item.config.color}"></div>
                    </div>
                </div>
            `).join('');
        }

        // Transaction List Rendering
        function renderTransactions() {
            const container = document.getElementById('transactionList');
            if (!container) return;

            const categoryFilter = document.getElementById('filterCategory')?.value || 'ALL';
            const isArchived = !!appData.archivedMonths[currentSelectedMonth];

            // Filter transactions by selected month and category filter
            let txs = appData.transactions.filter(t => t.date && t.date.startsWith(currentSelectedMonth));

            if (categoryFilter !== 'ALL') {
                if (categoryFilter === 'Pemasukan') {
                    txs = txs.filter(t => t.type === 'INCOME');
                } else {
                    txs = txs.filter(t => t.type === 'EXPENSE' && t.category === categoryFilter);
                }
            }

            // Sort descending by date
            txs.sort((a, b) => new Date(b.date) - new Date(a.date));

            if (txs.length === 0) {
                container.innerHTML = `
                    <div class="bg-white p-8 rounded-2xl border border-slate-100 text-center text-slate-400">
                        <i data-lucide="receipt" class="w-10 h-10 mx-auto mb-2 opacity-50 stroke-1"></i>
                        <p class="text-xs font-semibold">Tidak ada transaksi ditemukan</p>
                        <p class="text-[10px] text-slate-400 mt-0.5">Silakan tambah transaksi baru di bulan ini</p>
                    </div>
                `;
                lucide.createIcons();
                return;
            }

            container.innerHTML = txs.map(t => {
                const isExpense = t.type === 'EXPENSE';
                const config = isExpense ? (CATEGORY_CONFIG[t.category] || CATEGORY_CONFIG['Lain-lain']) : CATEGORY_CONFIG['Pemasukan'];
                
                // Format display date: DD MMM
                const dateObj = new Date(t.date);
                const dateFormatted = dateObj.toLocaleDateString('id-ID', { day: 'numeric', month: 'short' });

                return `
                    <div class="bg-white p-3.5 rounded-2xl border border-slate-100 shadow-sm flex items-center justify-between gap-3 hover:border-slate-200 transition">
                        <div class="flex items-center gap-3 min-w-0">
                            <div class="w-10 h-10 rounded-xl ${config.bg} ${config.text} flex items-center justify-center shrink-0">
                                <i data-lucide="${config.icon}" class="w-5 h-5"></i>
                            </div>
                            <div class="min-w-0">
                                <div class="flex items-center gap-2">
                                    <h4 class="text-xs font-bold text-slate-800 truncate">${t.note || (isExpense ? t.category : 'Pemasukan')}</h4>
                                    <span class="text-[9px] px-1.5 py-0.5 rounded bg-slate-100 text-slate-500 font-medium shrink-0">${t.method || 'Tunai'}</span>
                                </div>
                                <p class="text-[10px] text-slate-400 font-medium mt-0.5">${dateFormatted} • ${t.category || 'Pemasukan'}</p>
                            </div>
                        </div>

                        <div class="flex items-center gap-2 shrink-0">
                            <div class="text-right">
                                <p class="text-xs font-extrabold ${isExpense ? 'text-slate-800' : 'text-emerald-600'}">
                                    ${isExpense ? '-' : '+'}${formatIDR(t.amount)}
                                </p>
                            </div>

                            ${!isArchived ? `
                                <div class="flex items-center gap-1 ml-1 border-l border-slate-100 pl-1.5">
                                    <button onclick="editTransaction('${t.id}')" class="p-1 text-slate-400 hover:text-slate-600 transition" title="Edit">
                                        <i data-lucide="edit-3" class="w-3.5 h-3.5"></i>
                                    </button>
                                    <button onclick="deleteTransaction('${t.id}')" class="p-1 text-slate-400 hover:text-rose-500 transition" title="Hapus">
                                        <i data-lucide="trash" class="w-3.5 h-3.5"></i>
                                    </button>
                                </div>
                            ` : ''}
                        </div>
                    </div>
                `;
            }).join('');

            lucide.createIcons();
        }

        // Add / Edit Modal Controls
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

        function openTransactionModal(editId = null) {
            if (appData.archivedMonths[currentSelectedMonth]) {
                showToast("Bulan ini sedang terarsip & terkunci!");
                return;
            }

            const modal = document.getElementById('transactionModal');
            const form = document.getElementById('transactionForm');
            form.reset();

            if (editId) {
                const tx = appData.transactions.find(t => t.id === editId);
                if (!tx) return;

                document.getElementById('modalTitle').innerText = 'Edit Transaksi';
                document.getElementById('editTxId').value = tx.id;
                document.getElementById('txAmount').value = tx.amount;
                document.getElementById('txDate').value = tx.date;
                document.getElementById('txMethod').value = tx.method || 'Tunai';
                document.getElementById('txNote').value = tx.note || '';

                setType(tx.type || 'EXPENSE');
                if (tx.type === 'EXPENSE') {
                    document.getElementById('txCategory').value = tx.category || 'Makanan/Belanja';
                }
            } else {
                document.getElementById('modalTitle').innerText = 'Tambah Transaksi';
                document.getElementById('editTxId').value = '';
                
                // Set default date to today or 1st of selected month
                const todayStr = new Date().toISOString().split('T')[0];
                document.getElementById('txDate').value = todayStr.startsWith(currentSelectedMonth) ? todayStr : `${currentSelectedMonth}-01`;
                setType('EXPENSE');
            }

            modal.classList.remove('hidden');
        }

        function closeTransactionModal() {
            document.getElementById('transactionModal').classList.add('hidden');
        }

        function saveTransaction(e) {
            e.preventDefault();

            const editId = document.getElementById('editTxId').value;
            const type = document.getElementById('txType').value;
            const amount = parseFloat(document.getElementById('txAmount').value);
            const date = document.getElementById('txDate').value;
            const method = document.getElementById('txMethod').value;
            const note = document.getElementById('txNote').value.trim();
            const category = type === 'EXPENSE' ? document.getElementById('txCategory').value : 'Pemasukan';

            if (isNaN(amount) || amount <= 0) {
                showToast("Nominal transaksi harus lebih dari 0");
                return;
            }

            if (editId) {
                // Update
                const index = appData.transactions.findIndex(t => t.id === editId);
                if (index !== -1) {
                    appData.transactions[index] = {
                        id: editId,
                        type,
                        amount,
                        category,
                        method,
                        date,
                        note
                    };
                    showToast("Transaksi berhasil diperbarui");
                }
            } else {
                // Create
                const newTx = {
                    id: Date.now().toString(),
                    type,
                    amount,
                    category,
                    method,
                    date,
                    note
                };
                appData.transactions.push(newTx);
                showToast("Transaksi berhasil ditambahkan");
            }

            saveToLocalStorage();
            closeTransactionModal();
            renderAll();
        }

        function editTransaction(id) {
            openTransactionModal(id);
        }

        function deleteTransaction(id) {
            if (confirm("Hapus transaksi ini?")) {
                appData.transactions = appData.transactions.filter(t => t.id !== id);
                saveToLocalStorage();
                renderAll();
                showToast("Transaksi telah dihapus");
            }
        }

        // History / Archives Modal
        function openHistoryModal() {
            const modal = document.getElementById('historyModal');
            const list = document.getElementById('historyList');

            // Collect all unique YYYY-MM from transactions & archivedMonths
            const monthSet = new Set(Object.keys(appData.archivedMonths));
            appData.transactions.forEach(t => {
                if (t.date) monthSet.add(t.date.substring(0, 7));
            });

            const sortedMonths = Array.from(monthSet).sort().reverse();

            if (sortedMonths.length === 0) {
                list.innerHTML = `
                    <div class="text-center py-8 text-slate-400">
                        <p class="text-xs">Belum ada riwayat arsip data bulanan.</p>
                    </div>
                `;
            } else {
                list.innerHTML = sortedMonths.map(ym => {
                    const txs = appData.transactions.filter(t => t.date && t.date.startsWith(ym));
                    let inc = 0, exp = 0;
                    txs.forEach(t => {
                        const a = parseFloat(t.amount) || 0;
                        if (t.type === 'INCOME') inc += a;
                        else exp += a;
                    });

                    const isArch = !!appData.archivedMonths[ym];
                    const isCurrent = ym === currentSelectedMonth;

                    return `
                        <div class="bg-slate-50 p-3 rounded-2xl border border-slate-200/80 flex items-center justify-between gap-2">
                            <div>
                                <div class="flex items-center gap-1.5">
                                    <h4 class="text-xs font-bold text-slate-800">${formatMonthLabel(ym)}</h4>
                                    ${isArch ? '<span class="text-[9px] bg-amber-100 text-amber-700 px-1.5 py-0.2 rounded font-bold">Terarsip</span>' : ''}
                                    ${isCurrent ? '<span class="text-[9px] bg-emerald-100 text-emerald-700 px-1.5 py-0.2 rounded font-bold">Aktif</span>' : ''}
                                </div>
                                <div class="text-[10px] text-slate-500 mt-1 space-x-2">
                                    <span>Masuk: <strong class="text-emerald-600">${formatIDR(inc)}</strong></span>
                                    <span>Keluar: <strong class="text-rose-600">${formatIDR(exp)}</strong></span>
                                </div>
                            </div>

                            <button onclick="selectMonthFromHistory('${ym}')" class="bg-white hover:bg-slate-100 border border-slate-200 text-slate-700 text-xs font-bold px-3 py-1.5 rounded-xl shadow-sm transition">
                                Lihat
                            </button>
                        </div>
                    `;
                }).join('');
            }

            modal.classList.remove('hidden');
        }

        function closeHistoryModal() {
            document.getElementById('historyModal').classList.add('hidden');
        }
        // Scroll To Top Helper
        function scrollToTop() {
            const container = document.getElementById('mainContainer');
            if (container) {
                container.scrollTo({ top: 0, behavior: 'smooth' });
            }
        }

        // Quick Filter by Clicking Summary Cards
        function filterSummaryType(type) {
            const selectEl = document.getElementById('filterCategory');
            if (!selectEl) return;

            if (type === 'INCOME') {
                selectEl.value = 'Pemasukan';
                showToast("Menampilkan Riwayat Pemasukan");
            } else if (type === 'EXPENSE') {
                selectEl.value = 'ALL';
                showToast("Menampilkan Riwayat Pengeluaran");
            }

            renderTransactions();

            // Scroll smooth down to transactions list section
            const filterEl = document.getElementById('filterCategory');
            if (filterEl) {
                filterEl.scrollIntoView({ behavior: 'smooth', block: 'start' });
            }
        }

        // Export Data Modal
        function openExportModal() {
            document.getElementById('exportMonthName').innerText = formatMonthLabel(currentSelectedMonth);
            document.getElementById('exportModal').classList.remove('hidden');
        }

        function closeExportModal() {
            document.getElementById('exportModal').classList.add('hidden');
        }

        function exportData(format) {
            const monthlyTxs = appData.transactions.filter(t => t.date && t.date.startsWith(currentSelectedMonth));

            if (format === 'csv') {
                if (monthlyTxs.length === 0) {
                    showToast("Tidak ada transaksi untuk diekspor");
                    return;
                }

                let csvContent = "data:text/csv;charset=utf-8,ID,Tanggal,Tipe,Kategori,Nominal,Metode,Catatan\n";
                monthlyTxs.forEach(t => {
                    const row = [
                        t.id,
                        t.date,
                        t.type,
                        `"${t.category || ''}"`,
                        t.amount,
                        `"${t.method || ''}"`,
                        `"${(t.note || '').replace(/"/g, '""')}"`
                    ].join(",");
                    csvContent += row + "\n";
                });

                const encodedUri = encodeURI(csvContent);
                const link = document.createElement("a");
                link.setAttribute("href", encodedUri);
                link.setAttribute("download", `Pengeluaran_${currentSelectedMonth}.csv`);
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
                showToast("Berhasil mengunduh CSV");

            } else if (format === 'json') {
                const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(appData, null, 2));
                const link = document.createElement("a");
                link.setAttribute("href", dataStr);
                link.setAttribute("download", `Backup_Dompet_RumahTangga_${currentSelectedMonth}.json`);
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
                showToast("Berhasil mengunduh JSON Backup");
            }

            closeExportModal();
        }

        // Toast Helper
        function showToast(message) {
            const toast = document.getElementById('toast');
            const toastMsg = document.getElementById('toastMsg');
            toastMsg.innerText = message;

            toast.classList.remove('opacity-0', 'pointer-events-none');
            toast.classList.add('opacity-100');

            setTimeout(() => {
                toast.classList.remove('opacity-100');
                toast.classList.add('opacity-0', 'pointer-events-none');
            }, 2500);
        }
    </script>
</body>
</html>
