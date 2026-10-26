# decision-experiment
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>의사결정 및 반응시간 측정 실험</title>
    <style>
        * {
            box-sizing: border-box;
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
        }
        body {
            background-color: #f5f7fa;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
        }
        .container {
            background-color: #ffffff;
            padding: 40px;
            border-radius: 16px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.08);
            max-width: 700px;
            width: 100%;
            text-align: center;
            position: relative;
            z-index: 1;
        }
        .screen {
            display: none;
        }
        .screen.active {
            display: block;
        }
        h2 {
            color: #2c3e50;
            font-size: 1.35rem;
            line-height: 1.6;
            margin-bottom: 25px;
            word-break: keep-all;
        }
        .stat-box {
            background-color: #eef2f7;
            border-left: 5px solid #3498db;
            padding: 20px;
            border-radius: 8px;
            margin-bottom: 30px;
            font-size: 1.1rem;
            color: #34495e;
            line-height: 1.6;
            word-break: keep-all;
        }
        .btn-group {
            display: flex;
            flex-direction: column;
            gap: 12px;
        }
        button {
            background-color: #4a90e2;
            color: white;
            border: none;
            padding: 16px 24px;
            font-size: 1.05rem;
            font-weight: 600;
            border-radius: 8px;
            cursor: pointer;
            transition: background-color 0.2s ease, transform 0.1s ease;
            position: relative;
            z-index: 2;
        }
        button:hover {
            background-color: #357abd;
        }
        button:active {
            transform: scale(0.98);
        }
        button.next-btn {
            background-color: #2ecc71;
        }
        button.next-btn:hover {
            background-color: #27ae60;
        }
        
        /* 관리자 아이콘 버튼 */
        .admin-trigger {
            position: absolute;
            top: 15px;
            right: 20px;
            background: none;
            border: none;
            font-size: 1.2rem;
            color: #ccc;
            cursor: pointer;
            padding: 5px;
            z-index: 10;
        }
        .admin-trigger:hover {
            color: #666;
        }

        /* 관리자 모달 및 테이블 스타일 */
        .modal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.6);
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        .modal-content {
            background: white;
            padding: 30px;
            border-radius: 12px;
            max-width: 750px;
            width: 90%;
            max-height: 80vh;
            overflow-y: auto;
            text-align: left;
            position: relative;
            z-index: 1001;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 15px;
            font-size: 0.9rem;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 10px;
            text-align: center;
        }
        th {
            background-color: #f2f2f2;
            color: #333;
        }
        
        /* 모달 하단 버튼 영역 */
        .modal-btn-group {
            display: flex;
            justify-content: flex-end;
            gap: 10px;
            margin-top: 20px;
        }
        .close-btn {
            background-color: #e74c3c;
            padding: 10px 18px;
            font-size: 0.9rem;
        }
        .close-btn:hover {
            background-color: #c0392b;
        }
        .csv-btn {
            background-color: #3498db;
            padding: 10px 18px;
            font-size: 0.9rem;
        }
        .csv-btn:hover {
            background-color: #2980b9;
        }

        /* 개별 삭제 버튼 스타일 */
        .delete-btn {
            background-color: #e74c3c;
            color: white;
            border: none;
            padding: 6px 12px;
            font-size: 0.8rem;
            border-radius: 4px;
            cursor: pointer;
        }
        .delete-btn:hover {
            background-color: #c0392b;
        }
    </style>
</head>
<body>

<div class="container">
    <!-- 오른쪽 상단 관리자 전용 숨김 버튼 (⚙️ 아이콘) -->
    <button type="button" class="admin-trigger" onclick="openAdminModal()" title="연구자 통계창">⚙️</button>

    <!-- 화면 1: 질문 1 (전차 딜레마) -->
    <div id="screen1" class="screen active">
        <h2>브레이크가 고장난 전차가 5명의 인부를 향해 가고 있고, 선로를 바꾸면 다른 곳에 있는 1명이 죽습니다.</h2>
        <div class="btn-group">
            <button type="button" onclick="selectOption1('선로를 바꾼다')">선로를 바꾼다</button>
            <button type="button" onclick="selectOption1('선로를 바꾸지 않는다')">선로를 바꾸지 않는다</button>
        </div>
    </div>

    <!-- 화면 2: 질문 1 통계 메시지 확인 (동조 자극 노출) -->
    <div id="screen2" class="screen">
        <div id="statMessage" class="stat-box"></div>
        <button type="button" class="next-btn" onclick="goToScreen3()">넘어가기</button>
    </div>

    <!-- 화면 3: 질문 2 (아기 딜레마) -->
    <div id="screen3" class="screen">
        <h2>전시 상황에서 적의 발각을 피하기 위해 은폐해 있을 때, 아기가 울어 모두가 죽을 위기에 처합니다.</h2>
        <div class="btn-group">
            <button type="button" onclick="selectOption2('아이를 죽인다')">아이를 죽인다</button>
            <button type="button" onclick="selectOption2('아이를 살린다')">아이를 살린다</button>
        </div>
    </div>

    <!-- 화면 4: 일반 피실험자용 단순 종료 화면 -->
    <div id="screen4" class="screen">
        <h2>실험에 참여해 주셔서 감사합니다.</h2>
        <p style="color: #666; margin-top: 15px;">모든 응답이 성공적으로 제출되었습니다.</p>
    </div>
</div>

