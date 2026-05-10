## Client-Side Code

### 1. `index.html` (HTML Structure)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Real-Time Chat</title>
    <link rel="stylesheet" href="/style.css">
</head>
<body>
    <div id="login-container" class="container">
        <h1>Join Chat</h1>
        <input type="text" id="username-input" placeholder="Enter your username" required>
        <button id="join-btn">Join</button>
    </div>

    <div id="app-container" class="container hidden">
        <div class="sidebar">
            <div class="user-info">
                <span id="display-username"></span>
            </div>
            <div class="rooms-section">
                <h2>Rooms</h2>
                <ul id="rooms-list"></ul>
            </div>
            <div class="users-section">
                <h2>Online Users</h2>
                <ul id="users-list"></ul>
            </div>
        </div>

        <div class="chat-area">
            <div class="chat-header">
                <span id="current-room-name">General</span>
                <div id="private-mode-indicator" class="hidden">
                    <button id="back-to-room-btn">← Back to Room</button>
                    <span id="private-target-user"></span>
                </div>
            </div>
            <div id="messages-display" class="messages-display"></div>
            <div class="message-input-area">
                <input type="text" id="message-input" placeholder="Type a message...">
                <button id="send-btn">Send</button>
            </div>
        </div>
    </div>

    <script src="/socket.io/socket.io.js"></script>
    <script src="/script.js"></script>
</body>
</html>
```

### 2. `style.css` (CSS Styling)

```css
:root {
    --primary-color: #4CAF50;
    --secondary-color: #388E3C;
    --background-color: #f4f7f6;
    --card-background: #ffffff;
    --text-color: #333;
    --light-text-color: #666;
    --border-color: #ddd;
    --system-message-color: #888;
    --sent-message-bg: #DCF8C6;
    --received-message-bg: #E0E0E0;
    --private-sent-message-bg: #FFE0B2; /* Light orange for private sent */
    --private-received-message-bg: #FFCC80; /* Slightly darker orange for private received */
    --notification-badge-bg: #F44336;
    --notification-badge-text: #fff;
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    margin: 0;
    padding: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background-color: var(--background-color);
    color: var(--text-color);
}

.container {
    background-color: var(--card-background);
    border-radius: 10px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    padding: 25px;
    width: 90%;
    max-width: 1000px;
    display: flex;
    flex-direction: column;
    gap: 20px;
    box-sizing: border-box;
    height: 90vh;
}

#login-container {
    justify-content: center;
    align-items: center;
    text-align: center;
    height: auto;
    min-height: 300px;
}

#login-container h1 {
    color: var(--primary-color);
    margin-bottom: 20px;
}

#login-container input[type="text"] {
    width: calc(100% - 20px);
    padding: 12px;
    margin-bottom: 15px;
    border: 1px solid var(--border-color);
    border-radius: 5px;
    font-size: 1rem;
}

#login-container button {
    background-color: var(--primary-color);
    color: white;
    padding: 12px 25px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1rem;
    transition: background-color 0.3s ease;
}

#login-container button:hover {
    background-color: var(--secondary-color);
}

#app-container {
    flex-direction: row;
    padding: 0;
    overflow: hidden;
}

.hidden {
    display: none !important;
}

.sidebar {
    width: 250px;
    background-color: #f0f0f0;
    border-right: 1px solid var(--border-color);
    display: flex;
    flex-direction: column;
    padding: 15px;
    gap: 15px;
}

.user-info {
    padding-bottom: 10px;
    border-bottom: 1px solid var(--border-color);
    font-weight: bold;
    color: var(--primary-color);
    font-size: 1.1rem;
}

.sidebar h2 {
    font-size: 1rem;
    color: var(--light-text-color);
    margin-top: 0;
    margin-bottom: 10px;
    border-bottom: 1px solid var(--border-color);
    padding-bottom: 5px;
}

.sidebar ul {
    list-style: none;
    padding: 0;
    margin: 0;
    flex-grow: 1;
    overflow-y: auto;
}

