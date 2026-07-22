# StudyPlanner
<!DOCTYPE html>
<html lang="zh-HK">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>溫習打卡與專注空間</title>
    <style>
        :root {
            --bg-color: #f4f6f8;
            --card-bg: #ffffff;
            --primary: #4a6fa5;
            --primary-light: #e8eff5;
            --accent: #ff6b6b;
            --text-main: #333333;
            --text-secondary: #666666;
            --border: #e0e0e0;
            --success: #2ecc71;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-main);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
        }

        .container {
            width: 100%;
            max-width: 850px;
        }

        header {
            text-align: center;
            margin-bottom: 25px;
        }

        header h1 {
            color: var(--primary);
            margin-bottom: 5px;
            font-size: 1.8rem;
        }

        header p {
            color: var(--text-secondary);
            font-size: 0.95rem;
        }

        .card {
            background-color: var(--card-bg);
            border-radius: 12px;
            padding: 20px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.05);
            margin-bottom: 20px;
        }

        .card h2 {
            font-size: 1.2rem;
            color: var(--primary);
            border-bottom: 2px solid var(--primary-light);
            padding-bottom: 8px;
            margin-top: 0;
        }

        /* 倒數與打卡區 */
        .dashboard-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
        }

        @media(max-width: 600px) {
            .dashboard-grid { grid-template-columns: 1fr; }
        }

        .stat-box {
            text-align: center;
            padding: 15px;
            background: var(--primary-light);
            border-radius: 8px;
        }

        .stat-box .number {
            font-size: 2rem;
            font-weight: bold;
            color: var(--primary);
        }

        .btn {
            background-color: var(--primary);
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 1rem;
            transition: background 0.2s;
            width: 100%;
            margin-top: 10px;
        }

        .btn:hover {
            opacity: 0.9;
        }

        .btn:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }

        /* Planner 表單 (配合科目、內容、日期、開始時間、時長) */
        form {
            display: grid;
            grid-template-columns: 1fr 1.5fr 1fr 1fr 1fr auto;
            gap: 8px;
            align-items: center;
        }

        @media(max-width: 950px) {
            form { grid-template-columns: 1fr; }
        }

        input, select {
            padding: 10px;
            border: 1px solid var(--border);
            border-radius: 6px;
            font-size: 0.9rem;
            width: 100%;
            box-sizing: border-box;
        }

        /* 任務列表 */
        .task-item {
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 12px 10px;
            border-bottom: 1px solid var(--border);
        }

        .task-item:last-child {
            border-bottom: none;
        }

        .task-info {
            display: flex;
            align-items: center;
            gap: 10px;
            flex-wrap: wrap;
        }

        .task-info.completed span {
            text-decoration: line-through;
            color: var(--text-secondary);
        }

        .tag {
            background: var(--primary-light);
            color: var(--primary);
            padding: 3px 8px;
            border-radius: 4px;
            font-size: 0.8rem;
            font-weight: bold;
        }

        .date-tag, .duration-tag {
            background: #f0f0f0;
            color: var(--text-secondary);
            padding: 3px 8px;
            border-radius: 4px;
            font-size: 0.8rem;
        }

        /* 鬧鐘/計時器 Modal 提示 */
        #alert-modal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.5);
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }

        .modal-content {
            background: white;
            padding: 30px;
            border-radius: 12px;
            text-align: center;
            max-width: 400px;
            width: 90%;
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
        }

        .timer-display {
            font-size: 2.5rem;
            font-weight: bold;
            color: var(--accent);
            margin: 15px 0;
        }

        .history-list {
            max-height: 150px;
            overflow-y: auto;
            font-size: 0.9rem;
            color: var(--text-secondary);
        }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>每日溫習打卡與專注空間</h1>
        <p>保持清晰頭腦，每日累積微小進步，直達 2027年4月8日！</p>
    </header>

    <div class="dashboard-grid">
        <div class="card" style="margin-bottom:0;">
            <h2>距離 2027年4月8日</h2>
            <div class="stat-box" style="margin-top: 15px;">
                <div class="number" id="countdown-days">--</div>
                <div>日</div>
            </div>
        </div>

        <div class="card" style="margin-bottom:0;">
            <h2>每日打卡</h2>
            <div style="text-align: center; margin-top: 15px;">
                <button class="btn" id="checkin-btn" onclick="toggleCheckIn()">今日打卡</button>
                <p id="checkin-status" style="margin-top: 10px; font-size: 0.9rem; color: var(--text-secondary);"></p>
            </div>
        </div>
    </div>

    <div class="card" style="margin-top: 20px;">
        <h2>新增溫習計劃</h2>
        <form id="planner-form" onsubmit="addPlan(event)">
            <select id="subject-select" required>
                <option value="" disabled selected>選擇科目</option>
                <option value="Chin">Chin</option>
                <option value="Eng">Eng</option>
                <option value="Math">Math</option>
                <option value="CS">CS</option>
                <option value="Bio">Bio</option>
                <option value="Geog">Geog</option>
            </select>
            <input type="text" id="task-detail" placeholder="詳細內容（例：做Past Paper）" required>
            <input type="date" id="task-date" required>
            <input type="time" id="task-time" title="開始時間" required>
            <input type="number" id="task-duration" placeholder="預計時長(分鐘)" min="5" max="720" value="60" title="預計溫習時間（分鐘）" required>
            <button type="submit" class="btn" style="margin-top:0;">新增</button>
        </form>
    </div>

    <div class="card">
        <h2>所有溫習任務</h2>
        <div id="task-list">
            </div>
    </div>

    <div class="card">
        <h2>歷史打卡紀錄</h2>
        <div class="history-list" id="history-list">
            </div>
    </div>
