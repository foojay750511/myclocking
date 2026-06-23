<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>行動差勤與簽核管理系統</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        body { background-color: #f5f7fa; color: #333; padding-bottom: 70px; }
        .header { background: linear-gradient(135deg, #1e3c72, #2a5298); color: white; padding: 20px; text-align: center; border-bottom-left-radius: 15px; border-bottom-right-radius: 15px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
        .header h1 { font-size: 22px; margin-bottom: 5px; }
        .header p { font-size: 13px; opacity: 0.8; }
        .container { padding: 15px; max-width: 600px; margin: 0 auto; }
        .card { background: white; border-radius: 12px; padding: 18px; margin-bottom: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.04); border: 1px solid #eef2f5; }
        .card h3 { margin-bottom: 15px; font-size: 16px; color: #1e3c72; border-left: 4px solid #2a5298; padding-left: 8px; }
        
        .form-group { margin-bottom: 15px; }
        .form-group label { display: block; margin-bottom: 6px; font-size: 13px; font-weight: 600; color: #4a5568; }
        .form-control { width: 100%; padding: 11px; border: 1px solid #cbd5e0; border-radius: 8px; font-size: 14px; background-color: #fff; }
        .form-control:focus { border-color: #2a5298; outline: none; box-shadow: 0 0 0 3px rgba(42,82,152,0.1); }
        
        .btn { display: block; width: 100%; padding: 12px; margin-top: 5px; margin-bottom: 10px; font-size: 15px; font-weight: bold; border: none; border-radius: 8px; color: white; cursor: pointer; text-align: center; transition: all 0.2s; }
        .btn:disabled { background-color: #a0aec0 !important; cursor: not-allowed; transform: none !important; }
        .btn-primary { background-color: #2a5298; }
        .btn-success { background-color: #38a169; }
        .btn-danger { background-color: #e53e3e; }
        .btn-warning { background-color: #dd6b20; }
        
        .nav-bar { position: fixed; bottom: 0; left: 0; right: 0; background: white; display: flex; height: 60px; box-shadow: 0 -3px 12px rgba(0,0,0,0.08); border-top: 1px solid #edf2f7; z-index: 100; }
        .nav-item { flex: 1; text-align: center; display: flex; flex-direction: column; justify-content: center; align-items: center; color: #718096; font-size: 11px; text-decoration: none; cursor: pointer; }
        .nav-item.active { color: #2a5298; font-weight: bold; }
        .nav-icon { font-size: 20px; margin-bottom: 2px; }
        
        .page { display: none; }
        .page.active { display: block; }
        
        .table-responsive { overflow-x: auto; margin-top: 10px; }
        .data-table { width: 100%; border-collapse: collapse; font-size: 12px; text-align: left; }
        .data-table th, .data-table td { border: 1px solid #e2e8f0; padding: 10px; }
        .data-table th { background-color: #f7fafc; color: #4a5568; font-weight: bold; }
        
        /* 狀態標籤 */
        .status-badge { padding: 4px 8px; border-radius: 6px; color: white; font-size: 11px; font-weight: bold; display: inline-block; }
        .status-pending { background-color: #dd6b20; }  /* 送審中 */
        .status-signing { background-color: #3182ce; }  /* 簽核中 */
        .status-approved { background-color: #38a169; } /* 核准 */
        .status-rejected { background-color: #e53e3e; } /* 退回 */
        
        .review-box { background: #f7fafc; padding: 10px; border-radius: 8px; margin-top: 8px; border: 1px dashed #cbd5e0; }
    </style>
</head>
<body>

    <div class="header">
        <h1>雲端差勤與簽核管理系統</h1>
        <p>修正優化版 · 支援進階權限審核</p>
    </div>

    <div class="container">
        <div class="card" style="border: 1px solid #2a5298;">
            <h3>🔑 個人身份設定</h3>
            <div style="display: flex; gap: 10px;">
                <div class="form-group" style="flex: 1;">
                    <label>員工工號</label>
                    <input type="text" id="emp-id" class="form-control" placeholder="例如: A001" onchange="saveEmpInfo()">
                </div>
                <div class="form-group" style="flex: 1;">
                    <label>員工姓名</label>
                    <input type="text" id="emp-name" class="form-control" placeholder="例如: 張先生" onchange="saveEmpInfo()">
                </div>
            </div>
            <div class="form-group">
                <label>身份權限切換</label>
                <select id="user-role" class="form-control" onchange="toggleAdminTab()">
                    <option value="user">一般員工</option>
                    <option value="admin">最高權限管理員</option>
                </select>
            </div>
        </div>

        <div id="page-clock" class="page active">
            <div class="card">
                <h3>⏰ 今日出勤打卡 (防重複限制)</h3>
                <button class="btn btn-success" id="btn-clock-in" onclick="submitClock('上班')">🌞 上班打卡</button>
                <button class="btn btn-danger" id="btn-clock-out" onclick="submitClock('下班')">🌜 下班打卡</button>
            </div>
        </div>

        <div id="page-leave" class="page">
            <div class="card">
                <h3>🌴 線上請假申請 (預設送審中)</h3>
                <div class="form-group">
                    <label>假別選擇</label>
                    <select id="leave-type" class="form-control">
                        <option value="特休">特休 (薪資照給)</option>
                        <option value="病假">病假 (半薪)</option>
                        <option value="事假">事假 (無薪)</option>
                        <option value="公假/婚假/喪假">公假/婚假/喪假</option>
                    </select>
                </div>
                <div class="form-group">
                    <label>請假 開始時間</label>
                    <input type="datetime-local" id="leave-start" class="form-control">
                </div>
                <div class="form-group">
                    <label>請假 結束時間</label>
                    <input type="datetime-local" id="leave-end" class="form-control">
                </div>
                <div class="form-group">
                    <label>請假事由</label>
                    <input type="text" id="leave-reason" class="form-control" placeholder="請輸入請假原因">
                </div>
                <button class="btn btn-primary" id="btn-leave" onclick="submitLeave()">送出請假申請</button>
            </div>
        </div>

        <div id="page-overtime" class="page">
            <div class="card">
                <h3>💪 加班延長工時申報 (起訖時間模式)</h3>
                <div class="form-group">
                    <label>加班 開始時間</label>
                    <input type="datetime-local" id="ot-start" class="form-control" onchange="calculateOTHours()">
                </div>
                <div class="form-group">
                    <label>加班 結束時間</label>
                    <input type="datetime-local" id="ot-end" class="form-control" onchange="calculateOTHours()">
                </div>
                <div class="form-group">
                    <label>系統自動計算時數</label>
                    <input type="text" id="ot-hours-display" class="form-control" style="background-color: #e2e8f0;" readonly placeholder="請選擇起訖時間">
                </div>
                <div class="form-group">
                    <label>業務內容概述</label>
                    <input type="text" id="ot-reason" class="form-control" placeholder="請簡述加班處理業務">
                </div>
                <button class="btn btn-primary" id="btn-overtime" onclick="submitOvertime()">送出加班申報</button>
            </div>
        </div>

        <div id="page-history" class="page">
            <div class="card">
                <h3 id="history-title">🔍 個人差勤與簽核進度</h3>
                <div id="loading" style="display:none; text-align:center; padding:10px; color:#666;">通訊中，請稍候...</div>
                <div class="table-responsive">
                    <table class="data-table">
                        <thead>
                            <tr>
                                <th>類別/時間</th>
                                <th>項目內容</th>
                                <th>狀態/審核人員</th>
                            </tr>
                        </thead>
                        <tbody id="history-data"></tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>

    <div class="nav-bar">
        <div class="nav-item active" onclick="switchPage('clock')"><span class="nav-icon">⏰</span>打卡</div>
        <div class="nav-item" onclick="switchPage('leave')"><span class="nav-icon">🌴</span>請假</div>
        <div class="nav-item" onclick="switchPage('overtime')"><span class="nav-icon">💪</span>加班</div>
        <div class="nav-item" id="nav-history-btn" onclick="switchPage('history')"><span class="nav-icon">🔍</span>進度查詢</div>
    </div>

    <script>
        // ⚠️ 請在此處替換成您在第一階段重新部署得到的 Google GAS 正式網址 ⚠️
        const GAS_URL = "https://script.google.com/macros/s/AKfycbzg8-5hz86tCwUpKIMXhfOMmBcLojme-gBrxGMCA87HxelHjrTtJ5j_wb72pH8J7_4/exec"; 

        document.addEventListener("DOMContentLoaded", function() {
            if (localStorage.getItem('clock_emp_id')) document.getElementById('emp-id').value = localStorage.getItem('clock_emp_id');
            if (localStorage.getItem('clock_emp_name')) document.getElementById('emp-name').value = localStorage.getItem('clock_emp_name');
        });

        function saveEmpInfo() {
            localStorage.setItem('clock_emp_id', document.getElementById('emp-id').value.trim());
            localStorage.setItem('clock_emp_name', document.getElementById('emp-name').value.trim());
        }

        function validateIdentity() {
            const empId = document.getElementById('emp-id').value.trim();
            const empName = document.getElementById('emp-name').value.trim();
            if (!empId || !empName) { alert("❌ 請先填寫工號與姓名！"); return false; }
            return { empId, empName };
        }

        function toggleAdminTab() {
            const role = document.getElementById('user-role').value;
            const btn = document.getElementById('nav-history-btn');
            if (role === 'admin') {
                btn.innerHTML = '<span class="nav-icon">🛠️</span>最高簽核後台';
            } else {
                btn.innerHTML = '<span class="nav-icon">🔍</span>進度查詢';
            }
            if(document.getElementById('page-history').classList.contains('active')) fetchHistory();
        }

        function switchPage(pageId) {
            document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
            document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
            document.getElementById('page-' + pageId).classList.add('active');
            event.currentTarget.classList.add('active');
            if(pageId === 'history') fetchHistory();
        }

        function calculateOTHours() {
            const start = document.getElementById('ot-start').value;
            const end = document.getElementById('ot-end').value;
            if (!start || !end) return;
            const diff = new Date(end) - new Date(start);
            if (diff <= 0) { alert("❌ 結束時間必須大於開始時間！"); document.getElementById('ot-end').value = ""; return; }
            document.getElementById('ot-hours-display').value = (diff / (1000 * 60 * 60)).toFixed(1) + " 小時";
        }

        // 前端按鈕鎖（防重複送出）
        function setSubmitting(isSubmit, btnId) {
            const btns = ['btn-clock-in', 'btn-clock-out', 'btn-leave', 'btn-overtime'];
            btns.forEach(id => {
                const b = document.getElementById(id);
                if(b) {
                    b.disabled = isSubmit;
                    if(isSubmit && id === btnId) b.innerText = "連線傳送中，請勿關閉...";
                }
            });
            if(!isSubmit) {
                if(document.getElementById('btn-clock-in')) document.getElementById('btn-clock-in').innerText = "🌞 上班打卡";
                if(document.getElementById('btn-clock-out')) document.getElementById('btn-clock-out').innerText = "🌜 下班打卡";
                if(document.getElementById('btn-leave')) document.getElementById('btn-leave').innerText = "送出請假申請";
                if(document.getElementById('btn-overtime')) document.getElementById('btn-overtime').innerText = "送出加班申報";
            }
        }

        function sendPayload(payload, successMsg, btnId) {
            const identity = validateIdentity();
            if(!identity) return;
            payload.empId = identity.empId;
            payload.empName = identity.empName;

            setSubmitting(true, btnId);

            fetch(GAS_URL, { method: "POST", body: JSON.stringify(payload) })
            .then(res => res.json())
            .then(res => {
                setSubmitting(false);
                if(res.status === "success") {
                    alert(successMsg);
                    if(payload.action==='leave') { document.getElementById('leave-start').value=''; document.getElementById('leave-end').value=''; document.getElementById('leave-reason').value=''; }
                    if(payload.action==='overtime') { document.getElementById('ot-start').value=''; document.getElementById('ot-end').value=''; document.getElementById('ot-hours-display').value=''; document.getElementById('ot-reason').value=''; }
                } else { alert("失敗：" + res.message); }
            })
            .catch(err => { setSubmitting(false); alert("伺服器連線失敗"); });
        }

        function submitClock(type) {
            const identity = validateIdentity(); if(!identity) return;
            const btnId = type === '上班' ? 'btn-clock-in' : 'btn-clock-out';
            navigator.geolocation.getCurrentPosition(
                (pos) => {
                    sendPayload({ action: "clock", type: type, lat: pos.coords.latitude, lng: pos.coords.longitude }, `${type}打卡成功！`, btnId);
                },
                (err) => { alert("GPS定位失敗，網頁打卡必須允許位置存取權限。"); },
                { enableHighAccuracy: true, timeout: 7000 }
            );
        }

        function submitLeave() {
            const start = document.getElementById('leave-start').value;
            const end = document.getElementById('leave-end').value;
            const reason = document.getElementById('leave-reason').value;
            if(!start || !end || !reason) { alert("請填寫完整請假欄位"); return; }
            sendPayload({ action: "leave", type: document.getElementById('leave-type').value, start, end, reason }, "請假單已送出，目前狀態：送審中", "btn-leave");
        }

        function submitOvertime() {
            const start = document.getElementById('ot-start').value;
            const end = document.getElementById('ot-end').value;
            const hours = document.getElementById('ot-hours-display').value;
            const reason = document.getElementById('ot-reason').value;
            if(!start || !end || !reason) { alert("請填寫完整加班欄位"); return; }
            sendPayload({ action: "overtime", start, end, hours, reason }, "加班單已送出，目前狀態：送審中", "btn-overtime");
        }

        // 讀取歷史與審核資料
        function fetchHistory() {
            const identity = validateIdentity(); if(!identity) return;
            const role = document.getElementById('user-role').value;
            const loading = document.getElementById('loading');
            const tbody = document.getElementById('history-data');
            
            document.getElementById('history-title').innerText = role === 'admin' ? "🛠️ 最高權限簽核後台" : "🔍 個人差勤與簽核進度";
            loading.style.display = "block"; tbody.innerHTML = "";

            fetch(`${GAS_URL}?mode=${role}&empId=${identity.empId}`)
            .then(res => res.json())
            .then(data => {
                loading.style.display = "none";
                if(data.length === 0) { tbody.innerHTML = `<tr><td colspan="3" style="text-align:center;color:#999;">暫無紀錄</td></tr>`; return; }
                
                data.forEach(item => {
                    let badgeColor = "status-pending";
                    if(item.status === "核准") badgeColor = "status-approved";
                    if(item.status === "退回") badgeColor = "status-rejected";
                    if(item.status === "簽核中") badgeColor = "status-signing";

                    let actionHtml = "";
                    // 修正事項 4 & 5：如果是管理員權限，直接在前端畫面上長出審核與修改按鈕
                    if(role === 'admin' && item.mainType !== '打卡') {
                        actionHtml = `
                            <div class="review-box">
                                <strong>審核操作：</strong><br>
                                負責人姓名：<input type="text" id="rev-name-${item.id}" style="width:70px;padding:2px;" placeholder="主管名"><br>
                                意見：<input type="text" id="rev-comm-${item.id}" style="width:110px;padding:2px;" placeholder="備註"><br>
                                <button onclick="executeReview('${item.id}', '核准')" style="background-color:#38a169;color:white;border:none;padding:3px;margin-top:4px;border-radius:4px;cursor:pointer;">核准</button>
                                <button onclick="executeReview('${item.id}', '簽核中')" style="background-color:#3182ce;color:white;border:none;padding:3px;margin-top:4px;border-radius:4px;cursor:pointer;">簽核中</button>
                                <button onclick="executeReview('${item.id}', '退回')" style="background-color:#e53e3e;color:white;border:none;padding:3px;margin-top:4px;border-radius:4px;cursor:pointer;">退回</button>
                            </div>
                        `;
                    }

                    const row = `<tr>
                        <td><strong>[${item.mainType}]</strong><br><span style="color:#718096;font-size:11px;">${item.time}</span></td>
                        <td><span class="status-badge ${item.mainType==='打卡'?'status-approved':''}">${item.type}</span><br>${item.detail} ${actionHtml}</td>
                        <td><span class="status-badge ${badgeColor}">${item.status}</span><br><span style="font-size:11px;color:#718096;">簽核人: ${item.reviewer}</span></td>
                    </tr>`;
                    tbody.innerHTML += row;
                });
            })
            .catch(err => { loading.style.display = "none"; });
        }

        // 執行審核與簽核狀態更新
        function executeReview(targetId, statusValue) {
            const reviewer = document.getElementById(`rev-name-${targetId}`).value.trim();
            const comment = document.getElementById(`rev-comm-${targetId}`).value.trim();
            if(!reviewer) { alert("請填寫審核人員（主管姓名）"); return; }
            
            fetch(GAS_URL, {
                method: "POST",
                body: JSON.stringify({ action: "review", targetId, status: statusValue, reviewer, comment })
            })
            .then(res => res.json())
            .then(res => { if(res.status==='success') { alert('審核狀態更新成功！'); fetchHistory(); } })
        }
    </script>
</body>
</html>