.sidebar li {
    padding: 8px 10px;
    cursor: pointer;
    border-radius: 5px;
    margin-bottom: 5px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    transition: background-color 0.2s ease;
}

.sidebar li:hover {
    background-color: #e9e9e9;
}

.sidebar li.active, .sidebar li.private-selected {
    background-color: var(--primary-color);
    color: white;
}

.sidebar li.active:hover, .sidebar li.private-selected:hover {
    background-color: var(--secondary-color);
}

.notification-badge {
    background-color: var(--notification-badge-bg);
    color: var(--notification-badge-text);
    border-radius: 50%;
    padding: 2px 7px;
    font-size: 0.75rem;
    min-width: 10px;
    text-align: center;
    line-height: 1;
}

.chat-area {
    flex-grow: 1;
    display: flex;
    flex-direction: column;
    background-color: var(--card-background);
}

.chat-header {
    padding: 15px 20px;
    border-bottom: 1px solid var(--border-color);
    font-size: 1.2rem;
    font-weight: bold;
    color: var(--primary-color);
    display: flex;
    align-items: center;
    gap: 10px;
}

#private-mode-indicator {
    display: flex;
    align-items: center;
    gap: 10px;
    background-color: var(--private-received-message-bg);
    padding: 5px 10px;
    border-radius: 5px;
    color: var(--text-color);
    font-size: 0.9rem;
    font-weight: normal;
}

#private-mode-indicator button {
    background: none;
    border: 1px solid var(--text-color);
    color: var(--text-color);
    padding: 3px 8px;
    border-radius: 5px;
    cursor: pointer;
    font-size: 0.8rem;
    transition: background-color 0.2s ease;
}

#private-mode-indicator button:hover {
    background-color: rgba(0, 0, 0, 0.1);
}

.messages-display {
    flex-grow: 1;
    padding: 20px;
    overflow-y: auto;
    background-color: var(--background-color);
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.message {
    display: flex;
    flex-direction: column;
    max-width: 70%;
    padding: 10px 15px;
    border-radius: 15px;
    word-wrap: break-word;
}

.message.sent {
    align-self: flex-end;
    background-color: var(--sent-message-bg);
    border-bottom-right-radius: 2px;
}

.message.received {
    align-self: flex-start;
    background-color: var(--received-message-bg);
    border-bottom-left-radius: 2px;
}

.message.private-sent {
    align-self: flex-end;
    background-color: var(--private-sent-message-bg);
    border: 1px solid #FF9800; /* Orange border for private messages */
    border-bottom-right-radius: 2px;
}

.message.private-received {
    align-self: flex-start;
    background-color: var(--private-received-message-bg);
    border: 1px solid #FF9800; /* Orange border for private messages */
    border-bottom-left-radius: 2px;
}

.msg-meta {
    font-size: 0.75rem;
    color: var(--light-text-color);
    margin-bottom: 5px;
}

.msg-text {
    font-size: 0.95rem;
    color: var(--text-color);
}

.system-msg {
    text-align: center;
    font-style: italic;
    color: var(--system-message-color);
    font-size: 0.85rem;
    margin: 10px 0;
}

.message-input-area {
    display: flex;
    padding: 15px 20px;
    border-top: 1px solid var(--border-color);
    gap: 10px;
}

.message-input-area input[type="text"] {
    flex-grow: 1;
    padding: 10px;
    border: 1px solid var(--border-color);
    border-radius: 20px;
    font-size: 1rem;
}

.message-input-area button {
    background-color: var(--primary-color);
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 20px;
    cursor: pointer;
    font-size: 1rem;
    transition: background-color 0.3s ease;
}

.message-input-area button:hover {
    background-color: var(--secondary-color);
}

/* Responsive adjustments */
@media (max-width: 768px) {
    .container {
        width: 100%;
        height: 100vh;
        border-radius: 0;
        box-shadow: none;
    }

    #app-container {
        flex-direction: column;
    }

    .sidebar {
        width: 100%;
        height: 200px;
        border-right: none;
        border-bottom: 1px solid var(--border-color);
        flex-direction: row;
        overflow-x: auto;
        padding: 10px;
    }

    .sidebar .rooms-section, .sidebar .users-section {
        flex-shrink: 0;
        width: 50%;
        padding: 0 10px;
    }

    .sidebar h2 {
        display: none;
    }

    .sidebar ul {
        display: flex;
        flex-direction: row;
        gap: 10px;
    }

    .sidebar li {
        flex-shrink: 0;
        white-space: nowrap;
    }

    .chat-area {
        flex-grow: 1;
    }
}
```

### 3. `script.js` (JavaScript Logic)

```javascript
const socket = io();

