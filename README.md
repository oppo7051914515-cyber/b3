<!DOCTYPE html>
<html lang="th" class="h-full bg-slate-50">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ระบบเช็คชื่อนักเรียนด้วย QR Code & Dynamic OTP (Real-Time)</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- QRCode.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <!-- SweetAlert2 -->
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <!-- Google Fonts: Prompt -->
    <link href="https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <style>
        body { font-family: 'Prompt', sans-serif; }
        .glass-card {
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(12px);
        }
        .animate-pulse-fast {
            animation: pulse 1s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }
    </style>
</head>
<body class="h-full flex flex-col text-slate-800 antialiased selection:bg-indigo-500 selection:text-white">

    <!-- Navigation Header -->
    <header id="main-header" class="bg-indigo-700 text-white shadow-lg sticky top-0 z-40">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
            <div class="flex items-center space-x-3">
                <div class="bg-white/20 p-2 rounded-xl backdrop-blur-md">
                    <i data-lucide="qr-code" class="w-6 h-6 text-white"></i>
                </div>
                <div>
                    <h1 class="font-bold text-lg leading-tight">Smart Check-In Live</h1>
                    <p class="text-xs text-indigo-200" id="header-subtitle">ระบบเช็คชื่อ QR Code + Real-Time Sync</p>
                </div>
            </div>

            <!-- Role Switcher (Hidden when student scans QR code) -->
            <div id="role-switcher-container" class="flex items-center space-x-2">
                <div class="bg-indigo-900/50 p-1 rounded-xl flex items-center border border-indigo-400/30">
                    <button id="nav-teacher-btn" onclick="switchRole('teacher')" class="px-3 py-1.5 rounded-lg text-xs sm:text-sm font-medium transition-all duration-200 flex items-center space-x-1.5 bg-white text-indigo-700 shadow-sm">
                        <i data-lucide="user-check" class="w-4 h-4"></i>
                        <span class="hidden sm:inline">มุมมองคุณครู</span>
                    </button>
                    <button id="nav-student-btn" onclick="switchRole('student')" class="px-3 py-1.5 rounded-lg text-xs sm:text-sm font-medium transition-all duration-200 flex items-center space-x-1.5 text-indigo-200 hover:text-white">
                        <i data-lucide="smartphone" class="w-4 h-4"></i>
                        <span class="hidden sm:inline">มุมมองนักเรียน</span>
                    </button>
                </div>
            </div>
        </div>
    </header>

    <!-- Main Content Container -->
    <main class="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6">
        
        <!-- ================= TEACHER VIEW ================= -->
        <div id="teacher-view" class="space-y-6">
            <!-- Navigation Tabs -->
            <div class="flex border-b border-slate-200 space-x-4 sm:space-x-8 overflow-x-auto pb-1">
                <button onclick="switchTeacherTab('session')" id="tab-btn-session" class="tab-btn font-semibold text-sm pb-3 text-indigo-600 border-b-2 border-indigo-600 flex items-center space-x-2 whitespace-nowrap">
                    <i data-lucide="play-circle" class="w-4 h-4"></i>
                    <span>เปิดคาบเช็คชื่อ (Live Session)</span>
                </button>
                <button onclick="switchTeacherTab('classes')" id="tab-btn-classes" class="tab-btn font-medium text-sm pb-3 text-slate-500 hover:text-slate-700 flex items-center space-x-2 whitespace-nowrap">
                    <i data-lucide="book-open" class="w-4 h-4"></i>
                    <span>จัดการห้องเรียน & รายวิชา</span>
                </button>
                <button onclick="switchTeacherTab('students')" id="tab-btn-students" class="tab-btn font-medium text-sm pb-3 text-slate-500 hover:text-slate-700 flex items-center space-x-2 whitespace-nowrap">
                    <i data-lucide="users" class="w-4 h-4"></i>
                    <span>จัดการรายชื่อนักเรียน</span>
                    <span id="badge-total-students" class="bg-indigo-100 text-indigo-600 text-xs px-2 py-0.5 rounded-full font-bold">0</span>
                </button>
                <button onclick="switchTeacherTab('reports')" id="tab-btn-reports" class="tab-btn font-medium text-sm pb-3 text-slate-500 hover:text-slate-700 flex items-center space-x-2 whitespace-nowrap">
                    <i data-lucide="file-spreadsheet" class="w-4 h-4"></i>
                    <span>รายงานประวัติ</span>
                </button>
            </div>

            <!-- Tab 1: Live Attendance Session -->
            <div id="teacher-tab-session" class="space-y-6">
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    <!-- Control Box -->
                    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex flex-col justify-between">
                        <div>
                            <h2 class="text-base font-bold text-slate-800 mb-4 flex items-center space-x-2">
                                <i data-lucide="sliders" class="w-5 h-5 text-indigo-600"></i>
                                <span>เลือกห้องเรียนเพื่อเปิดคาบ</span>
                            </h2>
                            
                            <form id="start-session-form" onsubmit="window.handleStartSession(event)" class="space-y-4">
                                <div>
                                    <label class="block text-xs font-semibold text-slate-600 mb-1">เลือกห้องเรียน / รายวิชา</label>
                                    <select id="session-class-select" required class="w-full text-sm rounded-xl border-slate-300 border p-3 focus:ring-2 focus:ring-indigo-500 focus:outline-none bg-slate-50">
                                        <!-- Dynamic Options -->
                                    </select>
                                </div>
                                <div>
                                    <label class="block text-xs font-semibold text-slate-600 mb-1">ระยะเวลาเปลี่ยน OTP อัตโนมัติ</label>
                                    <select id="session-duration" class="w-full text-sm rounded-xl border-slate-300 border p-2.5 focus:ring-2 focus:ring-indigo-500 focus:outline-none bg-slate-50">
                                        <option value="15">15 วินาที</option>
                                        <option value="30" selected>30 วินาที (แนะนำ)</option>
                                        <option value="60">60 วินาที</option>
                                    </select>
                                </div>
                                <button type="submit" id="start-btn" class="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-semibold py-3.5 px-4 rounded-xl shadow-md transition flex items-center justify-center space-x-2 text-base">
                                    <i data-lucide="play" class="w-5 h-5 fill-current"></i>
                                    <span>🟢 เริ่มเปิดคาบเรียน</span>
                                </button>
                            </form>
                        </div>

                        <div id="active-session-summary" class="mt-6 pt-4 border-t border-slate-100 hidden">
                            <div class="flex items-center justify-between text-xs text-slate-500 mb-3">
                                <span>สถานะคาบเรียน:</span>
                                <span class="text-emerald-600 font-bold flex items-center">
                                    <span class="w-2.5 h-2.5 rounded-full bg-emerald-500 animate-ping mr-1.5"></span>
                                    เปิดใช้งานออนไลน์
                                </span>
                            </div>
                            <button onclick="window.handleEndSession()" class="w-full bg-rose-600 hover:bg-rose-700 text-white font-semibold py-3 rounded-xl shadow transition flex items-center justify-center space-x-2 text-sm">
                                <i data-lucide="square" class="w-4 h-4 fill-current"></i>
                                <span>🔴 ปิดคาบเรียน</span>
                            </button>
                        </div>
                    </div>

                    <!-- Display Dynamic OTP & QR Code -->
                    <div class="lg:col-span-2 bg-gradient-to-br from-indigo-950 via-indigo-900 to-slate-900 p-6 rounded-2xl shadow-xl text-white flex flex-col md:flex-row items-center justify-between gap-6 relative overflow-hidden">
                        <div id="no-active-session" class="w-full py-12 text-center text-indigo-200 flex flex-col items-center">
                            <i data-lucide="qr-code" class="w-16 h-16 mb-3 opacity-30"></i>
                            <p class="text-sm font-medium">ยังไม่ได้เปิดคาบเรียน</p>
                            <p class="text-xs text-indigo-300/70 mt-1">เลือกห้องเรียนแล้วกด "🟢 เริ่มเปิดคาบเรียน" เพื่อแสดง QR Code</p>
                        </div>

                        <div id="has-active-session" class="w-full flex flex-col sm:flex-row items-center justify-between gap-6 hidden">
                            <!-- QR Code Holder -->
                            <div class="bg-white p-4 rounded-2xl shadow-xl flex flex-col items-center shrink-0">
                                <div id="qrcode" class="w-44 h-44 flex items-center justify-center"></div>
                                <p class="text-[11px] text-slate-500 mt-2 font-medium flex items-center">
                                    <i data-lucide="smartphone" class="w-3.5 h-3.5 mr-1 text-indigo-600"></i> สแกนด้วยมือถือเพื่อลงชื่อ
                                </p>
                            </div>

                            <!-- OTP Display -->
                            <div class="flex-1 text-center sm:text-left space-y-3">
                                <div class="inline-block bg-indigo-500/30 text-indigo-200 text-xs px-3 py-1 rounded-full border border-indigo-400/30 font-medium">
                                    <span id="display-session-name">ห้องเรียน - วิชา</span>
                                </div>
                                <h3 class="text-xs text-indigo-300 tracking-wider uppercase font-semibold"> Dynamic OTP Code</h3>
                                
                                <div class="flex items-center justify-center sm:justify-start space-x-3">
                                    <span id="otp-display" class="font-mono text-5xl font-black tracking-widest text-amber-400 drop-shadow-lg">------</span>
                                </div>

                                <div class="space-y-1.5 pt-1">
                                    <div class="flex justify-between text-xs text-indigo-200">
                                        <span>เปลี่ยนรหัสใหม่ใน:</span>
                                        <span id="timer-text" class="font-bold text-amber-300">30s</span>
                                    </div>
                                    <div class="w-full bg-slate-800/80 h-2.5 rounded-full overflow-hidden border border-slate-700">
                                        <div id="timer-bar" class="bg-amber-400 h-full w-full transition-all duration-1000"></div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Real-time Attendance Monitor Grid -->
                <div class="bg-white rounded-2xl shadow-sm border border-slate-200 p-5">
                    <div class="flex flex-col sm:flex-row sm:items-center justify-between pb-4 border-b border-slate-100 gap-3">
                        <div>
                            <h3 class="font-bold text-slate-800 flex items-center space-x-2">
                                <i data-lucide="activity" class="w-5 h-5 text-indigo-600"></i>
                                <span>ตารางเช็คชื่อแบบ Real-Time</span>
                            </h3>
                            <p class="text-xs text-slate-500">สถานะจะอัปเดตทันทีเมื่อนักเรียนสแกนและส่งรหัสผ่านมือถือ</p>
                        </div>
                        <div class="flex items-center space-x-4 text-xs font-medium">
                            <span class="flex items-center text-slate-500"><span class="w-3 h-3 rounded-full bg-slate-200 inline-block mr-1.5"></span> 🔴 ยังไม่เช็ค</span>
                            <span class="flex items-center text-amber-600"><span class="w-3 h-3 rounded-full bg-amber-400 inline-block mr-1.5"></span> 🟡 ได้รับ OTP</span>
                            <span class="flex items-center text-emerald-600"><span class="w-3 h-3 rounded-full bg-emerald-500 inline-block mr-1.5"></span> 🟢 เช็คชื่อแล้ว</span>
                        </div>
                    </div>

                    <div id="attendance-grid" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-3 mt-4">
                        <div class="col-span-full py-12 text-center text-slate-400 text-sm">
                            ยังไม่มีการเปิดคาบเรียนในขณะนี้
                        </div>
                    </div>
                </div>
            </div>

            <!-- Tab 2: Classroom Management -->
            <div id="teacher-tab-classes" class="space-y-6 hidden">
                <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                    <!-- Add Class Form -->
                    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 h-fit">
                        <h3 class="font-bold text-slate-800 mb-4 flex items-center space-x-2">
                            <i data-lucide="plus-circle" class="w-5 h-5 text-indigo-600"></i>
                            <span>เพิ่มห้องเรียน / รายวิชาใหม่</span>
                        </h3>
                        <form id="add-class-form" onsubmit="window.handleAddClass(event)" class="space-y-4">
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">ชื่อห้องเรียน / ชั้นปี</label>
                                <input type="text" id="input-class-name" placeholder="เช่น ม.4/1, ม.5/2" required class="w-full text-sm rounded-xl border-slate-300 border p-2.5 focus:ring-2 focus:ring-indigo-500 focus:outline-none">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">ชื่อวิชา / รหัสวิชา</label>
                                <input type="text" id="input-subject-name" placeholder="เช่น ว30101 วิทยาการคำนวณ" required class="w-full text-sm rounded-xl border-slate-300 border p-2.5 focus:ring-2 focus:ring-indigo-500 focus:outline-none">
                            </div>
                            <button type="submit" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-medium py-2.5 rounded-xl transition shadow-sm">
                                บันทึกสร้างห้องเรียน
                            </button>
                        </form>
                    </div>

                    <!-- Class List -->
                    <div class="md:col-span-2 bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
                        <h3 class="font-bold text-slate-800 mb-4">รายชื่อห้องเรียนทั้งหมด</h3>
                        <div id="classroom-list-container" class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                            <!-- Dynamic Class Cards -->
                        </div>
                    </div>
                </div>
            </div>

            <!-- Tab 3: Student Management -->
            <div id="teacher-tab-students" class="space-y-6 hidden">
                <div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-4 bg-indigo-50 p-4 rounded-2xl border border-indigo-100">
                    <div>
                        <h3 class="font-bold text-indigo-900">จัดการรายชื่อนักเรียน</h3>
                        <p class="text-xs text-indigo-700">เพิ่ม แก้ไข หรือนำเข้ารายชื่อนักเรียนแยกตามห้องเรียน</p>
                    </div>
                    <div class="flex flex-wrap gap-2">
                        <button onclick="window.openAddStudentModal()" class="bg-indigo-600 hover:bg-indigo-700 text-white text-xs font-medium px-3.5 py-2 rounded-xl transition flex items-center space-x-1 shadow-sm">
                            <i data-lucide="user-plus" class="w-4 h-4"></i>
                            <span>เพิ่มนักเรียน (รายคน)</span>
                        </button>
                        <button onclick="window.openBatchAddModal()" class="bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-medium px-3.5 py-2 rounded-xl transition flex items-center space-x-1 shadow-sm">
                            <i data-lucide="file-text" class="w-4 h-4"></i>
                            <span>คัดลอกวางรายชื่อ (Batch)</span>
                        </button>
                        <button onclick="window.addSampleStudents()" class="bg-amber-500 hover:bg-amber-600 text-white text-xs font-medium px-3.5 py-2 rounded-xl transition flex items-center space-x-1 shadow-sm">
                            <i data-lucide="sparkles" class="w-4 h-4"></i>
                            <span>เพิ่มรายชื่อตัวอย่าง</span>
                        </button>
                    </div>
                </div>

                <!-- Filter & Search -->
                <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-200 flex flex-col sm:flex-row justify-between gap-4">
                    <div class="w-full sm:w-64">
                        <label class="block text-xs font-semibold text-slate-500 mb-1">เลือกกรองตามห้องเรียน</label>
                        <select id="student-filter-class" onchange="window.renderStudentList()" class="w-full text-sm rounded-xl border-slate-300 border p-2 focus:ring-2 focus:ring-indigo-500 focus:outline-none">
                            <option value="ALL">แสดงนักเรียนทุกห้อง</option>
                        </select>
                    </div>
                    <div class="w-full sm:w-64">
                        <label class="block text-xs font-semibold text-slate-500 mb-1">ค้นหา (รหัส หรือ ชื่อ)</label>
                        <input type="text" id="student-search-input" onkeyup="window.renderStudentList()" placeholder="พิมพ์ชื่อเพื่อค้นหา..." class="w-full text-sm rounded-xl border-slate-300 border p-2 focus:ring-2 focus:ring-indigo-500 focus:outline-none">
                    </div>
                </div>

                <!-- Students Table -->
                <div class="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
                    <div class="overflow-x-auto">
                        <table class="w-full text-left text-sm text-slate-600">
                            <thead class="bg-slate-50 text-slate-700 font-semibold border-b border-slate-200">
                                <tr>
                                    <th class="p-3.5 pl-5">รหัสประจำตัว</th>
                                    <th class="p-3.5">ชื่อ - นามสกุล</th>
                                    <th class="p-3.5">ห้องเรียน</th>
                                    <th class="p-3.5 text-right pr-5">จัดการ</th>
                                </tr>
                            </thead>
                            <tbody id="student-table-body" class="divide-y divide-slate-100">
                                <!-- Dynamic Student Rows -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>

            <!-- Tab 4: History Reports -->
            <div id="teacher-tab-reports" class="space-y-6 hidden">
                <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
                    <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-6 gap-4">
                        <div>
                            <h3 class="font-bold text-slate-800">ประวัติการเช็คชื่อย้อนหลัง</h3>
                            <p class="text-xs text-slate-500">บันทึกข้อมูลการเช็คชื่อในระบบเรียลไทม์</p>
                        </div>
                        <button onclick="window.exportToCSV()" class="bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-medium px-4 py-2.5 rounded-xl transition flex items-center space-x-2 shadow-sm">
                            <i data-lucide="download" class="w-4 h-4"></i>
                            <span>ส่งออกข้อมูล CSV / Excel</span>
                        </button>
                    </div>

                    <div class="overflow-x-auto">
                        <table class="w-full text-left text-sm text-slate-600">
                            <thead class="bg-slate-50 text-slate-700 font-semibold border-b border-slate-200">
                                <tr>
                                    <th class="p-3 pl-4">วันที่ / เวลา</th>
                                    <th class="p-3">ห้องเรียน / วิชา</th>
                                    <th class="p-3">จำนวนเข้าเรียน</th>
                                    <th class="p-3">สถานะ</th>
                                </tr>
                            </thead>
                            <tbody id="history-table-body" class="divide-y divide-slate-100">
                                <!-- Dynamic History Rows -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <!-- ================= STUDENT VIEW (LOCKED WHEN SCANNED) ================= -->
        <div id="student-view" class="max-w-md mx-auto space-y-6 hidden">
            <!-- Active Session Student Card -->
            <div id="student-session-card" class="bg-white p-6 rounded-3xl shadow-xl border border-slate-100 text-center space-y-5">
                <div class="bg-indigo-50 w-16 h-16 rounded-2xl flex items-center justify-center mx-auto text-indigo-600">
                    <i data-lucide="user-check" class="w-8 h-8"></i>
                </div>

                <div>
                    <h2 class="text-xl font-bold text-slate-800" id="student-class-title">กำลังโหลดข้อมูลคาบเรียน...</h2>
                    <p class="text-xs text-slate-500 mt-1" id="student-subject-sub">กรุณารอสักครู่</p>
                </div>

                <!-- Form to Submit OTP -->
                <form id="student-checkin-form" onsubmit="window.handleStudentSubmitOTP(event)" class="space-y-4 text-left pt-2">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">เลือกรายชื่อของคุณ</label>
                        <select id="student-select-name" required class="w-full text-sm font-medium rounded-xl border-slate-300 border p-3 focus:ring-2 focus:ring-indigo-500 focus:outline-none bg-slate-50">
                            <option value="">-- กำลังโหลดรายชื่อ --</option>
                        </select>
                    </div>

                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">รหัส Dynamic OTP (6 หลักบนหน้าจอครู)</label>
                        <input type="text" id="student-input-otp" maxlength="6" placeholder="000000" required class="w-full text-center text-3xl font-mono tracking-widest font-bold text-indigo-600 rounded-xl border-slate-300 border p-3 focus:ring-2 focus:ring-indigo-500 focus:outline-none bg-slate-50">
                    </div>

                    <button type="submit" id="student-submit-btn" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-semibold py-3.5 rounded-xl shadow-lg transition flex items-center justify-center space-x-2 text-base">
                        <i data-lucide="send" class="w-5 h-5"></i>
                        <span>ส่งรหัสลงชื่อเข้าเรียน</span>
                    </button>
                </form>

                <!-- Feedback Message -->
                <div id="student-result-message" class="hidden p-4 rounded-xl text-sm font-medium"></div>
            </div>

            <!-- No Session Active State for Student -->
            <div id="student-no-session" class="bg-white p-8 rounded-3xl shadow-lg border border-slate-100 text-center space-y-4 hidden">
                <div class="bg-rose-50 w-16 h-16 rounded-2xl flex items-center justify-center mx-auto text-rose-500">
                    <i data-lucide="alert-circle" class="w-8 h-8"></i>
                </div>
                <h3 class="text-lg font-bold text-slate-800">คาบเรียนนี้ยังไม่เปิด หรือสิ้นสุดลงแล้ว</h3>
                <p class="text-xs text-slate-500">หากคุณครูยังไม่เริ่มเปิดคาบเรียน กรุณารอคุณครูกดเปิดคาบแล้วสแกน QR Code อีกครั้ง</p>
            </div>
        </div>
    </main>

    <!-- Modal: Add Single Student -->
    <div id="modal-add-student" class="fixed inset-0 bg-slate-900/40 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-white w-full max-w-md rounded-2xl shadow-2xl border border-slate-100 overflow-hidden">
            <div class="p-5 bg-indigo-600 text-white flex justify-between items-center">
                <h3 class="font-bold">เพิ่มนักเรียนเข้าห้องเรียน</h3>
                <button onclick="window.closeModal('modal-add-student')" class="text-indigo-200 hover:text-white"><i data-lucide="x" class="w-5 h-5"></i></button>
            </div>
            <form onsubmit="window.handleSaveSingleStudent(event)" class="p-5 space-y-4">
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">เลือกห้องเรียน</label>
                    <select id="single-student-class" required class="w-full text-sm rounded-xl border-slate-300 border p-2.5 focus:ring-2 focus:ring-indigo-500 focus:outline-none">
                    </select>
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">รหัสประจำตัวนักเรียน</label>
                    <input type="text" id="single-student-id" placeholder="เช่น 1001" required class="w-full text-sm rounded-xl border-slate-300 border p-2.5 focus:ring-2 focus:ring-indigo-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">ชื่อ - นามสกุล</label>
                    <input type="text" id="single-student-name" placeholder="เช่น นายสมชาย ใจดี" required class="w-full text-sm rounded-xl border-slate-300 border p-2.5 focus:ring-2 focus:ring-indigo-500 focus:outline-none">
                </div>
                <div class="flex justify-end space-x-2 pt-2">
                    <button type="button" onclick="window.closeModal('modal-add-student')" class="px-4 py-2 text-xs font-medium text-slate-600 hover:bg-slate-100 rounded-xl">ยกเลิก</button>
                    <button type="submit" class="px-4 py-2 text-xs font-medium bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl shadow-sm">บันทึกข้อมูล</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Modal: Batch Add Students -->
    <div id="modal-batch-add" class="fixed inset-0 bg-slate-900/40 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-white w-full max-w-lg rounded-2xl shadow-2xl border border-slate-100 overflow-hidden">
            <div class="p-5 bg-emerald-600 text-white flex justify-between items-center">
                <div>
                    <h3 class="font-bold">เพิ่มรายชื่อนักเรียนแบบกลุ่ม (Batch)</h3>
                    <p class="text-xs text-emerald-100">คัดลอกรายชื่อจาก Excel มาวางได้พร้อมกันหลายคน</p>
                </div>
                <button onclick="window.closeModal('modal-batch-add')" class="text-emerald-200 hover:text-white"><i data-lucide="x" class="w-5 h-5"></i></button>
            </div>
            <form onsubmit="window.handleSaveBatchStudents(event)" class="p-5 space-y-4">
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">เลือกห้องเรียนที่ต้องการนำเข้า</label>
                    <select id="batch-student-class" required class="w-full text-sm rounded-xl border-slate-300 border p-2.5 focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                    </select>
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">วางรายชื่อ (รูปแบบ: รหัสประจำตัว [เว้นวรรค/Tab] ชื่อ-นามสกุล)</label>
                    <textarea id="batch-text-input" rows="6" placeholder="1001 นายสมชาย ใจดี&#10;1002 นางสาวสมหญิง รักเรียน&#10;1003 นายกิตติศักดิ์ มีสุข" required class="w-full font-mono text-xs rounded-xl border-slate-300 border p-3 focus:ring-2 focus:ring-emerald-500 focus:outline-none"></textarea>
                </div>
                <div class="flex justify-end space-x-2 pt-2">
                    <button type="button" onclick="window.closeModal('modal-batch-add')" class="px-4 py-2 text-xs font-medium text-slate-600 hover:bg-slate-100 rounded-xl">ยกเลิก</button>
                    <button type="submit" class="px-4 py-2 text-xs font-medium bg-emerald-600 hover:bg-emerald-700 text-white rounded-xl shadow-sm">นำเข้ารายชื่อทั้งหมด</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Firebase Real-Time SDK and Application Logic -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, updateDoc, deleteDoc, onSnapshot, collection, query, addDoc, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global Firebase Config fallback
        const firebaseConfig = typeof __firebase_config !== 'undefined'
            ? JSON.parse(__firebase_config)
            : {
                apiKey: "demo-api-key",
                authDomain: "demo-app.firebaseapp.com",
                projectId: "demo-app",
                storageBucket: "demo-app.appspot.com",
                messagingSenderId: "123456789",
                appId: "1:123456789:web:abcdef"
            };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        const appId = typeof __app_id !== 'undefined' ? __app_id : 'qr-checkin-live-app';

        // Public Collections helper paths
        const getPublicColl = (colName) => collection(db, 'artifacts', appId, 'public', 'data', colName);
        const getPublicDoc = (colName, docId) => doc(db, 'artifacts', appId, 'public', 'data', colName, docId);

        // App States
        let currentUser = null;
        let classrooms = [];
        let students = [];
        let activeSession = null;
        let attendanceRecords = [];
        let sessionTimerInterval = null;
        let isLockedStudentMode = false;
        let currentSessionIdParam = null;

        // App Initialization
        window.addEventListener('DOMContentLoaded', async () => {
            // Check URL Parameters
            const urlParams = new URLSearchParams(window.location.search);
            currentSessionIdParam = urlParams.get('session');

            if (currentSessionIdParam) {
                isLockedStudentMode = true;
                lockToStudentView();
            }

            // Firebase Auth Setup
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (err) {
                console.warn("Auth initialization issue, continuing anonymously:", err);
            }

            onAuthStateChanged(auth, async (user) => {
                if (!user) return;
                currentUser = user;

                // Load Realtime Data
                initRealtimeListeners();
            });

            lucide.createIcons();
        });

        function lockToStudentView() {
            document.getElementById('role-switcher-container').classList.add('hidden');
            document.getElementById('header-subtitle').innerText = "ลงชื่อเข้าเรียน (สำหรับนักเรียน)";
            document.getElementById('teacher-view').classList.add('hidden');
            document.getElementById('student-view').classList.remove('hidden');
        }

        function initRealtimeListeners() {
            if (!currentUser) return;

            // Listen to Classrooms
            onSnapshot(getPublicColl('classrooms'), (snapshot) => {
                classrooms = snapshot.docs.map(d => ({ id: d.id, ...d.data() }));
                if (classrooms.length === 0 && !isLockedStudentMode) {
                    seedDefaultData();
                } else {
                    updateAllDropdowns();
                    renderClassroomList();
                }
            }, (err) => console.error("Classrooms listener error:", err));

            // Listen to Students
            onSnapshot(getPublicColl('students'), (snapshot) => {
                students = snapshot.docs.map(d => ({ id: d.id, ...d.data() }));
                updateAllDropdowns();
                renderStudentList();
            }, (err) => console.error("Students listener error:", err));

            // Listen to Active Session
            onSnapshot(getPublicColl('active_sessions'), (snapshot) => {
                const sessions = snapshot.docs.map(d => ({ id: d.id, ...d.data() }));
                
                if (isLockedStudentMode && currentSessionIdParam) {
                    activeSession = sessions.find(s => s.id === currentSessionIdParam && s.status === 'ACTIVE') || null;
                    renderStudentSessionView();
                } else {
                    activeSession = sessions.find(s => s.status === 'ACTIVE') || null;
                    renderTeacherSessionUI();
                }
            }, (err) => console.error("Active Session listener error:", err));

            // Listen to Attendances
            onSnapshot(getPublicColl('attendances'), (snapshot) => {
                attendanceRecords = snapshot.docs.map(d => ({ id: d.id, ...d.data() }));
                renderAttendanceGrid();
                renderHistoryList();
            }, (err) => console.error("Attendances listener error:", err));
        }

        async function seedDefaultData() {
            if (!currentUser) return;
            const class1Ref = doc(getPublicColl('classrooms'), 'c1');
            const class2Ref = doc(getPublicColl('classrooms'), 'c2');

            await setDoc(class1Ref, { name: 'ม.4/1', subject: 'ว30101 วิทยาการคำนวณ', createdAt: Date.now() });
            await setDoc(class2Ref, { name: 'ม.5/2', subject: 'ค31102 คณิตศาสตร์เพิ่มเติม', createdAt: Date.now() });

            const sampleStudents = [
                { id: 'st1', studentId: '1001', name: 'นายกิตติพงษ์ ใจงาม', classId: 'c1' },
                { id: 'st2', studentId: '1002', name: 'นางสาวชลธิชา สุขสันต์', classId: 'c1' },
                { id: 'st3', studentId: '1003', name: 'นายธนกร วงศ์สว่าง', classId: 'c1' },
                { id: 'st4', studentId: '2001', name: 'นางสาวปรียาพร ดีเลิศ', classId: 'c2' }
            ];

            for (const s of sampleStudents) {
                await setDoc(doc(getPublicColl('students'), s.id), s);
            }
        }

        window.switchRole = function(role) {
            if (isLockedStudentMode) return; // Prevent switching when scanned
            const teacherView = document.getElementById('teacher-view');
            const studentView = document.getElementById('student-view');
            const teacherBtn = document.getElementById('nav-teacher-btn');
            const studentBtn = document.getElementById('nav-student-btn');

            if (role === 'teacher') {
                teacherView.classList.remove('hidden');
                studentView.classList.add('hidden');
                teacherBtn.className = "px-3 py-1.5 rounded-lg text-xs sm:text-sm font-medium transition-all duration-200 flex items-center space-x-1.5 bg-white text-indigo-700 shadow-sm";
                studentBtn.className = "px-3 py-1.5 rounded-lg text-xs sm:text-sm font-medium transition-all duration-200 flex items-center space-x-1.5 text-indigo-200 hover:text-white";
            } else {
                teacherView.classList.add('hidden');
                studentView.classList.remove('hidden');
                studentBtn.className = "px-3 py-1.5 rounded-lg text-xs sm:text-sm font-medium transition-all duration-200 flex items-center space-x-1.5 bg-white text-indigo-700 shadow-sm";
                teacherBtn.className = "px-3 py-1.5 rounded-lg text-xs sm:text-sm font-medium transition-all duration-200 flex items-center space-x-1.5 text-indigo-200 hover:text-white";
                renderStudentSessionView();
            }
        };

        window.switchTeacherTab = function(tabName) {
            ['session', 'classes', 'students', 'reports'].forEach(tab => {
                document.getElementById(`teacher-tab-${tab}`).classList.add('hidden');
                const btn = document.getElementById(`tab-btn-${tab}`);
                if (btn) btn.className = "tab-btn font-medium text-sm pb-3 text-slate-500 hover:text-slate-700 flex items-center space-x-2 whitespace-nowrap";
            });

            document.getElementById(`teacher-tab-${tabName}`).classList.remove('hidden');
            const activeBtn = document.getElementById(`tab-btn-${tabName}`);
            if (activeBtn) activeBtn.className = "tab-btn font-semibold text-sm pb-3 text-indigo-600 border-b-2 border-indigo-600 flex items-center space-x-2 whitespace-nowrap";
        };

        function updateAllDropdowns() {
            const sessionSelect = document.getElementById('session-class-select');
            const filterSelect = document.getElementById('student-filter-class');
            const singleSelect = document.getElementById('single-student-class');
            const batchSelect = document.getElementById('batch-student-class');

            let optionsHtml = classrooms.map(c => `<option value="${c.id}">${c.name} - ${c.subject}</option>`).join('');

            if (classrooms.length === 0) {
                optionsHtml = `<option value="">-- กรุณาสร้างห้องเรียนก่อน --</option>`;
            }

            if (sessionSelect) sessionSelect.innerHTML = optionsHtml;
            if (singleSelect) singleSelect.innerHTML = optionsHtml;
            if (batchSelect) batchSelect.innerHTML = optionsHtml;

            if (filterSelect) {
                filterSelect.innerHTML = `<option value="ALL">แสดงนักเรียนทุกห้อง</option>` + classrooms.map(c => 
                    `<option value="${c.id}">${c.name} - ${c.subject}</option>`
                ).join('');
            }

            const totalBadge = document.getElementById('badge-total-students');
            if (totalBadge) totalBadge.innerText = students.length;
        }

        window.handleAddClass = async function(e) {
            e.preventDefault();
            if (!currentUser) return;

            const name = document.getElementById('input-class-name').value.trim();
            const subject = document.getElementById('input-subject-name').value.trim();

            if (!name || !subject) return;

            const newDocRef = doc(getPublicColl('classrooms'), 'c_' + Date.now());
            await setDoc(newDocRef, { name, subject, createdAt: Date.now() });

            document.getElementById('add-class-form').reset();
            Swal.fire({ icon: 'success', title: 'สร้างห้องเรียนสำเร็จ!', text: `เพิ่ม ${name} เรียบร้อยแล้ว`, timer: 1500, showConfirmButton: false });
        };

        function renderClassroomList() {
            const container = document.getElementById('classroom-list-container');
            if (!container) return;

            if (classrooms.length === 0) {
                container.innerHTML = `<p class="text-sm text-slate-400 col-span-full">ยังไม่มีห้องเรียนในระบบ</p>`;
                return;
            }

            container.innerHTML = classrooms.map(c => {
                const count = students.filter(s => s.classId === c.id).length;
                return `
                    <div class="p-4 rounded-xl border border-slate-200 bg-slate-50 flex justify-between items-center">
                        <div>
                            <h4 class="font-bold text-slate-800">${c.name}</h4>
                            <p class="text-xs text-slate-500">${c.subject}</p>
                            <span class="inline-block mt-2 text-xs bg-indigo-100 text-indigo-700 px-2 py-0.5 rounded-md font-medium">นักเรียน ${count} คน</span>
                        </div>
                        <button onclick="window.deleteClassroom('${c.id}')" class="text-slate-400 hover:text-rose-600 p-2"><i data-lucide="trash-2" class="w-4 h-4"></i></button>
                    </div>
                `;
            }).join('');
            lucide.createIcons();
        }

        window.deleteClassroom = async function(classId) {
            const res = await Swal.fire({
                title: 'ยืนยันลบห้องเรียน?',
                text: "รายชื่อนักเรียนในห้องเรียนนี้จะถูกลบไปด้วย",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#e11d48',
                confirmButtonText: 'ลบห้องเรียน',
                cancelButtonText: 'ยกเลิก'
            });

            if (res.isConfirmed && currentUser) {
                await deleteDoc(getPublicDoc('classrooms', classId));
                // Delete associated students
                const classStudents = students.filter(s => s.classId === classId);
                for (const st of classStudents) {
                    await deleteDoc(getPublicDoc('students', st.id));
                }
                Swal.fire({ icon: 'success', title: 'ลบห้องเรียนเรียบร้อย', timer: 1200, showConfirmButton: false });
            }
        };

        window.openAddStudentModal = function() {
            if (classrooms.length === 0) {
                Swal.fire('แจ้งเตือน', 'กรุณาสร้างห้องเรียนก่อนเพิ่มนักเรียน', 'info');
                return;
            }
            document.getElementById('modal-add-student').classList.remove('hidden');
        };

        window.openBatchAddModal = function() {
            if (classrooms.length === 0) {
                Swal.fire('แจ้งเตือน', 'กรุณาสร้างห้องเรียนก่อนเพิ่มนักเรียน', 'info');
                return;
            }
            document.getElementById('modal-batch-add').classList.remove('hidden');
        };

        window.closeModal = function(id) {
            document.getElementById(id).classList.add('hidden');
        };

        window.handleSaveSingleStudent = async function(e) {
            e.preventDefault();
            if (!currentUser) return;

            const classId = document.getElementById('single-student-class').value;
            const studentId = document.getElementById('single-student-id').value.trim();
            const name = document.getElementById('single-student-name').value.trim();

            if (students.some(s => s.studentId === studentId && s.classId === classId)) {
                Swal.fire('รหัสซ้ำ', 'รหัสนักเรียนนี้นำเข้าไว้ในห้องนี้แล้ว', 'warning');
                return;
            }

            const docId = 'st_' + Date.now();
            await setDoc(getPublicDoc('students', docId), { id: docId, studentId, name, classId });
            window.closeModal('modal-add-student');
            document.getElementById('single-student-id').value = '';
            document.getElementById('single-student-name').value = '';
            Swal.fire({ icon: 'success', title: 'เพิ่มนักเรียนสำเร็จ!', timer: 1200, showConfirmButton: false });
        };

        window.handleSaveBatchStudents = async function(e) {
            e.preventDefault();
            if (!currentUser) return;

            const classId = document.getElementById('batch-student-class').value;
            const rawText = document.getElementById('batch-text-input').value.trim();
            if (!rawText) return;

            const lines = rawText.split('\n');
            let addedCount = 0;

            for (let i = 0; i < lines.length; i++) {
                const line = lines[i].trim();
                if (!line) continue;

                const parts = line.split(/[\t\s]+/);
                let stId, stName;

                if (parts.length >= 2) {
                    stId = parts[0];
                    stName = parts.slice(1).join(' ');
                } else {
                    stId = (1000 + students.length + i + 1).toString();
                    stName = line;
                }

                const docId = 'st_' + Date.now() + '_' + i;
                await setDoc(getPublicDoc('students', docId), { id: docId, studentId: stId, name: stName, classId });
                addedCount++;
            }

            window.closeModal('modal-batch-add');
            document.getElementById('batch-text-input').value = '';
            Swal.fire('สำเร็จ', `นำเข้ารายชื่อนักเรียนจำนวน ${addedCount} คนเรียบร้อยแล้ว`, 'success');
        };

        window.addSampleStudents = async function() {
            if (classrooms.length === 0) return;
            const targetClassId = classrooms[0].id;
            const samples = [
                { name: 'นายธนาธิป มีสุข' },
                { name: 'นางสาววิภาดา เจริญยิ่ง' },
                { name: 'นายณัฐวุฒิ สุขสวัสดิ์' }
            ];

            for (let i = 0; i < samples.length; i++) {
                const stId = (2000 + students.length + i + 1).toString();
                const docId = 'st_sample_' + Date.now() + '_' + i;
                await setDoc(getPublicDoc('students', docId), { id: docId, studentId: stId, name: samples[i].name, classId: targetClassId });
            }
            Swal.fire({ icon: 'success', title: 'เพิ่มนักเรียนตัวอย่างเรียบร้อย', timer: 1200, showConfirmButton: false });
        };

        window.renderStudentList = function() {
            const filterSelect = document.getElementById('student-filter-class');
            const searchInput = document.getElementById('student-search-input');
            const tbody = document.getElementById('student-table-body');

            if (!tbody || !filterSelect) return;

            const filterClass = filterSelect.value;
            const searchText = searchInput ? searchInput.value.toLowerCase() : '';

            let filtered = students;
            if (filterClass !== 'ALL') {
                filtered = filtered.filter(s => s.classId === filterClass);
            }

            if (searchText) {
                filtered = filtered.filter(s => (s.studentId && s.studentId.toLowerCase().includes(searchText)) || s.name.toLowerCase().includes(searchText));
            }

            if (filtered.length === 0) {
                tbody.innerHTML = `<tr><td colspan="4" class="p-6 text-center text-slate-400">ไม่พบรายชื่อนักเรียน</td></tr>`;
                return;
            }

            tbody.innerHTML = filtered.map(s => {
                const classroom = classrooms.find(c => c.id === s.classId);
                const className = classroom ? classroom.name : 'ไม่ระบุ';
                return `
                    <tr class="hover:bg-slate-50 transition">
                        <td class="p-3.5 pl-5 font-mono text-xs font-semibold text-slate-700">${s.studentId || '-'}</td>
                        <td class="p-3.5 font-medium text-slate-800">${s.name}</td>
                        <td class="p-3.5"><span class="bg-slate-100 text-slate-600 text-xs px-2.5 py-1 rounded-lg font-medium">${className}</span></td>
                        <td class="p-3.5 text-right pr-5">
                            <button onclick="window.deleteStudent('${s.id}')" class="text-slate-400 hover:text-rose-600 p-1"><i data-lucide="trash-2" class="w-4 h-4"></i></button>
                        </td>
                    </tr>
                `;
            }).join('');
            lucide.createIcons();
        };

        window.deleteStudent = async function(docId) {
            if (confirm("ต้องการลบรายชื่อนักเรียนคนนี้หรือไม่?") && currentUser) {
                await deleteDoc(getPublicDoc('students', docId));
            }
        };

        function generateOTP() {
            return Math.floor(100000 + Math.random() * 900000).toString();
        }

        window.handleStartSession = async function(e) {
            e.preventDefault();
            if (!currentUser) return;

            const classId = document.getElementById('session-class-select').value;
            const duration = parseInt(document.getElementById('session-duration').value);

            if (!classId) {
                Swal.fire('ข้อผิดพลาด', 'กรุณาสร้างและเลือกห้องเรียนก่อนเริ่มเปิดคาบ', 'warning');
                return;
            }

            const classroom = classrooms.find(c => c.id === classId);
            const classStudents = students.filter(s => s.classId === classId);

            if (classStudents.length === 0) {
                Swal.fire('ไม่พบนักเรียน', 'ห้องเรียนนี้ยังไม่มีนักเรียน กรุณาเพิ่มนักเรียนในเมนูจัดการรายชื่อก่อนเปิดคาบ', 'info');
                return;
            }

            const sessionId = 'session_' + Date.now();
            const sessionData = {
                id: sessionId,
                classId: classId,
                className: classroom.name,
                subjectName: classroom.subject,
                currentOTP: generateOTP(),
                duration: duration,
                status: 'ACTIVE',
                createdAt: Date.now()
            };

            await setDoc(getPublicDoc('active_sessions', sessionId), sessionData);
            Swal.fire({ icon: 'success', title: '🟢 เปิดคาบเรียนสำเร็จ!', text: 'ระบบเริ่มสร้าง QR Code และ OTP แบบ Real-Time แล้ว', timer: 1500, showConfirmButton: false });
        };

        function startOTPTimer(duration) {
            if (sessionTimerInterval) clearInterval(sessionTimerInterval);

            let timeLeft = duration;
            sessionTimerInterval = setInterval(async () => {
                if (!activeSession) {
                    clearInterval(sessionTimerInterval);
                    return;
                }

                timeLeft--;
                const timerText = document.getElementById('timer-text');
                const timerBar = document.getElementById('timer-bar');

                if (timerText) timerText.innerText = `${timeLeft}s`;
                if (timerBar) timerBar.style.width = `${(timeLeft / duration) * 100}%`;

                if (timeLeft <= 0) {
                    timeLeft = duration;
                    const newOTP = generateOTP();
                    if (currentUser && activeSession) {
                        await updateDoc(getPublicDoc('active_sessions', activeSession.id), { currentOTP: newOTP });
                    }
                }
            }, 1000);
        }

        function renderTeacherSessionUI() {
            const noSession = document.getElementById('no-active-session');
            const hasSession = document.getElementById('has-active-session');
            const summary = document.getElementById('active-session-summary');

            if (!activeSession) {
                if (noSession) noSession.classList.remove('hidden');
                if (hasSession) hasSession.classList.add('hidden');
                if (summary) summary.classList.add('hidden');
                if (sessionTimerInterval) clearInterval(sessionTimerInterval);
                return;
            }

            if (noSession) noSession.classList.add('hidden');
            if (hasSession) hasSession.classList.remove('hidden');
            if (summary) summary.classList.remove('hidden');

            document.getElementById('display-session-name').innerText = `${activeSession.className} - ${activeSession.subjectName}`;
            document.getElementById('otp-display').innerText = activeSession.currentOTP;

            // Generate Student Direct Scanner URL
            const studentCheckinUrl = `${window.location.origin}${window.location.pathname}?session=${activeSession.id}`;

            const qrContainer = document.getElementById('qrcode');
            if (qrContainer) {
                qrContainer.innerHTML = '';
                new QRCode(qrContainer, {
                    text: studentCheckinUrl,
                    width: 160,
                    height: 160,
                    colorDark : "#1e1b4b",
                    colorLight : "#ffffff",
                    correctLevel : QRCode.CorrectLevel.H
                });
            }

            startOTPTimer(activeSession.duration || 30);
            renderAttendanceGrid();
        }

        window.handleEndSession = async function() {
            if (!activeSession) return;
            const res = await Swal.fire({
                title: 'ยืนยันปิดคาบเรียน?',
                text: "ระบบจะบันทึกผลการเช็คชื่อเข้าสู่ประวัติย้อนหลัง",
                icon: 'question',
                showCancelButton: true,
                confirmButtonColor: '#e11d48',
                confirmButtonText: '🔴 ปิดคาบเรียน',
                cancelButtonText: 'ยกเลิก'
            });

            if (res.isConfirmed && currentUser) {
                await updateDoc(getPublicDoc('active_sessions', activeSession.id), { status: 'ENDED', endedAt: Date.now() });
                if (sessionTimerInterval) clearInterval(sessionTimerInterval);
                Swal.fire('ปิดคาบเรียบร้อย', 'บันทึกประวัติการเช็คชื่อเข้าสู่ระบบแล้ว', 'success');
            }
        };

        function renderAttendanceGrid() {
            const grid = document.getElementById('attendance-grid');
            if (!grid) return;

            if (!activeSession) {
                grid.innerHTML = `<div class="col-span-full py-12 text-center text-slate-400 text-sm">ยังไม่มีการเปิดคาบเรียนในขณะนี้</div>`;
                return;
            }

            const classStudents = students.filter(s => s.classId === activeSession.classId);

            if (classStudents.length === 0) {
                grid.innerHTML = `<div class="col-span-full py-8 text-center text-slate-400 text-sm">ไม่มีนักเรียนในห้องเรียนนี้</div>`;
                return;
            }

            grid.innerHTML = classStudents.map(student => {
                const record = attendanceRecords.find(r => r.sessionId === activeSession.id && r.studentId === student.id);

                let badgeStyle = "bg-slate-50 border-slate-200 text-slate-500";
                let statusBadge = "🔴 ยังไม่เช็ค";
                let actionBtn = "";

                if (record) {
                    if (record.status === 'PENDING') {
                        badgeStyle = "bg-amber-50 border-amber-300 text-amber-800 animate-pulse-fast";
                        statusBadge = `🟡 ได้รับ OTP: <span class="font-mono font-bold">${record.otpSubmitted}</span>`;
                        actionBtn = `<button onclick="window.verifyStudentAttendance('${record.id}')" class="mt-2 w-full bg-emerald-600 text-white text-xs py-1.5 rounded-lg font-medium shadow hover:bg-emerald-700">กดยืนยัน 🟢</button>`;
                    } else if (record.status === 'VERIFIED') {
                        badgeStyle = "bg-emerald-50 border-emerald-300 text-emerald-800 font-bold";
                        statusBadge = `🟢 เช็คชื่อสำเร็จ (${record.checkInTime})`;
                    }
                }

                return `
                    <div class="p-3.5 rounded-xl border ${badgeStyle} flex flex-col justify-between transition-all duration-300">
                        <div>
                            <div class="flex justify-between items-center mb-1">
                                <span class="text-xs font-mono font-semibold opacity-70">${student.studentId || '-'}</span>
                                <span class="text-[11px]">${statusBadge}</span>
                            </div>
                            <p class="text-sm font-semibold truncate">${student.name}</p>
                        </div>
                        ${actionBtn}
                    </div>
                `;
            }).join('');
        }

        window.verifyStudentAttendance = async function(recordId) {
            if (!currentUser) return;
            const timeStr = new Date().toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' });
            await updateDoc(getPublicDoc('attendances', recordId), {
                status: 'VERIFIED',
                checkInTime: timeStr
            });
            Swal.fire({ icon: 'success', title: 'ยืนยันสำเร็จ!', timer: 1000, showConfirmButton: false });
        };

        function renderStudentSessionView() {
            const card = document.getElementById('student-session-card');
            const noSessionCard = document.getElementById('student-no-session');
            const selectName = document.getElementById('student-select-name');

            if (!activeSession) {
                if (card) card.classList.add('hidden');
                if (noSessionCard) noSessionCard.classList.remove('hidden');
                return;
            }

            if (card) card.classList.remove('hidden');
            if (noSessionCard) noSessionCard.classList.add('hidden');

            document.getElementById('student-class-title').innerText = activeSession.className;
            document.getElementById('student-subject-sub').innerText = activeSession.subjectName;

            const classStudents = students.filter(s => s.classId === activeSession.classId);
            selectName.innerHTML = `<option value="">-- เลือกรายชื่อของคุณ --</option>` + classStudents.map(s => 
                `<option value="${s.id}">${s.studentId ? s.studentId + ' - ' : ''}${s.name}</option>`
            ).join('');
        }

        window.handleStudentSubmitOTP = async function(e) {
            e.preventDefault();
            if (!activeSession) {
                Swal.fire('คาบเรียนปิดแล้ว', 'ไม่สามารถส่ง OTP ได้ในขณะนี้', 'error');
                return;
            }

            const studentDocId = document.getElementById('student-select-name').value;
            const inputOTP = document.getElementById('student-input-otp').value.trim();
            const msgBox = document.getElementById('student-result-message');

            if (!studentDocId) {
                Swal.fire('กรุณาเลือกชื่อ', 'กรุณาเลือกรายชื่อนักเรียนของคุณ', 'warning');
                return;
            }

            if (inputOTP !== activeSession.currentOTP) {
                Swal.fire('OTP ไม่ถูกต้อง', 'รหัส OTP ไม่ถูกต้องหรือหมดอายุแล้ว กรุณาดูรหัสใหม่จากหน้าจอครู', 'error');
                return;
            }

            const targetStudent = students.find(s => s.id === studentDocId);
            const recordId = `${activeSession.id}_${studentDocId}`;
            const timeStr = new Date().toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' });

            await setDoc(getPublicDoc('attendances', recordId), {
                id: recordId,
                sessionId: activeSession.id,
                studentId: targetStudent.id,
                studentCode: targetStudent.studentId || '',
                studentName: targetStudent.name,
                otpSubmitted: inputOTP,
                status: 'VERIFIED',
                checkInTime: timeStr,
                timestamp: Date.now()
            });

            msgBox.classList.remove('hidden');
            msgBox.className = "p-4 rounded-xl text-sm font-medium bg-emerald-100 text-emerald-800 border border-emerald-300";
            msgBox.innerHTML = `🟢 <b>ลงชื่อเข้าเรียนสำเร็จ!</b><br>ยินดีต้อนรับ ${targetStudent.name} (บันทึกเวลา ${timeStr})`;

            document.getElementById('student-input-otp').value = '';
            Swal.fire('ลงชื่อสำเร็จ!', `เช็คชื่อคุณ ${targetStudent.name} เรียบร้อยแล้ว`, 'success');
        };

        function renderHistoryList() {
            const tbody = document.getElementById('history-table-body');
            if (!tbody) return;

            if (attendanceRecords.length === 0) {
                tbody.innerHTML = `<tr><td colspan="4" class="p-6 text-center text-slate-400">ยังไม่มีประวัติการเช็คชื่อ</td></tr>`;
                return;
            }

            // Group attendances by session
            const sessionsMap = {};
            attendanceRecords.forEach(r => {
                if (!sessionsMap[r.sessionId]) {
                    sessionsMap[r.sessionId] = [];
                }
                sessionsMap[r.sessionId].push(r);
            });

            tbody.innerHTML = Object.keys(sessionsMap).map(sessId => {
                const items = sessionsMap[sessId];
                const checked = items.filter(i => i.status === 'VERIFIED').length;
                const sampleTime = items[0]?.timestamp ? new Date(items[0].timestamp).toLocaleString('th-TH') : '-';

                return `
                    <tr class="hover:bg-slate-50 border-b border-slate-100">
                        <td class="p-3 pl-4 text-xs font-medium text-slate-500">${sampleTime}</td>
                        <td class="p-3 font-semibold text-slate-800">รหัสคาบ: ${sessId.slice(-6)}</td>
                        <td class="p-3 text-xs"><span class="font-bold text-emerald-600">${checked}</span> คน</td>
                        <td class="p-3"><span class="bg-emerald-50 text-emerald-700 border border-emerald-200 text-xs px-2.5 py-1 rounded-full font-medium">บันทึกแล้ว</span></td>
                    </tr>
                `;
            }).join('');
        }

        window.exportToCSV = function() {
            if (attendanceRecords.length === 0) {
                Swal.fire('ไม่มีข้อมูล', 'ไม่มีข้อมูลประวัติสำหรับส่งออก', 'info');
                return;
            }

            let csvContent = "data:text/csv;charset=utf-8,\uFEFF";
            csvContent += "วันที่-เวลา,รหัสคาบเรียน,รหัสนักเรียน,ชื่อ-นามสกุล,สถานะ,เวลาเช็คชื่อ\n";

            attendanceRecords.forEach(a => {
                const timeText = a.timestamp ? new Date(a.timestamp).toLocaleString('th-TH') : '-';
                const statusText = a.status === 'VERIFIED' ? 'มาเรียน' : 'รอตรวจสอบ';
                csvContent += `"${timeText}","${a.sessionId}","${a.studentCode || '-'}","${a.studentName}","${statusText}","${a.checkInTime || '-'}"\n`;
            });

            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", `attendance_live_report_${Date.now()}.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        };
    </script>
</body>
</html>
