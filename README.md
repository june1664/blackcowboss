<!DOCTYPE html>
<html>
<head>
    <title>텔레그램 자동 메시지 전송</title>
    <style>
        #messageList {
            max-height: 300px;
            overflow-y: auto;
            border: 1px solid #ccc;
            padding: 5px;
        }
    </style>
</head>
<body>
    <h1>텔레그램 자동 메시지 전송</h1>

    <div>
        <textarea id="rssUrlInput" rows="1" cols="50" placeholder="RSS URL을 입력하세요."></textarea>
        <select id="intervalSelect">
            <option value="30">30초</option>
            <option value="60">1분</option>
            <option value="300">5분</option>
            <option value="600">10분</option>
            <option value="900">15분</option>
            <option value="1800">30분</option>
            <option value="3600">1시간</option>
            <option value="10800">3시간</option>
            <option value="21600">6시간</option>
            <option value="32400">9시간</option>
            <option value="86400">1일</option>
            <option value="172800">2일</option>
            <option value="259200">3일</option>
        </select>
        <button onclick="startRssFeed()">RSS 피드 전송 시작</button>
        <button onclick="stopRssFeed()">RSS 피드 전송 중지</button>
        <span id="countdown"></span>
    </div>

    <h2>전송된 메시지 목록</h2>
    <ul id="messageList"></ul>

    <script>
        const botToken = '7762839036:AAGbrNLwnor2LcgrZqfuHHjZiitP9AN8NcQ';
        const chatId = '439543133';
        const messageList = document.getElementById('messageList');
        let rssInterval;
        let previousItems = [];
        let countdownInterval;
        let remainingSeconds;
        let rssData = [];
        let isFirstRun = true;

        function sendMessageToTelegram(message) {
            const apiUrl = `https://api.telegram.org/bot${botToken}/sendMessage?chat_id=${chatId}&text=${encodeURIComponent(message)}`;

            fetch(apiUrl)
                .then(response => {
                    if (!response.ok) {
                        throw new Error(`HTTP error! status: ${response.status}`);
                    }
                    return response.json();
                })
                .then(data => {
                    console.log('메시지 전송 성공:', data);
                    const listItem = document.createElement('li');
                    const now = new Date();
                    const time = now.toLocaleString();
                    listItem.textContent = `[${time}] ${message}`;
                    messageList.appendChild(listItem);
                })
                .catch(error => {
                    console.error('메시지 전송 실패:', error);
                    const listItem = document.createElement('li');
                    const now = new Date();
                    const time = now.toLocaleString();
                    listItem.textContent = `[${time}] ${message} (전송 실패)`;
                    messageList.appendChild(listItem);
                });
        }

        function fetchRssFeed(rssUrl) {
            fetch(`https://api.rss2json.com/v1/api.json?rss_url=${encodeURIComponent(rssUrl)}`)
                .then(response => response.json())
                .then(data => {
                    if (data.status === 'ok') {
                        rssData = data.items;
                        sendRssFeed();
                    } else {
                        console.error('RSS 피드 가져오기 실패:', data.message);
                    }
                })
                .catch(error => {
                    console.error('RSS 피드 가져오기 오류:', error);
                });
        }

        function sendRssFeed() {
            if (rssData.length > 0) {
                let newItem;

                if (isFirstRun) {
                    newItem = rssData[0];
                    sendMessageToTelegram(`[${newItem.title}](${newItem.link})`);
                    previousItems.push(newItem.guid);
                    rssData.shift();
                    isFirstRun = false;
                } else {
                    let randomIndex;
                    do {
                        randomIndex = Math.floor(Math.random() * rssData.length);
                        newItem = rssData[randomIndex];
                    } while (previousItems.includes(newItem.guid));

                    sendMessageToTelegram(`[${newItem.title}](${newItem.link})`);
                    previousItems.push(newItem.guid);
                    rssData.splice(randomIndex, 1);
                }
            }
        }

        function startRssFeed() {
            const rssUrl = document.getElementById('rssUrlInput').value.trim();
            const intervalSeconds = parseInt(document.getElementById('intervalSelect').value);

            if (rssUrl && !isNaN(intervalSeconds) && intervalSeconds > 0) {
                if (rssInterval) {
                    clearInterval(rssInterval);
                }
                previousItems = [];
                remainingSeconds = intervalSeconds;
                updateCountdown();
                rssInterval = setInterval(() => {
                    fetchRssFeed(rssUrl);
                    remainingSeconds = intervalSeconds;
                }, intervalSeconds * 1000);
                countdownInterval = setInterval(updateCountdown, 1000);
                const listItem = document.createElement('li');
                const now = new Date();
                const time = now.toLocaleString();
                listItem.textContent = `[${time}] RSS 피드 전송 시작: ${rssUrl} (${intervalSeconds}초 간격)`;
                messageList.appendChild(listItem);
                isFirstRun = true;
            } else {
                alert('RSS URL 및 유효한 전송 간격을 선택하세요.');
            }
        }

        function stopRssFeed() {
            if (rssInterval) {
                clearInterval(rssInterval);
                rssInterval = null;
                clearInterval(countdownInterval);
                document.getElementById('countdown').textContent = '';
                const listItem = document.createElement('li');
                const now = new Date();
                const time = now.toLocaleString();
                listItem.textContent = `[${time}] RSS 피드 전송 중지`;
                messageList.appendChild(listItem);
            }
        }

        function updateCountdown() {
            if (remainingSeconds > 0) {
                document.getElementById('countdown').textContent = `다음 전송까지 ${remainingSeconds}초 남음`;
                remainingSeconds--;
            } else {
                document.getElementById('countdown').textContent = '전송 중...';
            }
        }
    </script>
</body>
</html>