// DOM Elements
const loginContainer = document.getElementById('login-container');
const appContainer = document.getElementById('app-container');
const usernameInput = document.getElementById('username-input');
const joinBtn = document.getElementById('join-btn');
const displayUsername = document.getElementById('display-username');
const roomsList = document.getElementById('rooms-list');
const usersList = document.getElementById('users-list');
const messagesDisplay = document.getElementById('messages-display');
const messageInput = document.getElementById('message-input');
const sendBtn = document.getElementById('send-btn');
const currentRoomName = document.getElementById('current-room-name');
const privateModeIndicator = document.getElementById('private-mode-indicator');
const privateTargetUser = document.getElementById('private-target-user');
const backToRoomBtn = document.getElementById('back-to-room-btn');

let myUsername = '';
let mySessionId = '';
let currentRoom = 'General';
let privateMessageTarget = null; // null = room mode, username = private mode
let privateMessageTargetSessionId = null; // Session ID of the private message target
let unreadMessages = new Map(); // username-sessionId -> count of unread messages
let unreadRooms = new Map(); // roomName -> count of unread messages

// --- Login Logic ---
joinBtn.addEventListener('click', () => {
    const username = usernameInput.value.trim();
    if (username) {
        socket.emit('set-username', username);
    }
});

usernameInput.addEventListener('keypress', (e) => {
    if (e.key === 'Enter') joinBtn.click();
});

// --- Socket Events ---

socket.on('init-data', (data) => {
    myUsername = data.username;
    mySessionId = data.allUsers.find(u => u.username === data.username)?.sessionId || '';
    currentRoom = data.currentRoom;
    
    displayUsername.textContent = `${myUsername} (${mySessionId.substring(0, 8)}...)`;
    loginContainer.classList.add('hidden');
    appContainer.classList.remove('hidden');
    
    renderRooms(data.rooms);
    renderUsers(data.allUsers);
    
    // Load and display room history
    if (data.roomHistory && data.roomHistory.length > 0) {
        data.roomHistory.forEach(msg => {
            appendMessage(msg);
        });
    }
    
    addSystemMessage(`Welcome ${myUsername}! You joined ${currentRoom}.`);
});

socket.on('update-user-list', (users) => {
    renderUsers(users);
});

socket.on('room-switched', (data) => {
    currentRoom = data.room;
    currentRoomName.textContent = currentRoom;
    messagesDisplay.innerHTML = ''; // Clear chat for new room
    
    // Clear unread count for this room
    unreadRooms.delete(currentRoom);
    updateRoomNotifications();
    
    // Load and display room history
    if (data.history && data.history.length > 0) {
        data.history.forEach(msg => {
            appendMessage(msg);
        });
    }
    
    addSystemMessage(`You joined room: ${currentRoom}`);
    renderRoomsActiveState();
    
    // Exit private mode when switching rooms
    if (privateMessageTarget) {
        exitPrivateMode(false); 
    }
});

socket.on('receive-message', (msg) => {
    if (msg.type === 'private') {
        // --- Private Message Handling ---
        if ((privateMessageTarget === msg.from && privateMessageTargetSessionId === msg.sessionId) || 
            (msg.from === myUsername && msg.sessionId === mySessionId)) {
            appendMessage(msg);
            if (msg.from !== myUsername) {
                const key = `${msg.from}-${msg.sessionId}`;
                unreadMessages.delete(key);
                updateUserNotifications();
            }
        } else {
            // Unread private message
            const key = `${msg.from}-${msg.sessionId}`;
            const currentCount = unreadMessages.get(key) || 0;
            unreadMessages.set(key, currentCount + 1);
            updateUserNotifications();
            playNotificationSound();
        }
    } else {
        // --- Room Message Handling ---
        // 1. If we are in the same room AND NOT in private mode
        if (!privateMessageTarget && currentRoom === msg.room) {
            appendMessage(msg);
        } 
        // 2. If we are in a different room OR in private mode
        else {
            if (msg.from !== myUsername) {
                const currentCount = unreadRooms.get(msg.room) || 0;
                unreadRooms.set(msg.room, currentCount + 1);
                updateRoomNotifications();
                playNotificationSound();
            }
        }
    }
});