<!-- 연구자 전용 관리자 데이터 모달 창 -->
<div id="adminModal" class="modal">
    <div class="modal-content">
        <h3 style="margin-top:0; color:#2c3e50;">🔒 연구자 전용 수집 데이터 통계창</h3>
        <p style="font-size: 0.85rem; color: #666;">* 소요 시간은 밀리초(ms) 및 초(s) 단위로 누적 저장됩니다.</p>
        
        <table>
            <thead>
                <tr>
                    <th>참가자</th>
                    <th>1차 선택</th>
                    <th>$T_1$ (1차 시간)</th>
                    <th>2차 선택</th>
                    <th>$T_2$ (2차 시간)</th>
                    <th>비율 ($T_1 / T_2$)</th>
                    <th>삭제</th>
                </tr>
            </thead>
            <tbody id="adminDataTable">
                <!-- 데이터 삽입 영역 -->
            </tbody>
        </table>

        <div class="modal-btn-group">
            <button type="button" class="csv-btn" onclick="exportToCSV()">CSV 엑셀 다운로드</button>
            <button type="button" class="close-btn" onclick="closeAdminModal()">창 닫기</button>
        </div>
    </div>
</div>

<script>
    let startTime1 = 0, endTime1 = 0;
    let startTime2 = 0, endTime2 = 0;

    let allData = JSON.parse(localStorage.getItem('dnj_experiment_logs')) || [];

    let currentSession = {
        id: 0,
        q1_choice: "",
        t1_ms: 0,
        q2_choice: "",
        t2_ms: 0
    };

    window.onload = function() {
        startTime1 = performance.now();
    };

    function selectOption1(choice) {
        endTime1 = performance.now();
        currentSession.t1_ms = Math.round(endTime1 - startTime1);
        currentSession.q1_choice = choice;

        const statBox = document.getElementById('statMessage');
        if (choice === '선로를 바꾼다') {
            statBox.innerText = '23%의 사람은 선로를 바꾸지 않기로 결정했고, 77%의 사람들은 선로를 바꾸기로 결정했습니다.';
        } else {
            statBox.innerText = '77%의 사람들은 선로를 바꾸기로 하였고, 23%의 사람들은 선로를 바꾸기로 결정했습니다.';
        }

        document.getElementById('screen1').classList.remove('active');
        document.getElementById('screen2').classList.add('active');
    }

    function goToScreen3() {
        document.getElementById('screen2').classList.remove('active');
        document.getElementById('screen3').classList.add('active');
        startTime2 = performance.now();
    }

    function selectOption2(choice) {
        endTime2 = performance.now();
        currentSession.t2_ms = Math.round(endTime2 - startTime2);
        currentSession.q2_choice = choice;
        currentSession.id = allData.length + 1;

        allData.push({...currentSession});
        localStorage.setItem('dnj_experiment_logs', JSON.stringify(allData));

        document.getElementById('screen3').classList.remove('active');
        document.getElementById('screen4').classList.add('active');
    }

    function openAdminModal() {
        const password = prompt("연구자 비밀번호를 입력하세요:", "");
        if (password === "1234") {
            renderAdminTable();
            document.getElementById('adminModal').style.display = 'flex';
        } else if (password !== null) {
            alert("비밀번호가 일치하지 않습니다.");
        }
    }

    function closeAdminModal() {
        document.getElementById('adminModal').style.display = 'none';
    }

    function deleteRow(index) {
        if (confirm("해당 데이터를 삭제하시겠습니까?")) {
            allData.splice(index, 1);
            localStorage.setItem('dnj_experiment_logs', JSON.stringify(allData));
            renderAdminTable();
        }
    }

    function renderAdminTable() {
        const tableBody = document.getElementById('adminDataTable');
        tableBody.innerHTML = "";

        if (allData.length === 0) {
            tableBody.innerHTML = "<tr><td colspan='7'>수집된 데이터가 없습니다.</td></tr>";
            return;
        }

        allData.forEach((data, index) => {
            const t1_sec = (data.t1_ms / 1000).toFixed(2);
            const t2_sec = (data.t2_ms / 1000).toFixed(2);
            const ratio = (data.t2_ms > 0) ? (data.t1_ms / data.t2_ms).toFixed(2) : "N/A";

            const row = `
                <tr>
                    <td>#${data.id}</td>
                    <td>${data.q1_choice}</td>
                    <td>${data.t1_ms}ms (${t1_sec}초)</td>
                    <td>${data.q2_choice}</td>
                    <td>${data.t2_ms}ms (${t2_sec}초)</td>
                    <td><strong>${ratio}</strong></td>
                    <td><button type="button" class="delete-btn" onclick="deleteRow(${index})">삭제</button></td>
                </tr>
            `;
            tableBody.innerHTML += row;
        });
    }

    function exportToCSV() {
        if (allData.length === 0) {
            alert("다운로드할 데이터가 없습니다.");
            return;
        }

        let csvContent = "\uFEFF참가자,1차 선택,1차 시간(ms),2차 선택,2차 시간(ms),비율(T1/T2)\n";

        allData.forEach((data) => {
            const ratio = (data.t2_ms > 0) ? (data.t1_ms / data.t2_ms).toFixed(2) : "N/A";
            csvContent += `#${data.id},${data.q1_choice},${data.t1_ms},${data.q2_choice},${data.t2_ms},${ratio}\n`;
        });

        const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
        const link = document.createElement("a");
        const url = URL.createObjectURL(blob);
        link.setAttribute("href", url);
        link.setAttribute("download", `experiment_data_${new Date().toISOString().slice(0, 10)}.csv`);
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    }
</script>

</body>
</html>
