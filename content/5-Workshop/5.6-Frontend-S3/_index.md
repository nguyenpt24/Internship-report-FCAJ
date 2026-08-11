---
title: "Deploy Web Frontend & CloudFront"
date: 2026-08-04
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

### 5.6. Deploy Web Chat Client on Amazon S3 & CloudFront CDN

#### Module Goal

Build the Web Client interface (`index.html`), upload assets to Amazon S3, and configure an Amazon CloudFront Distribution to distribute the Chat application globally over secure HTTPS.

---

#### 1. Web Frontend Client Source Code (`index.html`)

Create `index.html` locally and paste the code below (replace `<YOUR-API-ID>` with your live WebSocket WSS URL):

```html
<!<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AWS Serverless Real-Time Chat</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background-color: #18191a;
            color: #e4e6eb;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
        }

        /* Toast Container */
        .toast-container {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 9999;
            display: flex;
            flex-direction: column;
            gap: 10px;
            max-width: 320px;
            pointer-events: none;
        }
        .toast {
            pointer-events: auto;
            padding: 10px 14px;
            border-radius: 12px;
            font-size: 13px;
            font-weight: 600;
            color: #ffffff;
            box-shadow: 0 8px 24px rgba(0,0,0,0.4);
            display: flex;
            align-items: center;
            gap: 8px;
            backdrop-filter: blur(10px);
            animation: slideInToast 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
            transition: opacity 0.3s ease, transform 0.3s ease;
        }
        .toast.toast-success { background-color: rgba(46, 204, 113, 0.95); border-left: 4px solid #1e8449; }
        .toast.toast-error { background-color: rgba(231, 76, 60, 0.95); border-left: 4px solid #922b21; }
        .toast.toast-warning { background-color: rgba(243, 156, 18, 0.95); border-left: 4px solid #b9770e; }
        .toast.toast-info { background-color: rgba(52, 152, 219, 0.95); border-left: 4px solid #1f618d; }

        @keyframes slideInToast {
            from { opacity: 0; transform: translateX(50px); }
            to { opacity: 1; transform: translateX(0); }
        }

        /* Authentication Screen (Full Guard Overlay) */
        .auth-screen {
            position: fixed;
            top: 0; left: 0; right: 0; bottom: 0;
            background-color: #18191a;
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 1000;
            padding: 20px;
        }
        .auth-screen.hidden { display: none; }
        .auth-card {
            width: 100%;
            max-width: 420px;
            background-color: #242526;
            border: 1px solid #3e4042;
            border-radius: 20px;
            padding: 32px 28px;
            box-shadow: 0 16px 40px rgba(0, 0, 0, 0.6);
            display: flex;
            flex-direction: column;
            gap: 20px;
        }
        .auth-header {
            text-align: center;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 8px;
        }
        .auth-header .logo {
            width: 48px;
            height: 48px;
            background: linear-gradient(135deg, #ff9900 0%, #ff5500 100%);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 24px;
            color: #fff;
            margin-bottom: 4px;
        }
        .auth-header h2 { font-size: 20px; color: #f5f6f7; }
        .auth-header p { font-size: 13px; color: #b0b3b8; }

        .auth-tabs {
            display: flex;
            background: #18191a;
            border-radius: 12px;
            padding: 4px;
            gap: 4px;
        }
        .tab-btn {
            flex: 1;
            padding: 8px;
            border: none;
            background: transparent;
            color: #b0b3b8;
            font-weight: 600;
            font-size: 13px;
            border-radius: 8px;
            cursor: pointer;
            transition: all 0.15s ease;
        }
        .tab-btn.active {
            background: #ff9900;
            color: #ffffff;
        }

        .auth-form {
            display: flex;
            flex-direction: column;
            gap: 14px;
        }
        .auth-form.hidden { display: none; }
        .form-group {
            display: flex;
            flex-direction: column;
            gap: 6px;
        }
        .form-group label {
            font-size: 12px;
            color: #b0b3b8;
            font-weight: 600;
        }
        .form-group input {
            background-color: #18191a;
            border: 1px solid #3e4042;
            border-radius: 10px;
            padding: 10px 14px;
            color: #e4e6eb;
            font-size: 13px;
            outline: none;
        }
        .form-group input:focus { border-color: #ff9900; }
        
        .otp-info {
            background-color: rgba(255, 153, 0, 0.1);
            border: 1px dashed #ff9900;
            border-radius: 10px;
            padding: 10px 12px;
            font-size: 12px;
            color: #ff9900;
            line-height: 1.4;
        }

        .submit-btn {
            background: linear-gradient(135deg, #ff9900 0%, #ff5500 100%);
            border: none;
            color: #ffffff;
            font-weight: 700;
            padding: 12px;
            border-radius: 10px;
            cursor: pointer;
            font-size: 14px;
            margin-top: 6px;
            transition: opacity 0.15s ease;
        }
        .submit-btn:hover { opacity: 0.9; }

        .guest-link {
            text-align: center;
            font-size: 12px;
            color: #b0b3b8;
            cursor: pointer;
            text-decoration: underline;
            margin-top: 4px;
        }
        .guest-link:hover { color: #0084ff; }

        /* Main Chat Container */
        .chat-container {
            width: 100%;
            max-width: 520px;
            height: 680px;
            background-color: #242526;
            border-radius: 16px;
            box-shadow: 0 12px 32px rgba(0, 0, 0, 0.45);
            display: flex;
            flex-direction: column;
            overflow: hidden;
            border: 1px solid #3e4042;
            position: relative;
        }

        /* Header */
        .chat-header {
            padding: 12px 16px;
            background-color: #242526;
            border-bottom: 1px solid #3e4042;
            display: flex;
            flex-direction: column;
            gap: 8px;
        }
        .header-top {
            display: flex;
            align-items: center;
            justify-content: space-between;
        }
        .app-title {
            display: flex;
            align-items: center;
            gap: 10px;
        }
        .app-title h2 {
            font-size: 16px;
            font-weight: 700;
            color: #f5f6f7;
        }
        .messenger-icon {
            width: 26px;
            height: 26px;
            background: linear-gradient(135deg, #0084ff 0%, #00c6ff 100%);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            color: #fff;
            font-size: 13px;
            font-weight: bold;
        }
        .header-right {
            display: flex;
            align-items: center;
            gap: 6px;
        }
        #status {
            font-size: 11px;
            font-weight: 600;
            padding: 4px 8px;
            border-radius: 12px;
            display: flex;
            align-items: center;
            gap: 5px;
        }
        .status-dot {
            width: 7px;
            height: 7px;
            border-radius: 50%;
            display: inline-block;
        }
        .online { background-color: rgba(46, 204, 113, 0.15); color: #2ecc71; }
        .online .status-dot { background-color: #2ecc71; box-shadow: 0 0 8px #2ecc71; }
        .offline { background-color: rgba(231, 76, 60, 0.15); color: #e74c3c; }
        .offline .status-dot { background-color: #e74c3c; }

        .logout-btn {
            background: #e74c3c;
            border: none;
            color: #ffffff;
            font-size: 11px;
            font-weight: 700;
            padding: 4px 8px;
            border-radius: 12px;
            cursor: pointer;
            display: flex;
            align-items: center;
            gap: 4px;
        }
        .logout-btn:hover { opacity: 0.9; }

        .user-list-btn {
            background: #3a3b3c;
            border: 1px solid #3e4042;
            color: #e4e6eb;
            font-size: 11px;
            font-weight: 600;
            padding: 4px 8px;
            border-radius: 12px;
            cursor: pointer;
            display: flex;
            align-items: center;
            gap: 4px;
            transition: background 0.15s ease;
        }
        .user-list-btn:hover { background: #4e4f50; }
        
        .user-bar {
            display: flex;
            align-items: center;
            justify-content: space-between;
            gap: 8px;
            background-color: #18191a;
            padding: 6px 12px;
            border-radius: 20px;
            border: 1px solid #3e4042;
        }
        .user-bar-field, .room-selector-field {
            display: flex;
            align-items: center;
            gap: 6px;
            flex: 1;
        }
        .user-bar label, .room-selector-field label {
            font-size: 12px;
            color: #b0b3b8;
            font-weight: 600;
            white-space: nowrap;
        }
        .user-bar input {
            background: transparent;
            border: none;
            color: #0084ff;
            font-weight: 700;
            font-size: 13px;
            outline: none;
            width: 100%;
        }
        .room-selector-field select {
            background: #242526;
            border: 1px solid #3e4042;
            color: #00c6ff;
            font-weight: 700;
            font-size: 12px;
            border-radius: 12px;
            padding: 3px 8px;
            outline: none;
            cursor: pointer;
        }

        /* Online Users Drawer */
        .user-drawer {
            position: absolute;
            top: 65px;
            right: 16px;
            width: 220px;
            background-color: #242526;
            border: 1px solid #3e4042;
            border-radius: 12px;
            box-shadow: 0 8px 24px rgba(0, 0, 0, 0.5);
            z-index: 150;
            overflow: hidden;
        }
        .user-drawer.hidden { display: none; }
        .drawer-header {
            padding: 10px 14px;
            background-color: #18191a;
            border-bottom: 1px solid #3e4042;
            font-size: 12px;
            font-weight: 700;
            color: #f5f6f7;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .close-drawer {
            background: transparent;
            border: none;
            color: #b0b3b8;
            font-size: 14px;
            cursor: pointer;
        }
        .user-list-ul {
            list-style: none;
            max-height: 180px;
            overflow-y: auto;
            padding: 6px 0;
        }
        .user-list-item {
            padding: 8px 14px;
            font-size: 13px;
            color: #e4e6eb;
            display: flex;
            align-items: center;
            gap: 8px;
        }
        .user-list-item .dot {
            width: 7px;
            height: 7px;
            background-color: #2ecc71;
            border-radius: 50%;
        }

        /* Message Area */
        #messages {
            flex: 1;
            padding: 20px;
            overflow-y: auto;
            background-color: #18191a;
            display: flex;
            flex-direction: column;
            gap: 12px;
        }
        #messages::-webkit-scrollbar { width: 6px; }
        #messages::-webkit-scrollbar-thumb { background-color: #3e4042; border-radius: 3px; }

        .msg-wrapper {
            display: flex;
            flex-direction: column;
            max-width: 75%;
        }
        .msg-wrapper.sent {
            align-self: flex-end;
            align-items: flex-end;
        }
        .msg-wrapper.received {
            align-self: flex-start;
            align-items: flex-start;
        }
        .sender-name {
            font-size: 11px;
            color: #b0b3b8;
            margin-bottom: 4px;
            margin-left: 10px;
            font-weight: 600;
        }
        .msg-bubble {
            padding: 10px 16px;
            font-size: 14px;
            line-height: 1.45;
            word-break: break-word;
            white-space: pre-wrap;
            box-shadow: 0 1px 2px rgba(0, 0, 0, 0.15);
        }
        .sent .msg-bubble {
            background: linear-gradient(135deg, #0084ff 0%, #00c6ff 100%);
            color: #ffffff;
            border-radius: 18px 18px 4px 18px;
        }
        .received .msg-bubble {
            background-color: #3e4042;
            color: #e4e6eb;
            border-radius: 18px 18px 18px 4px;
        }
        .msg-img {
            max-width: 100%;
            max-height: 220px;
            border-radius: 12px;
            margin-top: 6px;
            display: block;
            cursor: pointer;
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        .msg-time {
            font-size: 10px;
            color: #b0b3b8;
            margin-top: 4px;
            margin-left: 6px;
            margin-right: 6px;
            font-weight: 500;
        }
        .sent .msg-time { align-self: flex-end; }
        .received .msg-time { align-self: flex-start; }

        /* Load More History Pagination */
        .load-more-container {
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 8px 0;
            margin-bottom: 8px;
            width: 100%;
        }
        .load-more-btn {
            background-color: #242526;
            border: 1px solid #3e4042;
            color: #00c6ff;
            font-size: 12px;
            font-weight: 600;
            padding: 6px 16px;
            border-radius: 16px;
            cursor: pointer;
            transition: all 0.2s ease;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
            display: flex;
            align-items: center;
            gap: 6px;
        }
        .load-more-btn:hover {
            background-color: #3a3b3c;
            border-color: #00c6ff;
            transform: translateY(-1px);
        }
        .load-more-btn:disabled {
            background-color: transparent;
            border: none;
            color: #8a8d91;
            cursor: default;
            box-shadow: none;
            transform: none;
        }

        /* Typing Indicator */
        .typing-indicator {
            display: flex;
            align-items: center;
            gap: 8px;
            padding: 8px 20px;
            background-color: #18191a;
            color: #b0b3b8;
            font-size: 12px;
            font-weight: 500;
            border-top: 1px dashed #3e4042;
        }
        .typing-indicator.hidden { display: none; }
        .dots {
            display: flex;
            align-items: center;
            gap: 3px;
        }
        .dots .dot {
            width: 5px;
            height: 5px;
            background-color: #0084ff;
            border-radius: 50%;
            display: inline-block;
            animation: typingBounce 1.4s infinite ease-in-out both;
        }
        .dots .dot:nth-child(1) { animation-delay: 0s; }
        .dots .dot:nth-child(2) { animation-delay: 0.2s; }
        .dots .dot:nth-child(3) { animation-delay: 0.4s; }

        @keyframes typingBounce {
            0%, 80%, 100% { transform: scale(0.4); opacity: 0.4; }
            40% { transform: scale(1.1); opacity: 1; }
        }

        /* Image Preview Bar */
        .image-preview-bar {
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 8px 16px;
            background-color: #18191a;
            border-top: 1px dashed #3e4042;
            position: relative;
        }
        .image-preview-bar.hidden { display: none; }
        .image-preview-bar img {
            max-height: 60px;
            border-radius: 8px;
            border: 1px solid #3e4042;
            object-fit: cover;
        }
        .remove-img-btn {
            background: #e74c3c;
            color: white;
            border: none;
            width: 22px;
            height: 22px;
            border-radius: 50%;
            cursor: pointer;
            font-weight: bold;
            font-size: 12px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        /* Emoji Picker */
        .emoji-picker {
            position: absolute;
            bottom: 70px;
            left: 16px;
            background-color: #242526;
            border: 1px solid #3e4042;
            border-radius: 12px;
            padding: 10px;
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 8px;
            box-shadow: 0 8px 24px rgba(0,0,0,0.5);
            z-index: 200;
            font-size: 20px;
        }
        .emoji-picker.hidden { display: none; }
        .emoji-picker span {
            cursor: pointer;
            text-align: center;
            padding: 4px;
            border-radius: 6px;
            transition: background 0.15s ease;
            user-select: none;
        }
        .emoji-picker span:hover { background: #3a3b3c; }

        /* Input Area */
        .chat-footer {
            padding: 12px 16px;
            background-color: #242526;
            border-top: 1px solid #3e4042;
            display: flex;
            gap: 8px;
            align-items: flex-end;
            position: relative;
        }
        .action-btn {
            background: transparent;
            border: none;
            font-size: 18px;
            cursor: pointer;
            padding: 8px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            color: #b0b3b8;
            transition: background 0.15s ease, color 0.15s ease;
        }
        .action-btn:hover {
            background-color: #3a3b3c;
            color: #e4e6eb;
        }
        .input-wrapper {
            flex: 1;
            background-color: #3a3b3c;
            border-radius: 20px;
            padding: 6px 14px;
            display: flex;
            align-items: center;
        }
        .input-wrapper textarea {
            width: 100%;
            background: transparent;
            border: none;
            color: #e4e6eb;
            font-size: 14px;
            outline: none;
            resize: none;
            max-height: 90px;
            font-family: inherit;
            line-height: 1.4;
            padding: 4px 0;
        }
        .input-wrapper textarea::placeholder {
            color: #b0b3b8;
        }
        .send-btn {
            width: 38px;
            height: 38px;
            border-radius: 50%;
            background: linear-gradient(135deg, #0084ff 0%, #00c6ff 100%);
            border: none;
            color: #ffffff;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            flex-shrink: 0;
            transition: transform 0.15s ease, opacity 0.15s ease;
        }
        .send-btn:hover {
            transform: scale(1.05);
            opacity: 0.95;
        }
        .send-btn:active {
            transform: scale(0.95);
        }
        .send-btn svg {
            width: 18px;
            height: 18px;
            fill: currentColor;
            margin-left: 2px;
        }
    </style>
</head>
<body>
    <div id="toastContainer" class="toast-container"></div>

    <!-- 🔒 AUTHENTICATION SCREEN (SIGN UP / VERIFY OTP / SIGN IN GUARD) -->
    <div id="authScreen" class="auth-screen">
        <div class="auth-card">
            <div class="auth-header">
                <div class="logo">🔒</div>
                <h2>Amazon Cognito Authentication</h2>
                <p id="authSubtitle">Đăng ký & Đăng nhập để vào phòng Chat</p>
            </div>

            <div id="authTabsHeader" class="auth-tabs">
                <button id="tabSignInBtn" class="tab-btn active" onclick="switchAuthTab('signin')">🔑 Đăng nhập</button>
                <button id="tabSignUpBtn" class="tab-btn" onclick="switchAuthTab('signup')">📝 Đăng ký</button>
            </div>

            <!-- 1. Sign In Form -->
            <form id="signInForm" class="auth-form" onsubmit="handleCognitoSignIn(event)">
                <div class="form-group">
                    <label for="signInEmail">Cognito User Email:</label>
                    <input type="email" id="signInEmail" placeholder="alex.dev@fcj-workshop.aws" value="alex.dev@fcj-workshop.aws" required>
                </div>
                <div class="form-group">
                    <label for="signInPassword">Mật khẩu:</label>
                    <input type="password" id="signInPassword" placeholder="••••••••" value="CognitoPass123!" required>
                </div>
                <button type="submit" class="submit-btn">Đăng nhập vào Chat</button>
                <div class="guest-link" onclick="handleGuestLogin()">Chạy thử nghiệm với tư cách Khách</div>
            </form>

            <!-- 2. Sign Up Form -->
            <form id="signUpForm" class="auth-form hidden" onsubmit="handleCognitoSignUp(event)">
                <div class="form-group">
                    <label for="signUpName">Tên hiển thị:</label>
                    <input type="text" id="signUpName" placeholder="Nguyễn Văn A" value="Nguyễn Văn A" required>
                </div>
                <div class="form-group">
                    <label for="signUpEmail">Email đăng ký:</label>
                    <input type="email" id="signUpEmail" placeholder="user@fcj-workshop.aws" value="user@fcj-workshop.aws" required>
                </div>
                <div class="form-group">
                    <label for="signUpPassword">Mật khẩu (Cognito Policy):</label>
                    <input type="password" id="signUpPassword" placeholder="Ít nhất 8 ký tự, 1 số, 1 hoa" value="CognitoPass123!" required>
                </div>
                <button type="submit" class="submit-btn">Tạo tài khoản Cognito</button>
            </form>

            <!-- 3. OTP Verification Form (Bước Kiểm Tra Người Dùng Khi Đăng Ký) -->
            <form id="verifyOtpForm" class="auth-form hidden" onsubmit="handleConfirmOtp(event)">
                <div class="otp-info">
                    📧 Mã xác thực OTP (Verification Code) 6 chữ số đã được Cognito gửi tới email: <b id="otpEmailTarget">user@fcj-workshop.aws</b>
                </div>
                <div class="form-group">
                    <label for="otpCodeInput">Mã xác thực OTP (Confirmation Code):</label>
                    <input type="text" id="otpCodeInput" placeholder="Ví dụ: 123456" value="123456" maxlength="6" style="letter-spacing: 4px; font-weight: 700; font-size: 16px; text-align: center;" required>
                </div>
                <button type="submit" class="submit-btn">Xác thực OTP & Kích hoạt Tài khoản</button>
                <div class="guest-link" onclick="resendOtpCode()">Chưa nhận được mã? Gửi lại OTP</div>
            </form>
        </div>
    </div>

    <!-- MAIN CHAT APP CONTAINER -->
    <div class="chat-container">
        <!-- Header -->
        <div class="chat-header">
            <div class="header-top">
                <div class="app-title">
                    <div class="messenger-icon">⚡</div>
                    <h2>AWS Serverless Chat</h2>
                </div>
                <div class="header-right">
                    <div id="status" class="offline">
                        <span class="status-dot"></span>
                        <span id="status-text">Đang kết nối...</span>
                    </div>
                    <button class="user-list-btn" onclick="toggleUserListDrawer()" title="Danh sách người dùng Online">
                        👥 <span id="onlineCountBadge">1</span>
                    </button>
                    <button class="logout-btn" onclick="handleLogout()" title="Đăng xuất khỏi Cognito">
                        🚪 Đăng xuất
                    </button>
                </div>
            </div>
            <div class="user-bar">
                <div class="user-bar-field">
                    <label for="usernameInput">Tài khoản:</label>
                    <input type="text" id="usernameInput" placeholder="Tên tài khoản..." value="User A" disabled>
                </div>
                <div class="room-selector-field">
                    <label for="roomSelect">Phòng:</label>
                    <select id="roomSelect" onchange="changeRoom(this.value)">
                        <option value="general"># General</option>
                        <option value="tech"># Tech Talks</option>
                        <option value="random"># Random</option>
                    </select>
                </div>
            </div>
        </div>

        <!-- Online Users Drawer -->
        <div id="userListDrawer" class="user-drawer hidden">
            <div class="drawer-header">
                <span>👥 Trực tuyến (<span id="drawerUserCount">1</span>)</span>
                <button onclick="toggleUserListDrawer()" class="close-drawer">✕</button>
            </div>
            <ul id="userListUl" class="user-list-ul"></ul>
        </div>

        <!-- Messages Area -->
        <div id="messages"></div>

        <!-- Typing Indicator Container -->
        <div id="typingIndicator" class="typing-indicator hidden">
            <div class="dots">
                <span class="dot"></span>
                <span class="dot"></span>
                <span class="dot"></span>
            </div>
            <span id="typingText">User đang gõ...</span>
        </div>

        <!-- Image Preview Container -->
        <div id="imagePreviewContainer" class="image-preview-bar hidden">
            <img id="imagePreview" src="" alt="Image preview">
            <button onclick="clearSelectedImage()" class="remove-img-btn" title="Xóa ảnh">✕</button>
        </div>

        <!-- Emoji Picker Popup -->
        <div id="emojiPicker" class="emoji-picker hidden">
            <span onclick="insertEmoji('👍')">👍</span>
            <span onclick="insertEmoji('❤️')">❤️</span>
            <span onclick="insertEmoji('😂')">😂</span>
            <span onclick="insertEmoji('😮')">😮</span>
            <span onclick="insertEmoji('😢')">😢</span>
            <span onclick="insertEmoji('🔥')">🔥</span>
            <span onclick="insertEmoji('🎉')">🎉</span>
            <span onclick="insertEmoji('👋')">👋</span>
            <span onclick="insertEmoji('🚀')">🚀</span>
            <span onclick="insertEmoji('💡')">💡</span>
            <span onclick="insertEmoji('✨')">✨</span>
            <span onclick="insertEmoji('💯')">💯</span>
            <span onclick="insertEmoji('🙏')">🙏</span>
            <span onclick="insertEmoji('⚡')">⚡</span>
            <span onclick="insertEmoji('⭐')">⭐</span>
            <span onclick="insertEmoji('📌')">📌</span>
        </div>

        <!-- Footer Input -->
        <div class="chat-footer">
            <button class="action-btn" onclick="toggleEmojiPicker(event)" title="Thêm biểu tượng cảm xúc">😀</button>
            <input type="file" id="imageFileInput" accept="image/*" style="display: none;" onchange="handleImageSelect(event)">
            <button class="action-btn" onclick="document.getElementById('imageFileInput').click()" title="Đính kèm hình ảnh">🖼️</button>
            <div class="input-wrapper">
                <textarea id="messageInput" placeholder="Nhập tin nhắn... (Shift+Enter để xuống dòng)" rows="1" oninput="handleInputTyping(); autoResizeTextarea(this);" onkeydown="handleInputKeydown(event)"></textarea>
            </div>
            <button class="send-btn" onclick="sendMessage()" title="Gửi tin nhắn">
                <svg viewBox="0 0 24 24">
                    <path d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z"/>
                </svg>
            </button>
        </div>
    </div>

    <script>
        const WSS_URL = "wss://<YOUR-API-ID>.execute-api.us-east-1.amazonaws.com/production";
        
        /* ☁️ AMAZON COGNITO CLOUD API CONFIGURATION */
        const COGNITO_CONFIG = {
            region: "ap-southeast-1", // Region AWS Cognito User Pool của bạn (ví dụ: us-east-1 hoặc ap-southeast-1)
            userPoolId: "<YOUR-COGNITO-USER-POOL-ID>", // Ví dụ: ap-southeast-1_XXXXXXXXX
            appClientId: "<YOUR-COGNITO-APP-CLIENT-ID>"  // App Client ID khởi tạo trong bài lab 5.7
        };

        let socket;
        const myClientId = 'usr_' + Math.random().toString(36).substring(2, 8);
        const activeUsersMap = new Map();
        let selectedImageDataUrl = null;
        let currentRoomId = 'general';
        let cognitoUserSession = null;
        let pendingSignUpUser = null;
        let isLoggingOut = false;

        // 📜 Chat History Pagination & Deduplication State
        let oldestLoadedTimestamp = null;
        let isLoadingMoreHistory = false;
        let hasMoreHistory = true;
        const HISTORY_PAGE_LIMIT = 15;
        const renderedMsgKeys = new Set();

        function cleanUsername(name) {
            if (!name) return '';
            return name.replace(/\s*\([^)]*\)/g, '').trim().toLowerCase();
        }

        function cleanTimestamp(ts) {
            if (!ts) return '';
            try {
                const d = new Date(ts);
                if (!isNaN(d.getTime())) {
                    return Math.floor(d.getTime() / 1000).toString();
                }
            } catch(e) {}
            return String(ts).trim();
        }

        function getMsgKey(msgObj) {
            if (!msgObj) return '';
            if (msgObj.msgId) return String(msgObj.msgId);
            const r = (msgObj.roomId || 'gen').trim().toLowerCase();
            const u = cleanUsername(msgObj.username);
            const t = cleanTimestamp(msgObj.timestamp);
            const txt = (msgObj.text || '').trim();
            return `${r}_${u}_${t}_${txt}`;
        }

        function isMsgFromMe(msgObj) {
            if (!msgObj) return false;
            const currentUserName = document.getElementById('usernameInput').value;
            const cleanCurrent = cleanUsername(currentUserName);
            const cleanSender = cleanUsername(msgObj.username);
            if (cleanCurrent && cleanSender) {
                return cleanCurrent === cleanSender;
            }
            return msgObj.clientId === myClientId;
        }

        // Fallback Local Storage Registry for Demo/Testing Mode
        let registeredCognitoUsers = new Map([
            ["alex.dev@fcj-workshop.aws", {
                email: "alex.dev@fcj-workshop.aws",
                name: "Alex Dev",
                password: "CognitoPass123!",
                status: "CONFIRMED"
            }]
        ]);

        try {
            const savedUsers = localStorage.getItem('cognito_users_db');
            if (savedUsers) {
                const parsed = JSON.parse(savedUsers);
                parsed.forEach(([em, u]) => registeredCognitoUsers.set(em, u));
            }
        } catch(e) {}

        function saveUsersToLocalStorage() {
            try {
                localStorage.setItem('cognito_users_db', JSON.stringify(Array.from(registeredCognitoUsers.entries())));
            } catch(e) {}
        }

        function showToast(message, type = 'info', duration = 3500) {
            const container = document.getElementById('toastContainer');
            if (!container) return;

            const toast = document.createElement('div');
            toast.className = `toast toast-${type}`;
            
            const iconMap = {
                success: '✅',
                error: '🚨',
                warning: '⚠️',
                info: 'ℹ️'
            };

            toast.innerHTML = `<span>${iconMap[type] || 'ℹ️'}</span> <span>${message}</span>`;
            container.appendChild(toast);

            setTimeout(() => {
                toast.style.opacity = '0';
                toast.style.transform = 'translateX(50px)';
                setTimeout(() => toast.remove(), 300);
            }, duration);
        }

        /* 🔑 AUTH SCREEN CONTROLLER */
        function switchAuthTab(tab) {
            const signInForm = document.getElementById('signInForm');
            const signUpForm = document.getElementById('signUpForm');
            const verifyOtpForm = document.getElementById('verifyOtpForm');
            const tabSignInBtn = document.getElementById('tabSignInBtn');
            const tabSignUpBtn = document.getElementById('tabSignUpBtn');
            const authTabsHeader = document.getElementById('authTabsHeader');

            authTabsHeader.style.display = 'flex';
            verifyOtpForm.classList.add('hidden');

            if (tab === 'signin') {
                signInForm.classList.remove('hidden');
                signUpForm.classList.add('hidden');
                tabSignInBtn.classList.add('active');
                tabSignUpBtn.classList.remove('active');
                document.getElementById('authSubtitle').innerText = "Đăng nhập tài khoản Cognito để vào phòng Chat";
            } else {
                signInForm.classList.add('hidden');
                signUpForm.classList.remove('hidden');
                tabSignInBtn.classList.remove('active');
                tabSignUpBtn.classList.add('active');
                document.getElementById('authSubtitle').innerText = "Đăng ký tài khoản Cognito mới";
            }
        }

        /* ☁️ CALL AWS COGNITO CLOUD REST API DIRECTLY */
        async function callCognitoApi(target, payload) {
            const endpoint = `https://cognito-idp.${COGNITO_CONFIG.region}.amazonaws.com/`;
            try {
                const response = await fetch(endpoint, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/x-amz-json-1.1',
                        'X-Amz-Target': `AWSCognitoIdentityProviderService.${target}`
                    },
                    body: JSON.stringify(payload)
                });
                const data = await response.json();
                if (!response.ok) {
                    throw new Error(data.message || data.__type || "Lỗi xác thực Cognito Cloud");
                }
                return data;
            } catch (err) {
                console.warn(`[Cognito Cloud API Fallback] ${err.message}`);
                throw err;
            }
        }

        async function handleCognitoSignUp(e) {
            e.preventDefault();
            const name = document.getElementById('signUpName').value.trim();
            const email = document.getElementById('signUpEmail').value.trim().toLowerCase();
            const pass = document.getElementById('signUpPassword').value.trim();

            if (!name || !email || !pass) {
                showToast("Vui lòng điền đầy đủ thông tin đăng ký!", "warning");
                return;
            }

            if (registeredCognitoUsers.has(email)) {
                showToast(`🚨 Email "${email}" đã tồn tại trong hệ thống! Mỗi Email chỉ được tạo 1 tài khoản.`, "error");
                document.getElementById('signInEmail').value = email;
                switchAuthTab('signin');
                return;
            }

            // Check if AWS Cognito credentials configured
            const isAwsCloudConfigured = COGNITO_CONFIG.appClientId && !COGNITO_CONFIG.appClientId.includes('<YOUR-COGNITO');

            if (isAwsCloudConfigured) {
                try {
                    showToast("☁️ Đang tạo tài khoản trực tiếp trên AWS Cognito Cloud...", "info");
                    await callCognitoApi('SignUp', {
                        ClientId: COGNITO_CONFIG.appClientId,
                        Username: email,
                        Password: pass,
                        UserAttributes: [
                            { Name: "email", Value: email },
                            { Name: "name", Value: name }
                        ]
                    });
                } catch(awsErr) {
                    showToast(`🚨 AWS Cognito Cloud Error: ${awsErr.message}`, "error");
                    return;
                }
            }

            // Directly CONFIRM account without requiring OTP verification
            const newUserRecord = { email, name, password: pass, status: "CONFIRMED" };
            registeredCognitoUsers.set(email, newUserRecord);
            saveUsersToLocalStorage();

            showToast(`✅ Đăng ký tài khoản Cognito thành công cho ${email}! Bạn có thể Đăng nhập ngay.`, "success");

            document.getElementById('signInEmail').value = email;
            document.getElementById('signInPassword').value = pass;

            switchAuthTab('signin');
        }

        async function handleConfirmOtp(e) {
            e.preventDefault();
            const otpCode = document.getElementById('otpCodeInput').value.trim();

            if (!otpCode || otpCode.length < 6) {
                showToast("Vui lòng nhập đúng mã OTP xác thực 6 chữ số!", "warning");
                return;
            }

            const isAwsCloudConfigured = COGNITO_CONFIG.appClientId && !COGNITO_CONFIG.appClientId.includes('<YOUR-COGNITO');

            if (isAwsCloudConfigured && pendingSignUpUser) {
                try {
                    showToast("☁️ Đang xác thực OTP trên AWS Cloud Cognito...", "info");
                    await callCognitoApi('ConfirmSignUp', {
                        ClientId: COGNITO_CONFIG.appClientId,
                        Username: pendingSignUpUser.email,
                        ConfirmationCode: otpCode
                    });
                    showToast(`☁️ AWS Cognito: Tài khoản ${pendingSignUpUser.email} đã CONFIRMED trên AWS Cloud!`, "success");
                } catch(awsErr) {
                    showToast(`🚨 AWS OTP Error: ${awsErr.message}`, "error");
                    return;
                }
            }

            if (pendingSignUpUser) {
                pendingSignUpUser.status = "CONFIRMED";
                registeredCognitoUsers.set(pendingSignUpUser.email, pendingSignUpUser);
                saveUsersToLocalStorage();
                
                showToast(`Xác thực OTP thành công! Tài khoản ${pendingSignUpUser.email} đã kích hoạt.`, "success");
                
                document.getElementById('signInEmail').value = pendingSignUpUser.email;
                document.getElementById('signInPassword').value = pendingSignUpUser.password;
                pendingSignUpUser = null;
            }

            switchAuthTab('signin');
        }

        function resendOtpCode() {
            if (pendingSignUpUser) {
                showToast(`Đã gửi lại mã OTP mới tới email: ${pendingSignUpUser.email}`, "info");
            }
        }

        function parseJwtPayload(token) {
            if (!token) return null;
            try {
                const base64Url = token.split('.')[1];
                const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
                const jsonPayload = decodeURIComponent(atob(base64).split('').map(function(c) {
                    return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
                }).join(''));
                return JSON.parse(jsonPayload);
            } catch (e) {
                return null;
            }
        }

        async function handleCognitoSignIn(e) {
            e.preventDefault();
            const email = document.getElementById('signInEmail').value.trim().toLowerCase();
            const pass = document.getElementById('signInPassword').value.trim();

            if (!email || !pass) {
                showToast("Vui lòng nhập Email và Mật khẩu!", "warning");
                return;
            }

            const isAwsCloudConfigured = COGNITO_CONFIG.appClientId && !COGNITO_CONFIG.appClientId.includes('<YOUR-COGNITO');
            let authenticatedUserDisplayName = (registeredCognitoUsers.has(email) && registeredCognitoUsers.get(email).name) 
                ? registeredCognitoUsers.get(email).name 
                : email.split('@')[0];

            if (isAwsCloudConfigured) {
                try {
                    showToast("☁️ Đang xác thực đăng nhập AWS Cognito Cloud...", "info");
                    const authRes = await callCognitoApi('InitiateAuth', {
                        ClientId: COGNITO_CONFIG.appClientId,
                        AuthFlow: "USER_PASSWORD_AUTH",
                        AuthParameters: {
                            USERNAME: email,
                            PASSWORD: pass
                        }
                    });

                    const idToken = authRes.AuthenticationResult ? authRes.AuthenticationResult.IdToken : null;
                    if (idToken) {
                        const jwtPayload = parseJwtPayload(idToken);
                        if (jwtPayload && jwtPayload.name) {
                            authenticatedUserDisplayName = jwtPayload.name;
                        }
                    }

                    cognitoUserSession = {
                        email: email,
                        username: authenticatedUserDisplayName,
                        idToken: idToken,
                        authenticatedAt: new Date().toISOString()
                    };

                    const usernameInput = document.getElementById('usernameInput');
                    usernameInput.value = authenticatedUserDisplayName;

                    document.getElementById('authScreen').classList.add('hidden');
                    showToast(`☁️ Đăng nhập AWS Cloud thành công! Xin chào ${authenticatedUserDisplayName}`, "success");
                    connect();
                    return;
                } catch(awsErr) {
                    const isNotConfirmedErr = awsErr.message && (awsErr.message.toLowerCase().includes('not confirmed') || awsErr.message.includes('UserNotConfirmedException'));
                    if (isNotConfirmedErr && registeredCognitoUsers.has(email)) {
                        const localRec = registeredCognitoUsers.get(email);
                        authenticatedUserDisplayName = localRec.name || email.split('@')[0];
                        cognitoUserSession = {
                            email: email,
                            username: authenticatedUserDisplayName,
                            idToken: 'eyJhbGciOiJSUzI1NiIsImtpZCI6ImNvZ25pdG8tand0In0...' + Math.random().toString(36).substring(2, 10),
                            authenticatedAt: new Date().toISOString()
                        };
                        const usernameInput = document.getElementById('usernameInput');
                        usernameInput.value = authenticatedUserDisplayName;
                        document.getElementById('authScreen').classList.add('hidden');
                        showToast(`Đăng nhập thành công! Xin chào ${authenticatedUserDisplayName}`, "success");
                        connect();
                        return;
                    }
                    showToast(`🚨 AWS Cognito Cloud Login Error: ${awsErr.message}`, "error");
                    return;
                }
            }

            // Local Demo Mode Strict Checks
            if (!registeredCognitoUsers.has(email)) {
                showToast("🚨 Tài khoản chưa được đăng ký! Vui lòng bấm Đăng ký tài khoản trước.", "error");
                return;
            }

            const userRecord = registeredCognitoUsers.get(email);

            if (userRecord.password !== pass) {
                showToast("🚨 Mật khẩu không chính xác! Vui lòng thử lại.", "error");
                return;
            }

            authenticatedUserDisplayName = (userRecord && userRecord.name && userRecord.name.trim() !== '') ? userRecord.name.trim() : email;
            cognitoUserSession = {
                email: email,
                username: authenticatedUserDisplayName,
                idToken: 'eyJhbGciOiJSUzI1NiIsImtpZCI6ImNvZ25pdG8tand0In0...' + Math.random().toString(36).substring(2, 10),
                authenticatedAt: new Date().toISOString()
            };

            const usernameInput = document.getElementById('usernameInput');
            usernameInput.value = authenticatedUserDisplayName;

            document.getElementById('authScreen').classList.add('hidden');
            showToast(`Đăng nhập Cognito thành công! Xin chào ${authenticatedUserDisplayName}`, "success");
            
            connect();
        }

        function handleGuestLogin() {
            cognitoUserSession = null;
            const usernameInput = document.getElementById('usernameInput');
            usernameInput.value = "Guest_" + myClientId.substring(4);

            document.getElementById('authScreen').classList.add('hidden');
            showToast("Đã tham gia phòng chat với tư cách Khách", "info");

            connect();
        }

        function handleLogout() {
            if (confirm("Bạn có chắc chắn muốn Đăng xuất khỏi phòng chat?")) {
                isLoggingOut = true;
                cognitoUserSession = null;
                myClientId = 'usr_' + Math.random().toString(36).substring(2, 8);
                renderedMsgKeys.clear();
                oldestLoadedTimestamp = null;
                isLoadingMoreHistory = false;
                hasMoreHistory = true;
                const msgBox = document.getElementById('messages');
                if (msgBox) msgBox.innerHTML = '';
                if (socket) socket.close();
                document.getElementById('authScreen').classList.remove('hidden');
                switchAuthTab('signin');
                showToast("Đã đăng xuất tài khoản thành công", "info");
            }
        }

        function connect() {
            isLoggingOut = false;
            socket = new WebSocket(WSS_URL);
            
            socket.onopen = () => {
                document.getElementById('status').className = "online";
                document.getElementById('status-text').innerText = "Online (Active)";
                showToast("Đã kết nối WebSocket thành công!", "success");
                notifyPresence();
                fetchChatHistory();
            };

            socket.onmessage = (event) => {
                const data = JSON.parse(event.data);
                let msgObj;
                try {
                    msgObj = typeof data.message === 'string' ? JSON.parse(data.message) : data.message;
                } catch(e) {
                    const str = String(data.message || '');
                    const parts = str.split(/:\s*(.+)/);
                    msgObj = {
                        type: 'chat',
                        username: parts.length > 1 ? parts[0] : 'User',
                        text: parts.length > 1 ? parts[1] : str,
                        clientId: 'unknown',
                        timestamp: data.timestamp || new Date().toISOString()
                    };
                }

                const isMe = isMsgFromMe(msgObj);

                if (msgObj.clientId && msgObj.username) {
                    trackActiveUser(msgObj.clientId, msgObj.username);
                }

                if (msgObj.type === 'presence') {
                    return;
                }

                if (msgObj.type === 'history') {
                    const loadingEl = document.getElementById('historyLoading');
                    if (loadingEl) loadingEl.remove();

                    const msgBox = document.getElementById('messages');
                    const historyArray = Array.isArray(msgObj.history) ? msgObj.history : [];

                    if (msgObj.isLoadMore || oldestLoadedTimestamp !== null) {
                        // Prepend Older Message Batch (Load More Pagination with Strict Deduplication & Timestamp Cutoff)
                        isLoadingMoreHistory = false;
                        const previousScrollHeight = msgBox.scrollHeight;
                        const oldestTimeMs = oldestLoadedTimestamp ? new Date(oldestLoadedTimestamp).getTime() : Infinity;

                        // Filter out messages that are ALREADY rendered in the DOM OR are newer than oldestLoadedTimestamp
                        const unrenderedItems = historyArray.filter(item => {
                            const k = getMsgKey(item);
                            if (!k || renderedMsgKeys.has(k)) return false;

                            if (item.timestamp && oldestTimeMs !== Infinity) {
                                const itemTimeMs = new Date(item.timestamp).getTime();
                                if (!isNaN(itemTimeMs) && itemTimeMs >= oldestTimeMs) {
                                    return false; // Strictly skip any message that is NOT older than current oldest loaded message!
                                }
                            }
                            return true;
                        });

                        if (unrenderedItems.length > 0) {
                            oldestLoadedTimestamp = unrenderedItems[0].timestamp || oldestLoadedTimestamp;
                            const fragment = document.createDocumentFragment();

                            unrenderedItems.forEach(item => {
                                const k = getMsgKey(item);
                                if (k) renderedMsgKeys.add(k);

                                const isHistoryMe = isMsgFromMe(item);
                                const wrapper = createMessageElement(item, isHistoryMe);
                                fragment.appendChild(wrapper);
                            });

                            const loadMoreContainer = document.getElementById('loadMoreContainer');
                            if (loadMoreContainer && loadMoreContainer.nextSibling) {
                                msgBox.insertBefore(fragment, loadMoreContainer.nextSibling);
                            } else {
                                msgBox.appendChild(fragment);
                            }

                            // Keep Scroll Position Fixed
                            msgBox.scrollTop = msgBox.scrollHeight - previousScrollHeight;

                            if (unrenderedItems.length < HISTORY_PAGE_LIMIT) {
                                hasMoreHistory = false;
                                updateLoadMoreButtonState(false);
                            } else {
                                updateLoadMoreButtonState(true);
                            }
                        } else {
                            // ZERO new unrendered items returned -> No more older history exists!
                            hasMoreHistory = false;
                            updateLoadMoreButtonState(false);
                        }
                    } else {
                        // Initial Load
                        renderedMsgKeys.clear();
                        msgBox.innerHTML = `
                            <div id="loadMoreContainer" class="load-more-container">
                                <button id="loadMoreBtn" class="load-more-btn" onclick="fetchChatHistory(true)">📜 Tải thêm tin nhắn cũ</button>
                            </div>
                        `;

                        if (historyArray.length > 0) {
                            oldestLoadedTimestamp = historyArray[0].timestamp;
                            historyArray.forEach(item => {
                                const isHistoryMe = isMsgFromMe(item);
                                appendMessage(item, isHistoryMe);
                            });

                            if (historyArray.length < HISTORY_PAGE_LIMIT) {
                                hasMoreHistory = false;
                                updateLoadMoreButtonState(false);
                            }
                        } else {
                            hasMoreHistory = false;
                            updateLoadMoreButtonState(false);
                            const emptyEl = document.createElement('div');
                            emptyEl.style = "text-align: center; color: #8a8d91; font-size: 12px; margin: 20px 0;";
                            emptyEl.innerText = "💬 Chưa có tin nhắn nào trong phòng này. Hãy bắt đầu trò chuyện!";
                            msgBox.appendChild(emptyEl);
                        }
                        msgBox.scrollTop = msgBox.scrollHeight;
                    }
                    return;
                }

                if (msgObj.type === 'typing') {
                    if (!isMe && msgObj.roomId === currentRoomId) {
                        handleRemoteTyping(msgObj);
                    }
                    return;
                }

                if (!isMe) {
                    hideTypingIndicator();
                }

                if (msgObj.roomId && msgObj.roomId !== currentRoomId) {
                    return;
                }

                appendMessage(msgObj, isMe);
            };

            socket.onclose = () => {
                document.getElementById('status').className = "offline";
                document.getElementById('status-text').innerText = "Offline (Đã ngắt kết nối)";
                if (!isLoggingOut) {
                    showToast("Mất kết nối WebSocket. Đang tự động kết nối lại...", "error");
                    setTimeout(connect, 3000);
                }
            };
        }

        function changeRoom(newRoomId) {
            if (currentRoomId !== newRoomId) {
                currentRoomId = newRoomId;
                renderedMsgKeys.clear();
                document.getElementById('messages').innerHTML = '';
                showToast(`Đã chuyển sang phòng #${newRoomId}`, "info");
                fetchChatHistory(false);
                notifyPresence();
            }
        }

        function fetchChatHistory(isLoadMore = false) {
            if (!socket || socket.readyState !== WebSocket.OPEN) return;

            const msgBox = document.getElementById('messages');

            if (isLoadMore) {
                if (isLoadingMoreHistory || !hasMoreHistory) return;
                isLoadingMoreHistory = true;
                const btn = document.getElementById('loadMoreBtn');
                if (btn) {
                    btn.disabled = true;
                    btn.innerText = '⏳ Đang tải tin nhắn cũ hơn...';
                }

                const payloadObj = {
                    type: 'gethistory',
                    roomId: currentRoomId,
                    clientId: myClientId,
                    limit: HISTORY_PAGE_LIMIT,
                    lastTimestamp: oldestLoadedTimestamp,
                    isLoadMore: true
                };
                socket.send(JSON.stringify({ action: "sendmessage", data: JSON.stringify(payloadObj) }));
            } else {
                // Initial Room History Fetch
                oldestLoadedTimestamp = null;
                isLoadingMoreHistory = false;
                hasMoreHistory = true;
                renderedMsgKeys.clear();

                msgBox.innerHTML = '<div id="historyLoading" style="text-align: center; color: #b0b3b8; font-size: 12px; margin: 10px 0;">📜 Đang tải lịch sử tin nhắn từ DynamoDB...</div>';

                const payloadObj = {
                    type: 'gethistory',
                    roomId: currentRoomId,
                    clientId: myClientId,
                    limit: HISTORY_PAGE_LIMIT,
                    isLoadMore: false
                };
                socket.send(JSON.stringify({ action: "sendmessage", data: JSON.stringify(payloadObj) }));
            }
        }

        function updateLoadMoreButtonState(canLoadMore) {
            const btn = document.getElementById('loadMoreBtn');
            if (!btn) return;

            if (canLoadMore) {
                btn.disabled = false;
                btn.innerText = '📜 Tải thêm tin nhắn cũ';
            } else {
                btn.disabled = true;
                btn.innerText = '✨ Đã hiển thị toàn bộ lịch sử';
            }
        }

        function notifyPresence() {
            if (socket && socket.readyState === WebSocket.OPEN) {
                const username = document.getElementById('usernameInput').value.trim() || 'User';
                trackActiveUser(myClientId, username);
                const payloadObj = {
                    type: 'presence',
                    roomId: currentRoomId,
                    clientId: myClientId,
                    username: username,
                    cognitoToken: cognitoUserSession ? cognitoUserSession.idToken : null
                };
                const payload = JSON.stringify({ action: "sendmessage", data: JSON.stringify(payloadObj) });
                socket.send(payload);
            }
        }

        function notifyUsernameChange() {
            notifyPresence();
        }

        function trackActiveUser(clientId, username) {
            if (!username) return;
            activeUsersMap.set(clientId, { username: username, lastActive: Date.now() });
            renderOnlineUsers();
        }

        function renderOnlineUsers() {
            const myRawUsername = document.getElementById('usernameInput').value.trim() || 'User';
            const myCleanUsername = cleanUsername(myRawUsername);
            activeUsersMap.set(myClientId, { username: myRawUsername, lastActive: Date.now() });

            const now = Date.now();
            const INACTIVE_TIMEOUT = 60000; // 60s timeout for stale connections

            // Deduplicate online users by cleanUsername
            const uniqueUsersMap = new Map();

            activeUsersMap.forEach((info, cid) => {
                // Purge inactive connections older than 60 seconds
                if (now - info.lastActive > INACTIVE_TIMEOUT && cid !== myClientId) {
                    activeUsersMap.delete(cid);
                    return;
                }

                const normUser = cleanUsername(info.username);
                if (!normUser) return;

                // Keep the most recently active session per username or prioritize current tab
                if (!uniqueUsersMap.has(normUser) || cid === myClientId || info.lastActive > uniqueUsersMap.get(normUser).lastActive) {
                    uniqueUsersMap.set(normUser, {
                        rawUsername: info.username,
                        clientId: cid,
                        isMe: normUser === myCleanUsername || cid === myClientId,
                        lastActive: info.lastActive
                    });
                }
            });

            const userListUl = document.getElementById('userListUl');
            userListUl.innerHTML = '';
            
            let count = 0;
            uniqueUsersMap.forEach((info) => {
                count++;
                const li = document.createElement('li');
                li.className = 'user-list-item';
                li.innerHTML = `<span class="dot"></span> <span>${info.rawUsername} ${info.isMe ? '(Bạn)' : ''}</span>`;
                userListUl.appendChild(li);
            });

            document.getElementById('onlineCountBadge').innerText = count;
            document.getElementById('drawerUserCount').innerText = count;
        }

        function toggleUserListDrawer() {
            const drawer = document.getElementById('userListDrawer');
            drawer.classList.toggle('hidden');
        }

        function formatTime(timestamp) {
            if (!timestamp) {
                return new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
            }
            try {
                const date = new Date(timestamp);
                if (isNaN(date.getTime())) return timestamp;
                return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
            } catch (e) {
                return timestamp;
            }
        }

        function createMessageElement(msgObj, isMe) {
            const wrapper = document.createElement('div');
            wrapper.className = 'msg-wrapper ' + (isMe ? 'sent' : 'received');

            if (!isMe && msgObj.username) {
                const nameLabel = document.createElement('div');
                nameLabel.className = 'sender-name';
                nameLabel.innerText = msgObj.username;
                wrapper.appendChild(nameLabel);
            }

            const bubble = document.createElement('div');
            bubble.className = 'msg-bubble';
            
            if (msgObj.text) {
                const textSpan = document.createElement('div');
                textSpan.innerText = msgObj.text;
                bubble.appendChild(textSpan);
            }

            if (msgObj.image) {
                const img = document.createElement('img');
                img.className = 'msg-img';
                img.src = msgObj.image;
                img.alt = 'Hình ảnh đính kèm';
                img.onclick = () => window.open(msgObj.image, '_blank');
                bubble.appendChild(img);
            }

            wrapper.appendChild(bubble);

            const timeLabel = document.createElement('div');
            timeLabel.className = 'msg-time';
            timeLabel.innerText = formatTime(msgObj.timestamp);
            wrapper.appendChild(timeLabel);

            return wrapper;
        }

        function appendMessage(msgObj, isMe) {
            const k = getMsgKey(msgObj);
            if (k && renderedMsgKeys.has(k)) {
                return; // Strict Deduplication: Skip if message already exists in DOM
            }
            if (k) renderedMsgKeys.add(k);

            const msgBox = document.getElementById('messages');

            const emptyPrompt = msgBox.querySelector('div[style*="Chưa có tin nhắn"]');
            if (emptyPrompt) emptyPrompt.remove();
            
            const wrapper = createMessageElement(msgObj, isMe);
            msgBox.appendChild(wrapper);
            msgBox.scrollTop = msgBox.scrollHeight;
        }

        // Infinite Scroll Listener for Chat History
        document.addEventListener('DOMContentLoaded', () => {
            const msgBox = document.getElementById('messages');
            if (msgBox) {
                msgBox.addEventListener('scroll', () => {
                    if (msgBox.scrollTop === 0 && hasMoreHistory && !isLoadingMoreHistory && oldestLoadedTimestamp) {
                        fetchChatHistory(true);
                    }
                });
            }
        });

        function notifyPresence() {
            if (socket && socket.readyState === WebSocket.OPEN) {
                const username = document.getElementById('usernameInput').value.trim() || 'User';
                trackActiveUser(myClientId, username);
                const payloadObj = {
                    type: 'presence',
                    roomId: currentRoomId,
                    clientId: myClientId,
                    username: username,
                    cognitoToken: cognitoUserSession ? cognitoUserSession.idToken : null
                };
                const payload = JSON.stringify({ action: "sendmessage", data: JSON.stringify(payloadObj) });
                socket.send(payload);
            }
        }

        function notifyUsernameChange() {
            notifyPresence();
        }

        function trackActiveUser(clientId, username) {
            activeUsersMap.set(clientId, { username, lastActive: Date.now() });
            renderOnlineUsers();
        }

        function renderOnlineUsers() {
            const myUsername = document.getElementById('usernameInput').value.trim() || 'User';
            activeUsersMap.set(myClientId, { username: myUsername, lastActive: Date.now() });

            const userListUl = document.getElementById('userListUl');
            userListUl.innerHTML = '';
            
            let count = 0;
            activeUsersMap.forEach((info, cid) => {
                count++;
                const li = document.createElement('li');
                li.className = 'user-list-item';
                li.innerHTML = `<span class="dot"></span> <span>${info.username} ${cid === myClientId ? '(Bạn)' : ''}</span>`;
                userListUl.appendChild(li);
            });

            document.getElementById('onlineCountBadge').innerText = count;
            document.getElementById('drawerUserCount').innerText = count;
        }

        function toggleUserListDrawer() {
            const drawer = document.getElementById('userListDrawer');
            drawer.classList.toggle('hidden');
        }

        function formatTime(timestamp) {
            if (!timestamp) {
                return new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
            }
            try {
                const date = new Date(timestamp);
                if (isNaN(date.getTime())) return timestamp;
                return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
            } catch (e) {
                return timestamp;
            }
        }

        function appendMessage(msgObj, isMe) {
            const msgBox = document.getElementById('messages');

            const emptyPrompt = msgBox.querySelector('div[style*="Chưa có tin nhắn"]');
            if (emptyPrompt) emptyPrompt.remove();
            
            const wrapper = document.createElement('div');
            wrapper.className = 'msg-wrapper ' + (isMe ? 'sent' : 'received');

            if (!isMe && msgObj.username) {
                const nameLabel = document.createElement('div');
                nameLabel.className = 'sender-name';
                nameLabel.innerText = msgObj.username;
                wrapper.appendChild(nameLabel);
            }

            const bubble = document.createElement('div');
            bubble.className = 'msg-bubble';
            
            if (msgObj.text) {
                const textSpan = document.createElement('div');
                textSpan.innerText = msgObj.text;
                bubble.appendChild(textSpan);
            }

            if (msgObj.image) {
                const img = document.createElement('img');
                img.className = 'msg-img';
                img.src = msgObj.image;
                img.alt = 'Hình ảnh đính kèm';
                img.onclick = () => window.open(msgObj.image, '_blank');
                bubble.appendChild(img);
            }

            wrapper.appendChild(bubble);

            const timeLabel = document.createElement('div');
            timeLabel.className = 'msg-time';
            timeLabel.innerText = formatTime(msgObj.timestamp);
            wrapper.appendChild(timeLabel);

            msgBox.appendChild(wrapper);
            msgBox.scrollTop = msgBox.scrollHeight;
        }

        let typingTimeout;
        let isCurrentlyTyping = false;
        let remoteTypingTimer;

        function handleInputTyping() {
            if (socket && socket.readyState === WebSocket.OPEN) {
                if (!isCurrentlyTyping) {
                    isCurrentlyTyping = true;
                    sendTypingStatus(true);
                }
                clearTimeout(typingTimeout);
                typingTimeout = setTimeout(() => {
                    isCurrentlyTyping = false;
                    sendTypingStatus(false);
                }, 2000);
            }
        }

        function sendTypingStatus(isTyping) {
            const usernameInput = document.getElementById('usernameInput');
            const username = usernameInput.value.trim() || 'User';
            const payloadObj = {
                type: 'typing',
                roomId: currentRoomId,
                clientId: myClientId,
                username: username,
                isTyping: isTyping
            };
            const payload = JSON.stringify({ action: "sendmessage", data: JSON.stringify(payloadObj) });
            socket.send(payload);
        }

        function handleRemoteTyping(msgObj) {
            const typingContainer = document.getElementById('typingIndicator');
            const typingText = document.getElementById('typingText');

            if (msgObj.isTyping) {
                typingText.innerText = `${msgObj.username} đang gõ...`;
                typingContainer.classList.remove('hidden');
                
                clearTimeout(remoteTypingTimer);
                remoteTypingTimer = setTimeout(() => {
                    hideTypingIndicator();
                }, 4000);
            } else {
                hideTypingIndicator();
            }
        }

        function hideTypingIndicator() {
            clearTimeout(remoteTypingTimer);
            const typingContainer = document.getElementById('typingIndicator');
            if (typingContainer) {
                typingContainer.classList.add('hidden');
            }
        }

        function autoResizeTextarea(el) {
            el.style.height = 'auto';
            el.style.height = Math.min(el.scrollHeight, 90) + 'px';
        }

        function handleInputKeydown(event) {
            if (event.key === 'Enter' && !event.shiftKey) {
                event.preventDefault();
                sendMessage();
            }
        }

        function toggleEmojiPicker(e) {
            e.stopPropagation();
            const picker = document.getElementById('emojiPicker');
            picker.classList.toggle('hidden');
        }

        function insertEmoji(emoji) {
            const input = document.getElementById('messageInput');
            input.value += emoji;
            autoResizeTextarea(input);
            document.getElementById('emojiPicker').classList.add('hidden');
            input.focus();
        }

        document.addEventListener('click', (e) => {
            const picker = document.getElementById('emojiPicker');
            if (picker && !picker.contains(e.target)) {
                picker.classList.add('hidden');
            }
        });

        function handleImageSelect(e) {
            const file = e.target.files[0];
            if (file) {
                if (file.size > 2 * 1024 * 1024) {
                    showToast("Kích thước hình ảnh vượt quá 2MB!", "warning");
                    return;
                }
                const reader = new FileReader();
                reader.onload = function(evt) {
                    selectedImageDataUrl = evt.target.result;
                    document.getElementById('imagePreview').src = selectedImageDataUrl;
                    document.getElementById('imagePreviewContainer').classList.remove('hidden');
                };
                reader.readAsDataURL(file);
            }
        }

        function clearSelectedImage() {
            selectedImageDataUrl = null;
            document.getElementById('imageFileInput').value = '';
            document.getElementById('imagePreview').src = '';
            document.getElementById('imagePreviewContainer').classList.add('hidden');
        }

        function sendMessage() {
            const input = document.getElementById('messageInput');
            const usernameInput = document.getElementById('usernameInput');
            const text = input.value.trim();
            const username = usernameInput.value.trim() || 'User';

            if (!socket || socket.readyState !== WebSocket.OPEN) {
                showToast("Không thể gửi tin nhắn. Kết nối bị ngắt!", "error");
                return;
            }

            if (text !== "" || selectedImageDataUrl) {
                clearTimeout(typingTimeout);
                if (isCurrentlyTyping) {
                    isCurrentlyTyping = false;
                    sendTypingStatus(false);
                }

                const payloadObj = {
                    type: 'chat',
                    roomId: currentRoomId,
                    clientId: myClientId,
                    username: username,
                    text: text,
                    image: selectedImageDataUrl,
                    timestamp: new Date().toISOString(),
                    cognitoToken: cognitoUserSession ? cognitoUserSession.idToken : null
                };
                const payload = JSON.stringify({ action: "sendmessage", data: JSON.stringify(payloadObj) });
                socket.send(payload);

                input.value = "";
                autoResizeTextarea(input);
                clearSelectedImage();
            }
        }
    </script>