socket.on('private-history-loaded', (data) => {
    messagesDisplay.innerHTML = '';
    if (data.history && data.history.length > 0) {
        data.history.forEach(msg => {
            appendMessage(msg);
        });
    }
    const key = `${data.targetUsername}-${privateMessageTargetSessionId}`;
    unreadMessages.delete(key);
    updateUserNotifications();
});

socket.on('user-joined', (data) => {
    addSystemMessage(`${data.username} (${data.sessionId.substring(0, 8)}...) joined the room.`);
});

socket.on('user-left', (data) => {
    addSystemMessage(`${data.username} (${data.sessionId.substring(0, 8)}...) left the room.`);
});

socket.on('error-msg', (msg) => {
    alert(msg);
});

// --- UI Rendering Functions ---

function renderRooms(rooms) {
    roomsList.innerHTML = '';
    rooms.forEach(room => {
        const li = document.createElement('li');
        li.dataset.room = room;
        
        const roomSpan = document.createElement('span');
        roomSpan.textContent = room;
        li.appendChild(roomSpan);
        
        if (room === currentRoom && !privateMessageTarget) li.classList.add('active');
        
        li.addEventListener('click', () => {
            if (room !== currentRoom || privateMessageTarget) {
                socket.emit('join-room', room);
            }
        });
        roomsList.appendChild(li);
    });
    updateRoomNotifications();
}

function renderRoomsActiveState() {
    document.querySelectorAll('#rooms-list li').forEach(li => {
        li.classList.toggle('active', li.dataset.room === currentRoom && !privateMessageTarget);
    });
}

function updateRoomNotifications() {
    const roomElements = document.querySelectorAll('#rooms-list li');
    roomElements.forEach(li => {
        const roomName = li.dataset.room;
        
        const oldBadge = li.querySelector('.notification-badge');
        if (oldBadge) oldBadge.remove();
        
        if (unreadRooms.has(roomName)) {
            const badge = document.createElement('span');
            badge.classList.add('notification-badge');
            badge.textContent = unreadRooms.get(roomName);
            li.appendChild(badge);
        }
    });
}

function renderUsers(users) {
    usersList.innerHTML = '';
    users.forEach(user => {
        const li = document.createElement('li');
        const userSpan = document.createElement('span');
        const displayText = user.username === myUsername ? 
            `${user.username} (You)` : 
            `${user.username} (${user.sessionId.substring(0, 8)}...)`;
        userSpan.textContent = displayText;
        li.appendChild(userSpan);
        
        if (user.username !== myUsername || user.sessionId !== mySessionId) {
            li.title = 'Click to send private message';
            li.addEventListener('click', () => {
                enterPrivateMode(user.username, user.sessionId);
            });
            if (user.username === privateMessageTarget && user.sessionId === privateMessageTargetSessionId) {
                li.classList.add('private-selected');
            }
        }
        usersList.appendChild(li);
    });
    updateUserNotifications();
}

function updateUserNotifications() {
    const userElements = document.querySelectorAll('#users-list li');
    userElements.forEach(li => {
        const userSpan = li.querySelector('span');
        if (!userSpan) return;
        
        const oldBadge = li.querySelector('.notification-badge');
        if (oldBadge) oldBadge.remove();
        
        // Check all unread messages for this user
        for (let [key, count] of unreadMessages.entries()) {
            const [username, sessionId] = key.split('-');
            const displayText = `${username} (${sessionId.substring(0, 8)}...)`;
            if (userSpan.textContent.includes(displayText) || userSpan.textContent.includes(username)) {
                const badge = document.createElement('span');
                badge.classList.add('notification-badge');
                badge.textContent = count;
                li.appendChild(badge);
                break;
            }
        }
    });
}