</div>

<div id="alert-modal">
    <div class="modal-content">
        <h2 id="modal-title" style="color: var(--accent);">溫習時間到！</h2>
        <p id="modal-desc">距離 2027年4月8日仲有 <span id="modal-countdown"></span> 日</p>
        <div class="timer-display" id="timer-display">05:00</div>
        <div style="display: flex; gap: 10px;">
            <button class="btn" id="pause-btn" onclick="toggleTimer()">暫停</button>
            <button class="btn" style="background-color: var(--text-secondary);" onclick="closeModal()">關閉 / 休息</button>
        </div>
    </div>
</div>

<script>
    // 目標日期：2027年4月8日
    const targetDate = new Date('2027-04-08T00:00:00');
    
    // 資料儲存變數 (使用 localStorage)
    let tasks = JSON.parse(localStorage.getItem('study_tasks')) || [];
    let checkIns = JSON.parse(localStorage.getItem('study_checkins')) || {};
    
    // 計時器變數
    let timerInterval = null;
    let timeLeft = 300; // 5分鐘 = 300秒
    let isTimerRunning = false;
    let activeTaskKey = null;

    // 初始化頁面
    function init() {
        updateCountdown();
        checkTodayCheckIn();
        
        // 自動將日期選擇器預設為今日
        document.getElementById('task-date').value = getTodayString();

        renderTasks();
        renderHistory();
        
        // 每 30 秒檢查一次是否有時段符合
        setInterval(checkTaskTimeAlert, 30000);
        setInterval(updateCountdown, 60000);
    }

    // 1. 倒數功能
    function updateCountdown() {
        const now = new Date();
        const diff = targetDate - now;
        const days = Math.ceil(diff / (1000 * 60 * 60 * 24));
        document.getElementById('countdown-days').innerText = days >= 0 ? days : 0;
        return days;
    }

    // 2. 每日打卡功能
    function getTodayString() {
        const today = new Date();
        const year = today.getFullYear();
        const month = String(today.getMonth() + 1).padStart(2, '0');
        const day = String(today.getDate()).padStart(2, '0');
        return `${year}-${month}-${day}`;
    }

    function checkTodayCheckIn() {
        const todayStr = getTodayString();
        const btn = document.getElementById('checkin-btn');
        const status = document.getElementById('checkin-status');
        
        if (checkIns[todayStr]) {
            btn.innerText = "今日已打卡 ✓";
            btn.disabled = true;
            status.innerText = "做得好！繼續保持溫習節奏。";
        } else {
            btn.innerText = "今日打卡";
            btn.disabled = false;
            status.innerText = "今日仲未打卡，快啲開始溫書啦！";
        }
    }

    function toggleCheckIn() {
        const todayStr = getTodayString();
        checkIns[todayStr] = true;
        localStorage.setItem('study_checkins', JSON.stringify(checkIns));
        checkTodayCheckIn();
        renderHistory();
    }

    function renderHistory() {
        const historyList = document.getElementById('history-list');
        historyList.innerHTML = '';
        const sortedDates = Object.keys(checkIns).sort().reverse();
        
        if (sortedDates.length === 0) {
            historyList.innerHTML = '<p>暫時未有打卡紀錄，今日開始第一個打卡啦！</p>';
            return;
        }

        sortedDates.forEach(date => {
            const div = document.createElement('div');
            div.style.padding = '5px 0';
            div.innerHTML = `✅ 打卡成功日期：<strong>${date}</strong>`;
            historyList.appendChild(div);
        });
    }

    // 3. Study Planner 功能（加入計算完成時間）
    function addPlan(event) {
        event.preventDefault();
        const subject = document.getElementById('subject-select').value;
        const detail = document.getElementById('task-detail').value;
        const date = document.getElementById('task-date').value;
        const time = document.getElementById('task-time').value;
        const duration = parseInt(document.getElementById('task-duration').value) || 60;

        // 計算計劃完成時間（開始時間 + 預計時長）
        const [hours, minutes] = time.split(':').map(Number);
        const startDate = new Date();
        startDate.setHours(hours, minutes + duration, 0);
        const endHours = String(startDate.getHours()).padStart(2, '0');
        const endMinutes = String(startDate.getMinutes()).padStart(2, '0');
        const endTime = `${endHours}:${endMinutes}`;

        const newTask = {
            id: Date.now(),
            subject,
            detail,
            date,
            time,
            duration,
            endTime,
            completed: false
        };

        tasks.push(newTask);
        saveAndRenderTasks();
        
        // 重設表單，但保留預設日期為今天
        document.getElementById('planner-form').reset();
        document.getElementById('task-date').value = getTodayString();
        document.getElementById('task-duration').value = 60;
    }

    function toggleTask(id) {
        tasks = tasks.map(t => t.id === id ? { ...t, completed: !t.completed } : t);
        saveAndRenderTasks();
    }

    function deleteTask(id) {
        tasks = tasks.filter(t => t.id !== id);
        saveAndRenderTasks();
    }

    function saveAndRenderTasks() {
        localStorage.setItem('study_tasks', JSON.stringify(tasks));
        renderTasks();
    }

    function renderTasks() {
        const taskList = document.getElementById('task-list');
        taskList.innerHTML = '';

        if (tasks.length === 0) {
            taskList.innerHTML = '<p style="color: var(--text-secondary); text-align: center;">暫時未有溫習計劃，請在上方新增！</p>';
            return;
        }

        // 按日期與時間排序
        tasks.sort((a, b) => {
            if (a.date !== b.date) return a.date.localeCompare(b.date);
            return a.time.localeCompare(b.time);
        });

        tasks.forEach(task => {
            const div = document.createElement('div');
            div.className = 'task-item';
            div.innerHTML = `
                <div class="task-info ${task.completed ? 'completed' : ''}">
                    <input type="checkbox" ${task.completed ? 'checked' : ''} onclick="toggleTask(${task.id})">
                    <span class="tag">${task.subject}</span>
                    <span class="date-tag">📅 ${task.date}</span>
                    <span class="duration-tag">⏰ ${task.time} ~ ${task.endTime} (${task.duration}分鐘)</span>
                    <span><strong>${task.detail}</strong></span>
                </div>
                <button class="btn" style="width: auto; padding: 5px 10px; background-color: var(--accent); margin-top:0;" onclick="deleteTask(${task.id})">刪除</button>
            `;
            taskList.appendChild(div);
        });
    }

    // 4. 時段提醒與 5 分鐘計時器功能
    function checkTaskTimeAlert() {
        const now = new Date();
        const currentDateStr = getTodayString();
        const currentHours = String(now.getHours()).padStart(2, '0');
        const currentMinutes = String(now.getMinutes()).padStart(2, '0');
        const currentTimeStr = `${currentHours}:${currentMinutes}`;

        tasks.forEach(task => {
            const taskKey = `${task.date}_${task.time}`;
            if (task.date === currentDateStr && task.time === currentTimeStr && !task.completed && activeTaskKey !== taskKey) {
                activeTaskKey = taskKey;
                showAlertModal(task);
            }
        });
    }

    function showAlertModal(task) {
        const modal = document.getElementById('alert-modal');
        const daysLeft = document.getElementById('countdown-days').innerText;
        document.getElementById('modal-countdown').innerText = daysLeft;
        document.getElementById('modal-title').innerText = `夠鐘溫書啦！(${task.subject})`;
        
        modal.style.display = 'flex';
        startTimer();
    }

    function startTimer() {
        clearInterval(timerInterval);
        timeLeft = 300; // 5分鐘 = 300秒
        isTimerRunning = true;
        document.getElementById('pause-btn').innerText = "暫停";
        updateTimerDisplay();

        timerInterval = setInterval(() => {
            if (isTimerRunning) {
                if (timeLeft > 0) {
                    timeLeft--;
                    updateTimerDisplay();
                } else {
                    clearInterval(timerInterval);
                    isTimerRunning = false;
                    alert("5分鐘專注計時結束！休息時間差不多了，準備好繼續高效溫習了嗎？");
                    closeModal();
                }
            }
        }, 1000);
    }

    function toggleTimer() {
        isTimerRunning = !isTimerRunning;
        const btn = document.getElementById('pause-btn');
        btn.innerText = isTimerRunning ? "暫停" : "繼續";
    }

    function updateTimerDisplay() {
        const minutes = Math.floor(timeLeft / 60);
        const seconds = timeLeft % 60;
        document.getElementById('timer-display').innerText = 
            `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
    }

    function closeModal() {
        clearInterval(timerInterval);
        isTimerRunning = false;
        document.getElementById('alert-modal').style.display = 'none';
    }

    // 頁面載動時執行
    window.onload = init;
</script>

</body>
</html>
