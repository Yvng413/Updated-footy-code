<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>FootyAI - Dual Analyst Chat Shell</title>
  <style>
    /* 
       1. RESET & BASE STYLES
     */
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }

    body {
      background-color: #0f172a;
      color: #f8fafc;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }

    /* 
       2. WELCOME OVERLAY & MODAL
     */
    .welcome-overlay {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(15, 23, 42, 0.85);
      backdrop-filter: blur(8px);
      display: flex;
      justify-content: center;
      align-items: center;
      z-index: 100;
      padding: 20px;
    }

    .welcome-card {
      background-color: #1e293b;
      border: 1px solid #334155;
      border-radius: 16px;
      padding: 24px;
      max-width: 400px;
      width: 100%;
      text-align: center;
      box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.5);
    }

    .welcome-card h1 {
      font-size: 22px;
      margin-bottom: 8px;
      color: #f8fafc;
    }

    .welcome-card p.subtitle {
      font-size: 13px;
      color: #94a3b8;
      margin-bottom: 20px;
      line-height: 1.5;
    }

    .analyst-options {
      display: flex;
      flex-direction: column;
      gap: 12px;
      margin-bottom: 20px;
    }

    .analyst-btn {
      display: flex;
      align-items: center;
      padding: 12px 16px;
      background: #0f172a;
      border: 2px solid #334155;
      border-radius: 12px;
      cursor: pointer;
      transition: all 0.2s ease;
      text-align: left;
    }

    .analyst-btn:hover, .analyst-btn.active {
      border-color: #2563eb;
      background: #1e293b;
    }

    .analyst-btn-info {
      margin-left: 12px;
    }

    .analyst-btn-info h4 {
      font-size: 14px;
      color: #f8fafc;
    }

    .analyst-btn-info p {
      font-size: 11px;
      color: #94a3b8;
    }

    .start-btn {
      width: 100%;
      padding: 12px;
      background: #2563eb;
      color: #ffffff;
      border: none;
      border-radius: 10px;
      font-weight: bold;
      font-size: 15px;
      cursor: pointer;
    }

    /* 
       3. MAIN CHAT CONTAINER
    */
    .chat-container {
      width: 100%;
      max-width: 480px;
      height: 90vh;
      background-color: #1e293b;
      border-radius: 12px;
      display: flex;
      flex-direction: column;
      overflow: hidden;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
    }

    /* 
       4. HEADER SECTION
    */
    .chat-header {
      background-color: #0f172a;
      padding: 15px;
      display: flex;
      align-items: center;
      gap: 12px;
      border-bottom: 1px solid #334155;
      z-index: 10;
    }

    .bot-avatars-header {
      display: flex;
      position: relative;
    }

    .header-info h3 {
      font-size: 16px;
      color: #f8fafc;
    }

    .header-info p {
      font-size: 12px;
      color: #94a3b8;
    }

    /* 
       5. AVATAR STYLING & CANVASES
    */
    .avatar-wrapper {
      position: relative;
      width: 44px;
      height: 44px;
      display: flex;
      align-items: center;
      justify-content: center;
      flex-shrink: 0;
    }

    .wave-canvas {
      position: absolute;
      top: -10px;
      left: -10px;
      width: 64px;
      height: 64px;
      pointer-events: none;
      z-index: 1;
    }

    .avatar {
      width: 32px;
      height: 32px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: bold;
      font-size: 13px;
      position: relative;
      z-index: 2;
    }

    .avatar-tim {
      background-color: #0284c7;
      color: #ffffff;
      box-shadow: 0 0 10px rgba(56, 189, 248, 0.6);
    }

    .avatar-sam {
      background-color: #16a34a;
      color: #ffffff;
      box-shadow: 0 0 10px rgba(59, 130, 246, 0.6);
    }

    /* 
       6. CHAT MESSAGES BODY
    */
    .chat-messages-wrapper {
      position: relative;
      flex: 1;
      background-color: #0b1329;
      overflow: hidden;
    }

    #bg-wave-canvas {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      pointer-events: none;
      z-index: 1;
      opacity: 0.6;
    }

    .chat-messages {
      position: relative;
      z-index: 2;
      height: 100%;
      padding: 15px;
      overflow-y: auto;
      display: flex;
      flex-direction: column;
      gap: 16px;
    }

    .message-row {
      display: flex;
      align-items: flex-start;
      gap: 10px;
      max-width: 85%;
    }

    .message-row.user-row {
      align-self: flex-end;
      justify-content: flex-end;
    }

    .message-row.bot-row {
      align-self: flex-start;
    }

    .message {
      padding: 10px 14px;
      border-radius: 12px;
      font-size: 14px;
      line-height: 1.4;
      word-wrap: break-word;
      backdrop-filter: blur(4px);
    }

    .message-bot-tim {
      background-color: rgba(30, 41, 59, 0.85);
      border-left: 3px solid #ff4d4d;
      color: #e2e8f0;
    }

    .message-bot-sam {
      background-color: rgba(30, 41, 59, 0.85);
      border-left: 3px solid #4d94ff;
      color: #e2e8f0;
    }

    .message-user-msg {
      background-color: #2563eb;
      color: #ffffff;
      border-bottom-right-radius: 2px;
    }

    .sender-name {
      font-size: 11px;
      font-weight: bold;
      margin-bottom: 4px;
    }

    .tim-name { color: #ff6b6b; }
    .sam-name { color: #4d94ff; }

    /* 
       7. TYPING INDICATOR & FOOTER INPUT
    */
    .typing-status {
      font-size: 12px;
      color: #94a3b8;
      font-style: italic;
      padding: 4px 15px;
      height: 24px;
      background-color: #0b1329;
      position: relative;
      z-index: 10;
    }

    .chat-input-area {
      position: relative;
      z-index: 10;
      background-color: #0f172a;
      padding: 12px;
      display: flex;
      gap: 8px;
      border-top: 1px solid #334155;
    }

    .chat-input-area input {
      flex: 1;
      padding: 10px 14px;
      border-radius: 20px;
      border: 1px solid #334155;
      background-color: #1e293b;
      color: #ffffff;
      outline: none;
    }

    .chat-input-area input:disabled {
      background-color: #0f172a;
      cursor: not-allowed;
      opacity: 0.6;
    }

    .chat-input-area button {
      padding: 10px 18px;
      border-radius: 20px;
      border: none;
      background-color: #2563eb;
      color: #ffffff;
      font-weight: bold;
      cursor: pointer;
    }

    .chat-input-area button:disabled {
      background-color: #334155;
      cursor: not-allowed;
    }
  </style>
</head>
<body>

  <!-- Welcome Overlay Modal -->
  <div class="welcome-overlay" id="welcome-overlay">
    <div class="welcome-card">
      <h1>Welcome to FootyAI ⚽</h1>
      <p class="subtitle">Your real-time football intelligence companion. Select your preferred analyst persona to get started.</p>

      <div class="analyst-options">
        <div class="analyst-btn active" onclick="selectPersona('both', this)">
          <div style="display: flex;">
            <div class="avatar avatar-tim" style="width: 24px; height: 24px; font-size: 10px;">T</div>
            <div class="avatar avatar-sam" style="width: 24px; height: 24px; font-size: 10px; margin-left: -6px;">S</div>
          </div>
          <div class="analyst-btn-info">
            <h4>Dual Analysis (Both)</h4>
            <p>Get tactical breakdowns & statistical data</p>
          </div>
        </div>

        <div class="analyst-btn" onclick="selectPersona('tim', this)">
          <div class="avatar avatar-tim" style="width: 24px; height: 24px; font-size: 10px;">T</div>
          <div class="analyst-btn-info">
            <h4>Tactical Tim Only</h4>
            <p>Focus on formations, space & manager strategy</p>
          </div>
        </div>

        <div class="analyst-btn" onclick="selectPersona('sam', this)">
          <div class="avatar avatar-sam" style="width: 24px; height: 24px; font-size: 10px;">S</div>
          <div class="analyst-btn-info">
            <h4>Stats Sam Only</h4>
            <p>Focus on xG, betting odds & match metrics</p>
          </div>
        </div>
      </div>

      <button class="start-btn" onclick="startSession()">Enter Match Chat</button>
    </div>
  </div>

  <!-- Main Chat Container -->
  <div class="chat-container">

    <!-- Top Header -->
    <div class="chat-header">
      <div class="bot-avatars-header" id="header-avatars">
        <div class="avatar-wrapper" style="width: 34px; height: 34px;">
          <div class="avatar avatar-tim" id="header-tim" style="width: 28px; height: 28px; font-size: 11px;">T</div>
        </div>
        <div class="avatar-wrapper" style="width: 34px; height: 34px; margin-left: -8px;">
          <div class="avatar avatar-sam" id="header-sam" style="width: 28px; height: 28px; font-size: 11px;">S</div>
        </div>
      </div>
      <div class="header-info">
        <h3 id="header-title">Tactical Tim & Stats Sam</h3>
        <p id="status-text">Analysing upcoming match...</p>
      </div>
    </div>

    <!-- Chat Messages Wrapper with Background Wave Canvas -->
    <div class="chat-messages-wrapper" id="chat-wrapper">
      <canvas id="bg-wave-canvas"></canvas>
      <div class="chat-messages" id="chat-messages"></div>
    </div>

    <!-- Typing Indicator -->
    <div class="typing-status" id="typing-status"></div>

    <!-- User Input Footer Area -->
    <div class="chat-input-area">
      <input type="text" id="user-input" placeholder="Ask a question..." disabled>
      <button id="send-btn" onclick="sendMessage()" disabled>Send</button>
    </div>

  </div>

  <script>
    let activePersona = 'both';

    const chatMessages = document.getElementById('chat-messages');
    const userInput = document.getElementById('user-input');
    const sendBtn = document.getElementById('send-btn');
    const typingStatus = document.getElementById('typing-status');
    const statusText = document.getElementById('status-text');

    // Your API Key
    const API_KEY = "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";

    function selectPersona(mode, el) {
      activePersona = mode;
      document.querySelectorAll('.analyst-btn').forEach(btn => btn.classList.remove('active'));
      el.classList.add('active');
    }

    function startSession() {
      document.getElementById('welcome-overlay').style.display = 'none';

      const headerTim = document.getElementById('header-tim');
      const headerSam = document.getElementById('header-sam');
      const headerTitle = document.getElementById('header-title');

      if (activePersona === 'tim') {
        headerSam.parentElement.style.display = 'none';
        headerTitle.textContent = 'Tactical Tim';
      } else if (activePersona === 'sam') {
        headerTim.parentElement.style.display = 'none';
        headerSam.parentElement.style.marginLeft = '0';
        headerTitle.textContent = 'Stats Sam';
      } else {
        headerTim.parentElement.style.display = 'flex';
        headerSam.parentElement.style.display = 'flex';
        headerTitle.textContent = 'Tactical Tim & Stats Sam';
      }

      statusText.textContent = 'Analysis active. Ask a question!';
      userInput.placeholder = `Ask ${activePersona === 'both' ? 'Tim & Sam' : activePersona === 'tim' ? 'Tim' : 'Sam'} a question...`;
      userInput.disabled = false;
      sendBtn.disabled = false;
    }

    /* 
       8. BLUE FLUID WAVING GLOW ENGINE (AVATAR SURROUNDING)
    */
    function attachFluidWaveCanvas(container) {
      const canvas = document.createElement('canvas');
      canvas.className = 'wave-canvas';
      canvas.width = 64;
      canvas.height = 64;
      container.appendChild(canvas);

      const ctx = canvas.getContext('2d');
      let step = 0;

      function render() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        const centerX = canvas.width / 2;
        const centerY = canvas.height / 2;
        const baseRadius = 17;

        step += 0.05;

        ctx.save();
        ctx.translate(centerX, centerY);

        for (let waveIndex = 0; waveIndex < 3; waveIndex++) {
          ctx.beginPath();
          const points = 50;

          for (let i = 0; i <= points; i++) {
            const angle = (i / points) * Math.PI * 2;
            const waveOffset = Math.sin(angle * 3 + step + waveIndex * 1.4) * 3.5 +
                               Math.cos(angle * 2 - step * 0.7) * 2;

            const r = baseRadius + waveOffset;
            const x = r * Math.cos(angle);
            const y = r * Math.sin(angle);

            if (i === 0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
          }

          ctx.closePath();

          ctx.strokeStyle = waveIndex === 0 ? 'rgba(56, 189, 248, 0.9)' :
                            waveIndex === 1 ? 'rgba(96, 165, 250, 0.6)' :
                            'rgba(147, 197, 253, 0.3)';
          ctx.shadowColor = '#38bdf8';
          ctx.shadowBlur = 8;
          ctx.lineWidth = 2 - waveIndex * 0.4;
          ctx.stroke();
        }

        ctx.restore();
        requestAnimationFrame(render);
      }

      render();
    }

    /* 
       9. BACKGROUND BLUE GEMINI LIVE WAVING MOTION
    */
    function initBackgroundWave() {
      const canvas = document.getElementById('bg-wave-canvas');
      const wrapper = document.getElementById('chat-wrapper');
      if (!canvas || !wrapper) return;

      const ctx = canvas.getContext('2d');
      let step = 0;

      function resize() {
        canvas.width = wrapper.offsetWidth || 300;
        canvas.height = wrapper.offsetHeight || 400;
      }

      function renderBg() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        step += 0.02;

        const width = canvas.width;
        const height = canvas.height;
        const centerY = height * 0.55;

        for (let wave = 0; wave < 3; wave++) {
          ctx.beginPath();
          ctx.moveTo(0, height);

          for (let x = 0; x <= width; x += 5) {
            const y = centerY +
              Math.sin(x * 0.01 + step + wave * 1.2) * 25 +
              Math.cos(x * 0.02 - step * 0.8) * 15;
            ctx.lineTo(x, y);
          }

          ctx.lineTo(width, height);
          ctx.closePath();

          const grad = ctx.createLinearGradient(0, centerY - 40, 0, height);
          if (wave === 0) {
            grad.addColorStop(0, 'rgba(56, 189, 248, 0.25)');
            grad.addColorStop(1, 'rgba(15, 23, 42, 0)');
          } else if (wave === 1) {
            grad.addColorStop(0, 'rgba(37, 99, 235, 0.2)');
            grad.addColorStop(1, 'rgba(15, 23, 42, 0)');
          } else {
            grad.addColorStop(0, 'rgba(147, 197, 253, 0.15)');
            grad.addColorStop(1, 'rgba(15, 23, 42, 0)');
          }

          ctx.fillStyle = grad;
          ctx.fill();
        }

        requestAnimationFrame(renderBg);
      }

      resize();
      window.addEventListener('resize', resize);
      renderBg();
    }

    function appendMessage(sender, name, text) {
      const rowDiv = document.createElement('div');

      if (sender === 'tim') {
        rowDiv.className = 'message-row bot-row';
        rowDiv.innerHTML = `
          <div class="avatar-wrapper">
            <div class="avatar avatar-tim">T</div>
          </div>
          <div class="message message-bot-tim">
            <div class="sender-name tim-name">${name}</div>
            ${text}
          </div>
        `;
        chatMessages.appendChild(rowDiv);
        const wrapper = rowDiv.querySelector('.avatar-wrapper');
        attachFluidWaveCanvas(wrapper);

      } else if (sender === 'sam') {
        rowDiv.className = 'message-row bot-row';
        rowDiv.innerHTML = `
          <div class="avatar-wrapper">
            <div class="avatar avatar-sam">S</div>
          </div>
          <div class="message message-bot-sam">
            <div class="sender-name sam-name">${name}</div>
            ${text}
          </div>
        `;
        chatMessages.appendChild(rowDiv);
        const wrapper = rowDiv.querySelector('.avatar-wrapper');
        attachFluidWaveCanvas(wrapper);

      } else {
        rowDiv.className = 'message-row user-row';
        rowDiv.innerHTML = `
          <div class="message message-user-msg">
            ${text}
          </div>
        `;
        chatMessages.appendChild(rowDiv);
      }

      chatMessages.scrollTop = chatMessages.scrollHeight;
    }

    /* 
       10. REAL BACKEND API INTEGRATION 
    */
    async function sendMessage() {
      const text = userInput.value.trim();
      if (text === '') return;

      appendMessage('user', 'You', text);
      userInput.value = '';

      typingStatus.textContent = 'Thinking...';
      sendBtn.disabled = true;
      userInput.disabled = true;

      try {
        const response = await fetch('https://api.openai.com/v1/chat/completions', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${API_KEY}`
          },
          body: JSON.stringify({
            model: 'gpt-4o-mini',
            messages: [
              {
                role: 'system',
                content: `You are a football expert acting under the analyst mode: ${activePersona}. Tactical Tim focuses on team formations, tactics, and strategies. Stats Sam focuses on data, metrics, and probability.`
              },
              { role: 'user', content: text }
            ]
          })
        });

        const data = await response.json();

        if (!response.ok) {
          throw new Error(data.error?.message || 'API Request failed');
        }

        const replyText = data.choices?.[0]?.message?.content || 'No response returned.';
        typingStatus.textContent = '';

        // Appending response depending on the active persona view
        if (activePersona === 'sam') {
          appendMessage('sam', 'Stats Sam', replyText);
        } else if (activePersona === 'tim') {
          appendMessage('tim', 'Tactical Tim', replyText);
        } else {
          appendMessage('tim', 'Tactical Tim & Stats Sam', replyText);
        }

      } catch (error) {
        console.error(error);
        typingStatus.textContent = '';
        appendMessage('tim', 'System', `Error: ${error.message}`);
      } finally {
        sendBtn.disabled = false;
        userInput.disabled = false;
        userInput.focus();
      }
    }

    userInput.addEventListener('keypress', function(e) {
      if (e.key === 'Enter') sendMessage();
    });

    document.addEventListener("DOMContentLoaded", function () {
      initBackgroundWave();
    });
  </script>
</body>
</html>