function enterPrivateMode(targetUsername, targetSessionId) {
    privateMessageTarget = targetUsername;
    privateMessageTargetSessionId = targetSessionId;
    privateModeIndicator.classList.remove('hidden');
    privateTargetUser.textContent = `with ${targetUsername} (${targetSessionId.substring(0, 8)}...)`;
    messageInput.placeholder = `Private message to ${targetUsername}...`;
    
    document.querySelectorAll('#users-list li').forEach(li => {
        const span = li.querySelector('span');
        if (span && span.textContent.includes(targetUsername) && span.textContent.includes(targetSessionId.substring(0, 8))) {
            li.classList.add('private-selected');
        } else {
            li.classList.remove('private-selected');
        }
    });
    
    document.querySelectorAll('#rooms-list li').forEach(li => {
        li.classList.remove('active');
    });
    
    socket.emit('load-private-history', targetUsername);
    addSystemMessage(`Now chatting privately with ${targetUsername} (${targetSessionId.substring(0, 8)}...)`);
}

function exitPrivateMode(clearDisplay = true) {
    privateMessageTarget = null;
    privateMessageTargetSessionId = null;
    if (clearDisplay) {
        messagesDisplay.innerHTML = '';
        socket.emit('join-room', currentRoom);
    }
    
    privateModeIndicator.classList.add('hidden');
    messageInput.placeholder = 'Type a message...';
    
    unreadRooms.delete(currentRoom);
    updateRoomNotifications();
    
    document.querySelectorAll('#users-list li').forEach(li => {
        li.classList.remove('private-selected');
    });
    
    renderRoomsActiveState();
}

backToRoomBtn.addEventListener('click', () => {
    exitPrivateMode();
});

function appendMessage(msg) {
    const div = document.createElement('div');
    div.classList.add('message');
    const isSentByMe = msg.from === myUsername && msg.sessionId === mySessionId;
    
    if (msg.type === 'private') {
        div.classList.add(isSentByMe ? 'private-sent' : 'private-received');
    } else {
        div.classList.add(isSentByMe ? 'sent' : 'received');
    }
    
    const senderDisplay = isSentByMe ? 'You' : `${msg.from} (${msg.sessionId.substring(0, 8)}...)`;
    div.innerHTML = `<div class="msg-meta">${senderDisplay} • ${msg.time}</div>
                     <div class="msg-text">${msg.text}</div>`;
    
    messagesDisplay.appendChild(div);
    messagesDisplay.scrollTop = messagesDisplay.scrollHeight;
}

function addSystemMessage(text) {
    const div = document.createElement('div');
    div.classList.add('system-msg');
    div.textContent = text;
    messagesDisplay.appendChild(div);
    messagesDisplay.scrollTop = messagesDisplay.scrollHeight;
}

function playNotificationSound() {
    try {
        const audioContext = new (window.AudioContext || window.webkitAudioContext)();
        const oscillator = audioContext.createOscillator();
        const gain = audioContext.createGain();
        oscillator.connect(gain);
        gain.connect(audioContext.destination);
        oscillator.frequency.value = 800;
        oscillator.type = 'sine';
        gain.gain.setValueAtTime(0.3, audioContext.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.1);
        oscillator.start(audioContext.currentTime);
        oscillator.stop(audioContext.currentTime + 0.1);
    } catch(e) { console.log('Audio error:', e); }
}

sendBtn.addEventListener('click', () => {
    const text = messageInput.value.trim();
    if (text) {
        if (privateMessageTarget) {
            socket.emit('private-message', { to: privateMessageTarget, text });
        } else {
            socket.emit('send-message', { text });
        }
        messageInput.value = '';
        messageInput.focus();
    }
});

messageInput.addEventListener('keypress', (e) => {
    if (e.key === 'Enter') sendBtn.click();
});
```