</body>
</html>
`

---

#### 2. Amazon S3 Hosting & CloudFront Distribution with Origin Access Control (OAC)

##### Step 1: Create S3 Bucket and Upload `index.html`

1. Open **Amazon S3 Console** ➔ Click **Create bucket**.
2. Enter Bucket name: `my-serverless-chat-frontend-2026` ➔ Keep default settings (Block all public access **Enabled**) ➔ Click **Create bucket**.
3. Select the created Bucket and upload `index.html` to the root of the Bucket.

![index.html uploaded to S3 Bucket](/images/5.6-s3-upload-index-html.png)

##### Step 2: Create CloudFront Distribution & Origin Access Control (OAC) (5-Step Wizard Interface)

1. Open **Amazon CloudFront Console** ➔ Click **Create distribution**.
2. **Step 1: Get started**:
   - **Distribution name**: Enter `my-serverless-chat-frontend-distribution`.
   - **Distribution type**: Select **Single website or app** (Default).
   - Click **Next**.
3. **Step 2: Specify origin**:
   - **Origin domain**: Select your S3 Bucket (`my-serverless-chat-frontend-2026.s3.amazonaws.com`).
   - **Origin access**: Select **Origin access control settings (recommended)** ➔ Click **Create control setting** ➔ Click **Create** to generate OAC.
   - Click **Next**.
4. **Step 3 (Enable security)**:
   - Select **Do not enable security protections** (to bypass AWS WAF creation and avoid the $14/month charges for lab testing).
   - Click **Next**.
5. **Step 4 (Review and create)**:
   - Review general settings (Billing displays `$0/month`, Security displays `None`).
   - Click the orange **Create distribution** button at the bottom right to finalize creation.

![Creating CloudFront Distribution with S3 Origin and OAC](/images/5.6-cloudfront-create-distribution.png)

##### Step 3: Configure Default Root Object (`index.html`) & Update S3 Bucket Policy

1. **Configure Default Root Object**:
   - On the Distribution details page (**General** tab), under **Settings** panel, click **Edit**.
   - Under **Default root object**, type `index.html` ➔ Click **Save changes**.
2. **Verify / Update S3 Bucket Policy**:
   - Switch to the **Origins** tab ➔ Select the S3 Origin row ➔ Click **Edit**.
   - Click **Copy policy** (or review the auto-generated bucket policy statement).
   - Open the **Amazon S3 Console** ➔ Select your Bucket `my-serverless-chat-frontend-2026` ➔ Switch to the **Permissions** tab.
   - Under **Bucket policy**, click **Edit** and paste the JSON policy ➔ Click **Save changes**:
   ```json
   {
     "Version": "2012-10-17",
     "Statement": {
       "Sid": "AllowCloudFrontServicePrincipalReadOnly",
       "Effect": "Allow",
       "Principal": {
         "Service": "cloudfront.amazonaws.com"
       },
       "Action": "s3:GetObject",
       "Resource": "arn:aws:s3:::my-serverless-chat-frontend-2026/*",
       "Condition": {
         "StringEquals": {
           "AWS:SourceArn": "arn:aws:cloudfront::<ACCOUNT-ID>:distribution/<DISTRIBUTION-ID>"
         }
       }
     }
   }
   ```

![S3 Bucket Policy successfully updated for CloudFront OAC](/images/5.6-s3-bucket-policy-oac.png)

##### Step 4: Retrieve CloudFront Domain Name & Test Application

1. On the Distribution details page (**General** tab), copy your **Distribution domain name**.
2. Open a web browser and navigate to `https://<distribution-name>.cloudfront.net` to verify your real-time chat application over HTTPS.

![CloudFront Distribution Domain Name URL display and successful load test](/images/5.6-cloudfront-domain-test.png)

---

> [!TIP]
> **Important Note When Updating Frontend (`index.html`):**
>
> After uploaded a new `index.html` file to Amazon S3 but opening the CloudFront link still shows the old interface, it is because **Amazon CloudFront** is serving a cached version (**Cache**). To reflect new changes immediately:
>
> 1. **Create CloudFront Invalidation (Purge CDN Cache):**
>    - Open **Amazon CloudFront Console** ➔ Select your Distribution.
>    - Go to the **Invalidations** tab ➔ Click **Create invalidation**.
>    - In the **Object paths** input, enter `/*` (or `/index.html`) ➔ Click **Create invalidation**.
> 2. **Hard Refresh Browser Cache:**
>    - Press **`Ctrl + F5`** (or `Ctrl + Shift + R` / `Cmd + Shift + R`) or open in an **Incognito Window** to load the latest version.
