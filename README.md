<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
<title>Modern Chat Room</title>
<style>
  :root {
    --bg-body: #202225;
    --bg-chat: #36393f;
    --bg-header: #2f3136;
    --bg-input: #40444b;
    --primary: #5865F2;
    --primary-hover: #4752c4;
    --danger: #ed4245;
    --success: #3ba55c;
    --text-main: #dcddde;
    --text-muted: #72767d;
    --mention-bg: rgba(250, 168, 26, 0.1);
    --mention-text: #faa61a;
    --msg-hover: #32353b;
    --border: #202225;
    --shadow: 0 2px 10px 0 rgba(0,0,0,0.2);
    
    /* Role Colors */
    --role-owner: #ff0000;   /* RED */
    --role-admin: #ff9900;   /* ORANGE */
    --role-member: #00ff00;  /* GREEN */
  }

  * { box-sizing: border-box; outline: none; }
  
  body {
    margin: 0;
    padding: 0;
    font-family: 'Segoe UI', 'Roboto', 'Helvetica', sans-serif;
    background-color: var(--bg-body);
    color: var(--text-main);
    height: 100vh;
    display: flex;
    justify-content: center;
    overflow: hidden;
  }

  /* --- Utility Classes --- */
  .hidden { display: none !important; }
  .flex { display: flex; }
  .flex-col { flex-direction: column; }
  .center { align-items: center; justify-content: center; }
  
  /* --- Buttons & Inputs --- */
  button {
    cursor: pointer; border: none; font-family: inherit; transition: all 0.2s;
  }
  
  input, select, textarea {
    background: var(--bg-input); border: none; color: var(--text-main);
    padding: 10px; border-radius: 4px; font-size: 14px;
    font-family: inherit;
  }
  input:focus, textarea:focus { border-bottom: 2px solid var(--primary); }
  select { appearance: none; cursor: pointer; }

  .btn-primary {
    background: var(--primary); color: white; padding: 10px 20px;
    border-radius: 4px; font-weight: 600;
  }
  .btn-primary:hover { background: var(--primary-hover); }
  
  .btn-danger { background: var(--danger); color: white; padding: 8px 16px; border-radius: 4px; font-size: 13px; }
  .btn-danger:hover { background: #c93b3e; }

  .btn-icon {
    background: transparent; color: var(--text-muted); padding: 8px;
    border-radius: 4px;
  }
  .btn-icon:hover { background: rgba(255,255,255,0.1); color: var(--text-main); }

  .link-btn {
    background: none; border: none; color: var(--primary); text-decoration: underline;
    font-size: 12px; padding: 0; cursor: pointer;
  }
  .link-btn:hover { color: var(--primary-hover); }

  /* --- Auth Container --- */
  #authContainer {
    width: 100%; max-width: 400px; margin-top: 5vh;
    background: var(--bg-chat); padding: 30px; border-radius: 8px;
    box-shadow: var(--shadow);
    animation: slideUp 0.3s ease-out;
  }

  .auth-toggle {
    display: flex; margin-bottom: 20px; border-bottom: 2px solid var(--bg-input);
  }
  .auth-toggle button {
    flex: 1; background: transparent; color: var(--text-muted);
    padding: 10px; font-weight: 600; border-radius: 4px 4px 0 0;
  }
  .auth-toggle button.active {
    background: var(--bg-chat); color: var(--primary);
    border-bottom: 2px solid var(--primary); margin-bottom: -2px;
  }

  .field { margin-bottom: 15px; }
  .field label { display: block; margin-bottom: 5px; color: var(--text-muted); font-size: 12px; text-transform: uppercase; font-weight: bold; }
  .field input { width: 100%; }

  /* --- Chat Container --- */
  #chatRoom {
    width: 100%; max-width: 1000px; height: 100%;
    background: var(--bg-chat);
    display: flex; flex-direction: column;
    position: relative;
    box-shadow: var(--shadow);
  }

  /* Header */
  #chatHeader {
    height: 60px; background: var(--bg-header);
    display: flex; align-items: center; justify-content: space-between;
    padding: 0 16px; border-bottom: 1px solid #202225;
    flex-shrink: 0;
  }
  .header-left { display: flex; align-items: center; gap: 10px; }
  .header-title { font-weight: bold; font-size: 16px; }
  .role-tag { 
    font-size: 10px; padding: 2px 6px; border-radius: 4px; 
    background: rgba(0,0,0,0.3); text-transform: uppercase; margin-left: 8px;
    letter-spacing: 0.5px;
  }

  /* Sidebar */
  #sidebarPanel {
    position: absolute; top: 60px; left: 0; bottom: 0; width: 260px;
    background: var(--bg-header); z-index: 100;
    transform: translateX(-100%); transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    display: flex; flex-direction: column; border-right: 1px solid #202225;
  }
  #sidebarPanel.open { transform: translateX(0); }
  .sidebar-overlay {
    position: absolute; top: 60px; left: 0; right: 0; bottom: 0;
    background: rgba(0,0,0,0.5); z-index: 90; opacity: 0; pointer-events: none; transition: opacity 0.3s;
  }
  .sidebar-overlay.open { opacity: 1; pointer-events: auto; }

  .sidebar-section { padding: 15px; border-bottom: 1px solid rgba(0,0,0,0.2); }
  .sidebar-section h4 { margin: 0 0 10px 0; color: var(--text-muted); font-size: 12px; text-transform: uppercase; }
  .sidebar-item {
    padding: 8px; border-radius: 4px; color: var(--text-muted); cursor: pointer;
    display: flex; align-items: center; gap: 8px; font-size: 14px;
  }
  .sidebar-item:hover { background: rgba(255,255,255,0.05); color: var(--text-main); }
  .sidebar-divider { height: 1px; background: #202225; margin: 5px 0; }

  /* Messages Area */
  #messages {
    flex: 1; overflow-y: auto; padding: 20px;
    display: flex; flex-direction: column; gap: 4px;
    scroll-behavior: smooth;
  }

  /* Message Bubbles */
  .msg-row {
    display: flex; align-items: flex-start; padding: 8px 12px;
    border-radius: 4px; margin-top: 8px; position: relative;
    transition: background 0.1s;
  }
  .msg-row:hover { background: var(--msg-hover); }
  .msg-row.system-msg { 
    justify-content: center; background: transparent !important; 
    color: var(--text-muted); font-size: 12px; font-style: italic;
    padding: 5px;
  }
  .msg-row.admin-msg {
    justify-content: center; color: var(--danger); font-weight: bold;
    background: rgba(237, 66, 69, 0.1) !important; border: 1px solid var(--danger);
  }

  .avatar {
    width: 40px; height: 40px; border-radius: 50%; background: var(--primary);
    color: white; display: flex; align-items: center; justify-content: center;
    font-weight: bold; margin-right: 12px; flex-shrink: 0; font-size: 16px;
  }
  
  .msg-content-box { flex: 1; min-width: 0; }
  .msg-header { display: flex; align-items: baseline; gap: 8px; margin-bottom: 4px; }
  .username { font-weight: 600; cursor: pointer; }
  .timestamp { font-size: 10px; color: var(--text-muted); }
  .msg-text { line-height: 1.4; word-wrap: break-word; white-space: pre-wrap; }

  /* Roles in Chat */
  .role-owner { color: var(--role-owner); }
  .role-admin { color: var(--role-admin); }
  .role-member { color: var(--role-member); }

  /* Mention Highlighting */
  .mention {
    background: var(--mention-bg); color: var(--mention-text);
    padding: 0 2px; border-radius: 3px; cursor: pointer; font-weight: 500;
  }
  .mention:hover { text-decoration: underline; background: rgba(250, 168, 26, 0.2); }

  /* Reply Preview */
  #replyPreviewBar {
    padding: 8px 16px; background: #2f3136; border-top: 1px solid #202225;
    display: none; align-items: center; gap: 10px;
  }
  #replyPreviewBar.active { display: flex; }
  .reply-line { width: 2px; height: 20px; background: var(--primary); border-radius: 2px; }
  .reply-text { font-size: 12px; color: var(--text-muted); flex: 1; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
  .reply-text strong { color: var(--primary); }

  /* Input Area */
  #chatInputArea {
    padding: 0 16px 20px 16px; background: var(--bg-chat);
    display: flex; align-items: flex-end; gap: 10px;
    flex-shrink: 0; position: relative;
  }
  #chatInput {
    flex: 1; background: var(--bg-input); border-radius: 8px;
    padding: 12px; max-height: 150px; overflow-y: auto;
  }

  /* Emoji Picker */
  #emojiPicker {
    position: absolute; bottom: 70px; left: 16px;
    background: #2f3136; border: 1px solid #202225;
    border-radius: 8px; padding: 10px;
    display: grid; grid-template-columns: repeat(6, 1fr); gap: 5px;
    box-shadow: 0 5px 20px rgba(0,0,0,0.5); z-index: 50;
  }
  .emoji-btn {
    font-size: 20px; cursor: pointer; padding: 5px;
    border-radius: 4px; text-align: center; transition: background 0.2s;
  }
  .emoji-btn:hover { background: rgba(255,255,255,0.1); }
  
  /* Modals */
  .modal-overlay {
    position: fixed; top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(0,0,0,0.7); z-index: 2000; display: flex;
    align-items: center; justify-content: center; opacity: 0; pointer-events: none;
    transition: opacity 0.2s;
  }
  .modal-overlay.show { opacity: 1; pointer-events: auto; }
  
  .modal {
    background: var(--bg-chat); width: 90%; max-width: 500px;
    border-radius: 8px; padding: 24px; box-shadow: 0 10px 25px rgba(0,0,0,0.5);
    transform: scale(0.9); transition: transform 0.2s;
    display: flex; flex-direction: column; max-height: 90vh;
  }
  .modal-overlay.show .modal { transform: scale(1); }
  .modal h2 { margin-top: 0; color: var(--text-main); flex-shrink: 0; }
  .modal p { color: var(--text-muted); line-height: 1.5; }
  
  /* Suggestion List */
  #suggestionsList {
    margin-top: 15px;
    border-top: 1px solid #444;
    padding-top: 10px;
    overflow-y: auto;
    flex: 1;
    min-height: 100px;
  }
  .suggestion-item {
    background: rgba(0,0,0,0.2);
    padding: 10px;
    border-radius: 4px;
    margin-bottom: 8px;
    position: relative;
  }
  .suggestion-header {
    display: flex; justify-content: space-between; font-size: 11px; color: var(--text-muted); margin-bottom: 4px;
  }
  .suggestion-text { font-size: 13px; color: var(--text-main); }

  /* User List */
  .user-item {
    display: flex; align-items: center; justify-content: space-between;
    padding: 8px; border-radius: 4px;
  }
  .user-item:hover { background: rgba(255,255,255,0.05); }
  .user-status { width: 8px; height: 8px; border-radius: 50%; margin-right: 8px; }
  .status-on { background: var(--success); box-shadow: 0 0 5px var(--success); }
  .status-off { background: var(--text-muted); }

  /* User Action Modal Specifics */
  .action-grid {
    display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 15px;
  }
  .duration-group {
    display: flex; gap: 5px; margin-bottom: 10px;
  }
  .duration-group input { flex: 2; }
  .duration-group select { flex: 1; }
  .divider-line { height: 1px; background: #444; margin: 15px 0; }
  .modal-section-title {
    color: var(--text-muted); font-size: 12px; text-transform: uppercase; letter-spacing: 1px; margin-bottom: 8px; font-weight: bold;
  }

  /* Toast Notifications */
  #toastContainer {
    position: fixed; bottom: 20px; right: 20px; z-index: 3000;
    display: flex; flex-direction: column; gap: 10px;
  }
  .toast {
    background: var(--bg-chat); color: var(--text-main); padding: 12px 20px;
    border-radius: 4px; border-left: 4px solid var(--primary);
    box-shadow: 0 5px 15px rgba(0,0,0,0.3);
    animation: slideInRight 0.3s forwards; font-size: 14px;
    display: flex; align-items: center; gap: 10px; min-width: 250px;
  }
  .toast.success { border-color: var(--success); }
  .toast.error { border-color: var(--danger); }
  .toast.info { border-color: var(--primary); }

  /* Animations */
  @keyframes slideUp { from { transform: translateY(20px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
  @keyframes slideInRight { from { transform: translateX(100%); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
  @keyframes fadeOut { to { opacity: 0; transform: translateY(10px); } }

  /* Custom Scrollbar */
  ::-webkit-scrollbar { width: 8px; }
  ::-webkit-scrollbar-track { background: #2e3338; }
  ::-webkit-scrollbar-thumb { background: #202225; border-radius: 4px; }
  ::-webkit-scrollbar-thumb:hover { background: #18191c; }

</style>
</head>
<body>

<!-- Audio Element for Notification (Ding Sound) -->
<audio id="pingSound" src="data:audio/wav;base64,UklGRl9vT1BXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YU"></audio>

<!-- Toast Container -->
<div id="toastContainer"></div>

<!-- Authentication Screen -->
<div id="authContainer">
  <div class="auth-toggle">
    <button id="btnShowSignup" class="active">Sign Up</button>
    <button id="btnShowLogin">Login</button>
  </div>

  <form id="signupForm">
    <h2 style="margin: 0 0 20px 0;">Create Account</h2>
    <div class="field">
      <label>Username</label>
      <input type="text" id="signupUsername" placeholder="Enter username" required />
    </div>
    <div class="field">
      <label>Email</label>
      <input type="email" id="signupEmail" placeholder="Enter email" required />
    </div>
    <div class="field">
      <label>Password</label>
      <input type="password" id="signupPassword" placeholder="Enter password" required />
    </div>
    <div class="field">
      <label style="font-weight: normal; text-transform: none;">
        <input type="checkbox" id="agreeTerms" required style="width: auto; margin-right: 5px;" />
        I agree to the terms
      </label>
    </div>
    <button type="submit" class="btn-primary" style="width: 100%">Sign Up</button>
  </form>

  <form id="loginForm" class="hidden">
    <h2 style="margin: 0 0 20px 0;">Welcome Back</h2>
    <div class="field">
      <label>Username</label>
      <input type="text" id="loginUsername" placeholder="Enter username" required />
    </div>
    <div class="field">
      <label>Password</label>
      <input type="password" id="loginPassword" placeholder="Enter password" required />
    </div>
    <div style="margin-top: 10px; text-align: center;">
      <button type="button" id="forgotPassBtn" class="link-btn">Forgot Password?</button>
    </div>
    <button type="submit" class="btn-primary" style="width: 100%; margin-top: 15px;">Login</button>
  </form>
</div>

<!-- Chat Room -->
<div id="chatRoom" class="hidden">
  
  <!-- Sidebar Overlay -->
  <div id="sidebarOverlay" class="sidebar-overlay"></div>

  <!-- Sidebar Panel -->
  <div id="sidebarPanel">
    <div class="sidebar-section">
      <h4>Menu</h4>
      <div class="sidebar-item" id="menuRules">
        <span>📜</span> Rules
      </div>
      <div class="sidebar-item" id="menuSuggestions">
        <span>💡</span> Suggestions
      </div>
      <div class="sidebar-item" id="menuUsers">
        <span>👥</span> User List
      </div>
      <div class="sidebar-divider"></div>
      <div class="sidebar-item" id="menuLogout" style="color: var(--danger);">
        <span>🚪</span> Logout
      </div>
    </div>

    <!-- Admin Panel (Only visible to Admin/Owner) -->
    <div id="adminPanel" class="sidebar-section hidden">
      <h4>Administration</h4>
      <div class="sidebar-item" id="menuClearChat">
        <span>🧹</span> Clear Chat
      </div>
      <div class="sidebar-divider"></div>
      <!-- Quick Action -->
      <div style="padding: 0 8px;">
        <h4>Quick Action</h4>
        <input type="text" id="roleTarget" placeholder="Target username" style="width: 100%; margin-bottom: 5px;" />
        <select id="roleSelect" style="width: 100%; background: var(--bg-input); color: white; border: none; padding: 8px; border-radius: 4px; margin-bottom: 5px;">
          <option value="member">Member</option>
          <option value="admin">Admin</option>
          <option value="owner">Owner</option>
        </select>
        <button id="btnAssignRole" class="btn-primary" style="width: 100%; padding: 5px; font-size: 12px;">Apply Role</button>
      </div>
    </div>
  </div>

  <!-- Header -->
  <header id="chatHeader">
    <div class="header-left">
      <button id="menuToggle" class="btn-icon">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <line x1="3" y1="12" x2="21" y2="12"></line>
          <line x1="3" y1="6" x2="21" y2="6"></line>
          <line x1="3" y1="18" x2="21" y2="18"></line>
        </svg>
      </button>
      <div class="header-title">
        General Chat
        <span id="myRoleTag" class="role-tag">Member</span>
      </div>
    </div>
  </header>

  <!-- Messages -->
  <div id="messages"></div>

  <!-- Reply Preview -->
  <div id="replyPreviewBar">
    <div class="reply-line"></div>
    <div class="reply-text" id="replyTextContent"></div>
    <button id="cancelReplyBtn" class="btn-icon" style="padding: 4px;">✕</button>
  </div>

  <!-- Input Area -->
  <div id="chatInputArea">
    <!-- Emoji Picker (Hidden by default) -->
    <div id="emojiPicker" class="hidden">
      <!-- Emojis injected by JS -->
    </div>

    <input id="chatInput" type="text" placeholder="Message #general" autocomplete="off" />
    
    <!-- Emoji Button -->
    <button id="emojiBtn" class="btn-icon" title="Emoji">
      <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <circle cx="12" cy="12" r="10"></circle>
        <path d="M8 14s1.5 2 4 2 4-2 4-2"></path>
        <line x1="9" y1="9" x2="9.01" y2="9"></line>
        <line x1="15" y1="9" x2="15.01" y2="9"></line>
      </svg>
    </button>

    <button id="sendBtn" class="btn-primary" style="padding: 10px 14px; border-radius: 50%;">
      <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <line x1="22" y1="2" x2="11" y2="13"></line>
        <polygon points="22 2 15 22 11 13 2 9 22 2"></polygon>
      </svg>
    </button>
  </div>
</div>

<!-- Modals -->
<div id="rulesModal" class="modal-overlay">
  <div class="modal">
    <h2>Chat Rules</h2>
    <p>1. Be respectful to all members.</p>
    <p>2. No spamming or flooding the chat.</p>
    <p>3. Listen to Admins and Owners.</p>
    <p>4. Have fun!</p>
    <button class="btn-primary" style="margin-top: 15px; width: 100%" onclick="closeModals()">Got it</button>
  </div>
</div>

<!-- Password Reset Step 1: Request Code -->
<div id="forgotPassModal" class="modal-overlay">
  <div class="modal">
    <h2>Reset Password</h2>
    <p>Enter your email address. We will send you a verification code.</p>
    <div class="field">
      <label>Email Address</label>
      <input type="email" id="resetEmailInput" placeholder="you@example.com" required />
    </div>
    <button class="btn-primary" style="width: 100%" onclick="requestPasswordReset()">Send Code</button>
    <button class="btn-icon" style="margin-top:10px; width:100%; color:var(--text-muted);" onclick="closeModals()">Cancel</button>
  </div>
</div>

<!-- Password Reset Step 2: Verify Code -->
<div id="verifyCodeModal" class="modal-overlay">
  <div class="modal">
    <h2>Enter Verification Code</h2>
    <p>We sent a code to <span id="resetEmailDisplay" style="color:var(--primary); font-weight:bold;"></span>.</p>
    <div class="field">
      <label>6-Digit Code</label>
      <input type="text" id="resetCodeInput" placeholder="123456" maxlength="6" style="letter-spacing: 5px; font-size: 20px; text-align: center;" required />
    </div>
    <div class="field">
      <label>New Password</label>
      <input type="password" id="resetNewPassInput" placeholder="New Password" required />
    </div>
    <button class="btn-primary" style="width: 100%" onclick="confirmPasswordReset()">Change Password</button>
    <button class="btn-icon" style="margin-top:10px; width:100%; color:var(--text-muted);" onclick="closeModals()">Cancel</button>
  </div>
</div>

<!-- Suggestions Modal -->
<div id="suggestionsModal" class="modal-overlay">
  <div class="modal">
    <h2>Suggestion Box</h2>
    <p style="margin-bottom: 10px;">Have an idea to improve the chat? Let us know below.</p>
    
    <div class="field">
      <textarea id="suggestionInput" rows="3" placeholder="Type your suggestion here..." style="width: 100%; resize: vertical;"></textarea>
    </div>
    <button class="btn-primary" style="width: 100%" onclick="submitSuggestion()">Submit Suggestion</button>

    <!-- Admin View Area -->
    <div id="suggestionsList" class="hidden">
      <h4 style="margin: 10px 0; color: var(--text-muted); font-size: 12px; text-transform: uppercase;">Previous Suggestions</h4>
      <!-- Suggestions injected here -->
    </div>

    <div class="divider-line"></div>
    <button class="btn-primary" style="width: 100%" onclick="closeModals()">Close</button>
  </div>
</div>

<!-- User List Modal -->
<div id="userListModal" class="modal-overlay">
  <div class="modal">
    <h2>Online Users</h2>
    <div id="userListDisplay" style="max-height: 300px; overflow-y: auto; margin-bottom: 15px;">
      <!-- Populated via JS -->
    </div>
    <button class="btn-primary" style="width: 100%" onclick="closeModals()">Close</button>
  </div>
</div>

<!-- User Action Modal -->
<div id="userActionModal" class="modal-overlay">
  <div class="modal">
    <h2 id="actionModalTitle">Manage User</h2>
    
    <!-- Admin Only Content -->
    <div id="adminActionContent">
      <div class="modal-section-title">Moderation</div>
      
      <div class="duration-group">
        <input type="number" id="modDurationVal" placeholder="10" min="1" />
        <select id="modDurationUnit">
          <option value="m">Minutes</option>
          <option value="h">Hours</option>
          <option value="d">Days</option>
        </select>
      </div>

      <div class="action-grid">
        <button class="btn-danger" onclick="executeModAction('mute')">Mute</button>
        <button class="btn-danger" onclick="executeModAction('kick')">Kick</button>
        <button class="btn-danger" style="grid-column: span 2;" onclick="executeModAction('ban')">Ban</button>
      </div>

      <div class="divider-line"></div>

      <div class="modal-section-title">Account Settings</div>
      <div class="field">
        <label>Change Username</label>
        <div style="display:flex; gap:5px;">
          <input type="text" id="newUsernameInput" placeholder="New name..." />
          <button class="btn-primary" style="padding: 10px;" onclick="executeRename()">Change</button>
        </div>
      </div>
    </div>

    <!-- Member/No Perm Content -->
    <div id="memberActionContent" class="hidden">
      <p style="text-align: center; color: var(--text-muted);">
        You do not have permission to manage this user.
      </p>
    </div>

    <div class="divider-line"></div>
    <button class="btn-primary" style="width: 100%" onclick="closeModals()">Close</button>
  </div>
</div>

<script>
  /** 
   * STATE MANAGEMENT & CONSTANTS 
   */
  const ROLES = {
    owner: { name: "Owner", color: "#ff0000", priority: 3 },    // RED
    admin: { name: "Admin", color: "#ff9900", priority: 2 },    // ORANGE
    member: { name: "Member", color: "#00ff00", priority: 1 }   // GREEN
  };

  const DB_KEYS = {
    USERS: 'chat_users_v2',
    MESSAGES: 'chat_messages_v2',
    MUTES: 'chat_mutes_v2',
    BANS: 'chat_bans_v2',
    KICKS: 'chat_kicks_v2',
    SESSION: 'chat_session_username_v2',
    SUGGESTIONS: 'chat_suggestions_v2',
    RESET_CODES: 'chat_reset_codes' // New key for password reset
  };

  // Common Emojis
  const EMOJIS = [
    '😀','😂','😍','🤔','😎','😭','😡','👍','👎','🎉',
    '🔥','❤️','💀','🤣','😴','👀','🙄','💯','🚀','✅',
    '❌','⚠️','👋','🤝','🤡','👻','💩','👽','🤖','👺'
  ];

  let state = {
    users: JSON.parse(localStorage.getItem(DB_KEYS.USERS)) || {},
    messages: JSON.parse(localStorage.getItem(DB_KEYS.MESSAGES)) || [],
    muted: JSON.parse(localStorage.getItem(DB_KEYS.MUTES)) || {},
    banned: JSON.parse(localStorage.getItem(DB_KEYS.BANS)) || {},
    kicked: JSON.parse(localStorage.getItem(DB_KEYS.KICKS)) || {},
    suggestions: JSON.parse(localStorage.getItem(DB_KEYS.SUGGESTIONS)) || [],
    resetCodes: JSON.parse(localStorage.getItem(DB_KEYS.RESET_CODES)) || {},
    currentUser: localStorage.getItem(DB_KEYS.SESSION) || null,
    replyingTo: null,
    actionTarget: null,
    lastMessageId: 0,
    resetEmailTemp: null // Temporary holder during reset flow
  };

  let muteInterval = null;

  /**
   * UTILITY FUNCTIONS
   */
  function escapeHtml(text) {
    if (!text) return "";
    return text
      .replace(/&/g, "&amp;")
      .replace(/</g, "&lt;")
      .replace(/>/g, "&gt;")
      .replace(/"/g, "&quot;")
      .replace(/'/g, "&#039;");
  }

  function showToast(message, type = 'info', duration = 3000) {
    const container = document.getElementById('toastContainer');
    const toast = document.createElement('div');
    toast.className = `toast ${type}`;
    
    let icon = 'ℹ️';
    if (type === 'success') icon = '✅';
    if (type === 'error') icon = '⛔';

    toast.innerHTML = `<span>${icon}</span> <span>${message}</span>`;
    container.appendChild(toast);

    setTimeout(() => {
      toast.style.animation = 'fadeOut 0.3s forwards';
      toast.addEventListener('animationend', () => toast.remove());
    }, duration);
  }

  function saveState() {
    localStorage.setItem(DB_KEYS.USERS, JSON.stringify(state.users));
    localStorage.setItem(DB_KEYS.MESSAGES, JSON.stringify(state.messages));
    localStorage.setItem(DB_KEYS.MUTES, JSON.stringify(state.muted));
    localStorage.setItem(DB_KEYS.BANS, JSON.stringify(state.banned));
    localStorage.setItem(DB_KEYS.KICKS, JSON.stringify(state.kicked));
    localStorage.setItem(DB_KEYS.SUGGESTIONS, JSON.stringify(state.suggestions));
    localStorage.setItem(DB_KEYS.RESET_CODES, JSON.stringify(state.resetCodes));
  }

  function getUserRole(username) {
    return state.users[username]?.role || 'member';
  }

  function getUserColor(username) {
    const role = getUserRole(username);
    return ROLES[role]?.color || ROLES.member.color;
  }

  function isMod(username) {
    const role = getUserRole(username);
    return ['admin', 'owner'].includes(role);
  }

  function formatTime(ms) {
    if (ms === Infinity) return "Permanent";
    if (ms <= 0) return "Expired";
    const s = Math.floor((ms / 1000) % 60);
    const m = Math.floor((ms / (1000 * 60)) % 60);
    const h = Math.floor((ms / (1000 * 60 * 60)));
    return `${h}h ${m}m ${s}s`;
  }

  /**
   * PASSWORD RESET LOGIC
   */
  
  // Step 1: Generate Code and "Send" it
  function requestPasswordReset() {
    const email = document.getElementById('resetEmailInput').value.trim().toLowerCase();
    
    if (!email) return showToast('Please enter your email.', 'error');

    // Find user by email
    let username = null;
    for (const [u, data] of Object.entries(state.users)) {
      if (data.email && data.email.toLowerCase() === email) {
        username = u;
        break;
      }
    }

    if (!username) return showToast('No account found with that email.', 'error');

    // Generate 6-digit code
    const code = Math.floor(100000 + Math.random() * 900000).toString();
    const expiry = Date.now() + (5 * 60 * 1000); // 5 minutes

    state.resetCodes[email] = { code, expiry, username };
    saveState();

    // SIMULATED EMAIL SENDING
    // In a real app, you would call an API here (e.g., EmailJS).
    // Since this is a standalone HTML file, we show the code in a Toast.
    showToast(`Verification code sent to ${email}`, 'success', 8000);
    
    // Explicitly show the code so the user can proceed with the demo
    setTimeout(() => {
      alert(`DEMO MODE: Your verification code is ${code}`);
    }, 500);

    state.resetEmailTemp = email;
    closeModals();
    openVerifyModal();
  }

  function openVerifyModal() {
    const modal = document.getElementById('verifyCodeModal');
    document.getElementById('resetEmailDisplay').textContent = state.resetEmailTemp;
    document.getElementById('resetCodeInput').value = '';
    document.getElementById('resetNewPassInput').value = '';
    modal.classList.add('show');
  }

  // Step 2: Verify Code and Update Password
  function confirmPasswordReset() {
    const code = document.getElementById('resetCodeInput').value;
    const newPass = document.getElementById('resetNewPassInput').value;
    const email = state.resetEmailTemp;

    if (!code || !newPass) return showToast('Please fill in all fields.', 'error');

    const record = state.resetCodes[email];
    
    if (!record) return showToast('Invalid or expired request.', 'error');
    if (record.code !== code) return showToast('Incorrect verification code.', 'error');
    if (Date.now() > record.expiry) {
      delete state.resetCodes[email];
      saveState();
      return showToast('Code expired. Please request a new one.', 'error');
    }

    // Reset Password
    state.users[record.username].password = newPass;
    
    // Cleanup
    delete state.resetCodes[email];
    state.resetEmailTemp = null;
    saveState();

    showToast('Password changed successfully! Please login.', 'success');
    closeModals();
    showLogin(); // Switch back to login screen
  }

  window.requestPasswordReset = requestPasswordReset;
  window.confirmPasswordReset = confirmPasswordReset;


  /**
   * NOTIFICATION SOUND LOGIC
   */
  function playPingSound() {
    const audio = document.getElementById('pingSound');
    if(audio) {
      audio.currentTime = 0;
      audio.play().catch(e => console.log("Audio play failed (interaction needed):", e));
    }
  }

  function checkNotification(msg) {
    if (msg.sender === state.currentUser) return;

    const isMention = msg.content && msg.content.includes(`@${state.currentUser}`);
    const isReply = msg.replyTo && msg.replyTo.sender === state.currentUser;

    if (isMention || isReply) {
      playPingSound();
      const originalTitle = document.title;
      document.title = `(@Ping) ${originalTitle}`;
      setTimeout(() => document.title = originalTitle, 3000);
    }
  }

  /**
   * AUTH LOGIC
   */
  function initAuth() {
    if (state.currentUser) {
      if (!state.users[state.currentUser] || isBanned(state.currentUser)) {
        logout();
      } else {
        enterChat();
      }
    } else {
      showAuthScreen();
    }
  }

  function showAuthScreen() {
    document.getElementById('authContainer').classList.remove('hidden');
    document.getElementById('chatRoom').classList.add('hidden');
  }

  function showSignup() {
    document.getElementById('btnShowSignup').classList.add('active');
    document.getElementById('btnShowLogin').classList.remove('active');
    document.getElementById('signupForm').classList.remove('hidden');
    document.getElementById('loginForm').classList.add('hidden');
  }

  function showLogin() {
    document.getElementById('btnShowLogin').classList.add('active');
    document.getElementById('btnShowSignup').classList.remove('active');
    document.getElementById('loginForm').classList.remove('hidden');
    document.getElementById('signupForm').classList.add('hidden');
  }

  document.getElementById('btnShowSignup').onclick = showSignup;
  document.getElementById('btnShowLogin').onclick = showLogin;
  document.getElementById('forgotPassBtn').onclick = () => {
    closeModals(); // Ensure other modals are closed
    document.getElementById('forgotPassModal').classList.add('show');
  };

  function enterChat() {
    if (!state.currentUser) return;
    
    if (state.users[state.currentUser]) {
      state.users[state.currentUser].isOnline = true;
      saveState();
    }

    document.getElementById('authContainer').classList.add('hidden');
    document.getElementById('chatRoom').classList.remove('hidden');
    
    const currentRole = getUserRole(state.currentUser);
    const roleTag = document.getElementById('myRoleTag');
    roleTag.textContent = ROLES[currentRole].name;
    roleTag.style.color = ROLES[currentRole].color;
    
    updateUIForRole();
    renderMessages();
    checkMuteStatus();
    startMuteTimer();
    initEmojiPicker();
  }

  function logout() {
    if (state.currentUser && state.users[state.currentUser]) {
      state.users[state.currentUser].isOnline = false;
      saveState();
    }
    stopMuteTimer();
    localStorage.removeItem(DB_KEYS.SESSION);
    state.currentUser = null;
    location.reload();
  }

  /**
   * EMOJI PICKER LOGIC
   */
  function initEmojiPicker() {
    const picker = document.getElementById('emojiPicker');
    picker.innerHTML = '';
    EMOJIS.forEach(emoji => {
      const span = document.createElement('div');
      span.className = 'emoji-btn';
      span.textContent = emoji;
      span.onclick = () => {
        const input = document.getElementById('chatInput');
        input.value += emoji;
        input.focus();
        picker.classList.add('hidden');
      };
      picker.appendChild(span);
    });
  }

  document.getElementById('emojiBtn').onclick = (e) => {
    e.stopPropagation();
    document.getElementById('emojiPicker').classList.toggle('hidden');
  };

  document.addEventListener('click', (e) => {
    const picker = document.getElementById('emojiPicker');
    const btn = document.getElementById('emojiBtn');
    if (!picker.contains(e.target) && e.target !== btn) {
      picker.classList.add('hidden');
    }
  });

  /**
   * CHAT LOGIC
   */
  const messagesDiv = document.getElementById('messages');
  const chatInput = document.getElementById('chatInput');
  const replyPreviewBar = document.getElementById('replyPreviewBar');
  
  function renderMessages() {
    messagesDiv.innerHTML = '';
    state.messages.forEach(msg => renderSingleMessage(msg));
    scrollToBottom();
    if(state.messages.length > 0) {
        state.lastMessageId = state.messages[state.messages.length-1].id;
    }
  }

  function renderSingleMessage(msg) {
    const div = document.createElement('div');
    
    if (msg.type === 'system') {
      div.className = 'msg-row system-msg';
      div.textContent = msg.content;
      messagesDiv.appendChild(div);
      return;
    }
    
    if (msg.type === 'admin') {
      div.className = 'msg-row admin-msg';
      div.textContent = msg.content;
      messagesDiv.appendChild(div);
      return;
    }

    const isMe = msg.sender === state.currentUser;
    const role = getUserRole(msg.sender);
    const colorClass = `role-${role}`;
    const avatarLetter = msg.sender.charAt(0).toUpperCase();
    
    let safeContent = escapeHtml(msg.content);
    safeContent = safeContent.replace(/@(\w+)/g, (match, username) => {
      if (state.users[username]) {
        return `<span class="mention" onclick="insertAtCursor('@${username} ')">@${username}</span>`;
      }
      return match;
    });

    let replyHtml = '';
    if (msg.replyTo) {
      replyHtml = `
        <div style="font-size: 11px; color: var(--text-muted); margin-bottom: 2px; border-left: 2px solid var(--primary); padding-left: 6px;">
          <span style="font-weight:bold; color:${getUserColor(msg.replyTo.sender)}">${msg.replyTo.sender}</span>: 
          ${escapeHtml(msg.replyTo.content).substring(0, 30)}...
        </div>
      `;
    }

    div.className = 'msg-row';
    const isPinged = msg.content.includes(`@${state.currentUser}`) || (msg.replyTo && msg.replyTo.sender === state.currentUser);
    
    if (isPinged) {
        div.style.background = 'rgba(250, 168, 26, 0.1)';
    }

    div.innerHTML = `
      <div class="avatar" style="background: ${isMe ? '#5865F2' : '#4f545c'}">${avatarLetter}</div>
      <div class="msg-content-box">
        <div class="msg-header">
          <span class="username ${colorClass}">${msg.sender}</span>
          <span class="timestamp">${new Date(msg.timestamp).toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})}</span>
        </div>
        ${replyHtml}
        <div class="msg-text">${safeContent}</div>
      </div>
      <button class="btn-icon reply-trigger" style="opacity:0; margin-left: 5px;" title="Reply">
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 10h10a8 8 0 0 1 8 8v2M3 10l6 6m-6-6l6-6"/></svg>
      </button>
    `;

    div.addEventListener('mouseenter', () => div.querySelector('.reply-trigger').style.opacity = '1');
    div.addEventListener('mouseleave', () => div.querySelector('.reply-trigger').style.opacity = '0');

    div.querySelector('.reply-trigger').onclick = () => startReply(msg.sender, msg.content);

    messagesDiv.appendChild(div);
  }

  function addMessageToState(type, content, sender = 'System', replyTo = null) {
    const msg = {
      id: Date.now(),
      type,
      content,
      sender,
      timestamp: Date.now(),
      replyTo
    };
    state.messages.push(msg);
    if (state.messages.length > 100) state.messages.shift();
    saveState();
    renderSingleMessage(msg);
    scrollToBottom();
  }

  function scrollToBottom() {
    messagesDiv.scrollTop = messagesDiv.scrollHeight;
  }

  window.startReply = (sender, content) => {
    state.replyingTo = { sender, content };
    replyPreviewBar.classList.add('active');
    document.getElementById('replyTextContent').innerHTML = `Replying to <strong>${sender}</strong>: "${escapeHtml(content)}"`;
    chatInput.focus();
  };

  document.getElementById('cancelReplyBtn').onclick = () => {
    state.replyingTo = null;
    replyPreviewBar.classList.remove('active');
  };

  window.insertAtCursor = (text) => {
    chatInput.value += text;
    chatInput.focus();
  };

  document.getElementById('sendBtn').onclick = sendMessage;
  chatInput.addEventListener('keypress', (e) => { if (e.key === 'Enter') sendMessage(); });

  function sendMessage() {
    const text = chatInput.value.trim();
    if (!text) return;

    if (isMuted(state.currentUser)) {
      showToast(`You are muted for ${formatTime(state.muted[state.currentUser] - Date.now())}`, 'error');
      return;
    }

    if (text.startsWith('/')) {
      handleCommand(text);
      chatInput.value = '';
      cancelReplyUI();
      return;
    }

    addMessageToState('user', text, state.currentUser, state.replyingTo);
    chatInput.value = '';
    cancelReplyUI();
  }

  function cancelReplyUI() {
    state.replyingTo = null;
    replyPreviewBar.classList.remove('active');
  }

  /**
   * SUGGESTIONS LOGIC
   */
  function submitSuggestion() {
    const input = document.getElementById('suggestionInput');
    const text = input.value.trim();
    
    if (!text) return showToast('Please write a suggestion first.', 'info');
    
    const newSuggestion = {
      id: Date.now(),
      user: state.currentUser,
      text: text,
      date: new Date().toLocaleString()
    };

    state.suggestions.push(newSuggestion);
    saveState();
    
    input.value = '';
    showToast('Suggestion submitted! Thank you.', 'success');
    
    if (isMod(state.currentUser)) {
      renderSuggestionsList();
    }
  }

  function deleteSuggestion(id) {
    state.suggestions = state.suggestions.filter(s => s.id !== id);
    saveState();
    renderSuggestionsList();
    showToast('Suggestion deleted.', 'info');
  }

  function renderSuggestionsList() {
    const listContainer = document.getElementById('suggestionsList');
    listContainer.innerHTML = '';

    if (state.suggestions.length === 0) {
      listContainer.innerHTML = '<p style="color:var(--text-muted); text-align:center;">No suggestions yet.</p>';
      return;
    }

    const sorted = [...state.suggestions].sort((a, b) => b.id - a.id);

    sorted.forEach(s => {
      const item = document.createElement('div');
      item.className = 'suggestion-item';
      item.innerHTML = `
        <div class="suggestion-header">
          <span><strong>${escapeHtml(s.user)}</strong> • ${s.date}</span>
          ${isMod(state.currentUser) ? `<button class="btn-icon" onclick="deleteSuggestion(${s.id})" style="color:var(--danger); padding:2px;">✕</button>` : ''}
        </div>
        <div class="suggestion-text">${escapeHtml(s.text)}</div>
      `;
      listContainer.appendChild(item);
    });
  }

  window.submitSuggestion = submitSuggestion;
  window.deleteSuggestion = deleteSuggestion;

  /**
   * USER LIST & MODAL LOGIC
   */
  
  function openUserActionModal(username) {
    state.actionTarget = username;
    const modal = document.getElementById('userActionModal');
    const title = document.getElementById('actionModalTitle');
    const adminContent = document.getElementById('adminActionContent');
    const memberContent = document.getElementById('memberActionContent');

    title.textContent = `Manage User: ${username}`;
    document.getElementById('newUsernameInput').value = ''; 

    if (isMod(state.currentUser)) {
      adminContent.classList.remove('hidden');
      memberContent.classList.add('hidden');
    } else {
      adminContent.classList.add('hidden');
      memberContent.classList.remove('hidden');
    }

    modal.classList.add('show');
  }

  function executeModAction(actionType) {
    const target = state.actionTarget;
    if (!target) return;

    const val = parseInt(document.getElementById('modDurationVal').value);
    const unit = document.getElementById('modDurationUnit').value;

    if (!val || val <= 0) {
      showToast('Please enter a valid duration', 'error');
      return;
    }

    const durationStr = `${val}${unit}`; 

    applyMod(actionType, target, durationStr);
    closeModals();
  }

  function executeRename() {
    const oldName = state.actionTarget;
    const newName = document.getElementById('newUsernameInput').value.trim();

    if (!newName) return showToast('Username cannot be empty', 'error');
    if (state.users[newName]) return showToast('Username already taken', 'error');
    if (oldName === newName) return;

    const userData = state.users[oldName];
    
    state.users[newName] = { ...userData };

    delete state.users[oldName];

    if (oldName === state.currentUser) {
      state.currentUser = newName;
      localStorage.setItem(DB_KEYS.SESSION, newName);
      const currentRole = getUserRole(newName);
      const roleTag = document.getElementById('myRoleTag');
      roleTag.textContent = ROLES[currentRole].name;
      roleTag.style.color = ROLES[currentRole].color;
    }

    if (state.banned[oldName]) {
      state.banned[newName] = state.banned[oldName];
      delete state.banned[oldName];
    }
    if (state.muted[oldName]) {
      state.muted[newName] = state.muted[oldName];
      delete state.muted[oldName];
    }

    saveState();
    showToast(`User renamed to ${newName}`, 'success');
    addMessageToState('admin', `${oldName} changed their name to ${newName}`);
    
    renderUserList(); 
    closeModals();
  }

  window.executeRename = executeRename;
  window.executeModAction = executeModAction;
  window.openUserActionModal = openUserActionModal;

  function renderUserList() {
    const list = document.getElementById('userListDisplay');
    list.innerHTML = '';
    
    Object.keys(state.users).forEach(user => {
      const u = state.users[user];
      const role = ROLES[u.role];
      const item = document.createElement('div');
      item.className = 'user-item';
      
      const nameSpan = document.createElement('span');
      nameSpan.textContent = user;
      nameSpan.style.cursor = "pointer";
      nameSpan.style.textDecoration = "underline";
      nameSpan.style.fontWeight = "bold";
      nameSpan.style.color = role.color;
      nameSpan.onclick = () => openUserActionModal(user);

      item.innerHTML = `
        <div style="display:flex; align-items:center;">
          <div class="user-status ${u.isOnline ? 'status-on' : 'status-off'}"></div>
        </div>
        <span style="font-size:11px; color: #777;">${role.name}</span>
      `;
      
      item.insertBefore(nameSpan, item.children[1]);
      list.appendChild(item);
    });
  }

  /**
   * MODERATION & COMMANDS
   */
  function handleCommand(cmdStr) {
    const parts = cmdStr.trim().split(/\s+/);
    const cmd = parts[0].toLowerCase();
    const args = parts.slice(1);

    if (cmd === '/owner') {
      const pass = args.join(' ');
      if (pass === 'Oceanlife1') {
        setUserRole(state.currentUser, 'owner');
        showToast('You are now the Owner!', 'success');
      } else {
        showToast('Incorrect owner password.', 'error');
      }
      return;
    }
    
    if (cmd === '/admin') {
      const pass = args.join(' ');
      if (pass === 'cirbana') {
        setUserRole(state.currentUser, 'admin');
        showToast('You are now an Admin!', 'success');
      } else {
        showToast('Incorrect admin password.', 'error');
      }
      return;
    }

    if (['/kick', '/ban', '/mute', '/unban', '/unmute', '/clear'].includes(cmd)) {
      if (!isMod(state.currentUser)) {
        showToast('Only Admins/Owners can use moderation commands.', 'error');
        return;
      }
    }

    switch (cmd) {
      case '/kick': applyMod('kick', args[0], args[1]); break;
      case '/ban': applyMod('ban', args[0], args[1]); break;
      case '/mute': applyMod('mute', args[0], args[1]); break;
      case '/unmute': removeMod('unmute', args[0]); break;
      case '/unban': removeMod('unban', args[0]); break;
      case '/clear':
        state.messages = [];
        saveState();
        renderMessages();
        addMessageToState('admin', 'Chat cleared by ' + state.currentUser);
        showToast('Chat cleared', 'success');
        break;
      default:
        showToast('Unknown command', 'error');
    }
  }

  function applyMod(type, target, durationStr) {
    if (!isMod(state.currentUser)) {
      showToast('Permission Denied: You are not an Admin or Owner.', 'error');
      return;
    }

    if (!target) return showToast(`Usage: /${type} <user> [duration]`, 'info');
    if (!state.users[target]) return showToast('User not found', 'error');
    if (state.users[target].role === 'owner') return showToast('Cannot moderate Owner', 'error');

    let duration = 0; 
    let displayTime = "Permanently";

    if (durationStr) {
      const match = durationStr.match(/(\d+)([smhd])/i); 
      if (match) {
        const val = parseInt(match[1]);
        const unit = match[2].toLowerCase();
        const now = Date.now();
        if (unit === 's') duration = now + (val * 1000);
        if (unit === 'm') duration = now + (val * 60000);
        if (unit === 'h') duration = now + (val * 3600000);
        if (unit === 'd') duration = now + (val * 86400000);
        displayTime = durationStr;
      }
    }

    if (type === 'kick') {
      state.kicked[target] = duration || Date.now() + 60000; 
      state.users[target].isOnline = false;
      addMessageToState('admin', `${target} has been KICKED by ${state.currentUser}`);
    } else if (type === 'ban') {
      state.banned[target] = duration || Infinity;
      addMessageToState('admin', `${target} has been BANNED by ${state.currentUser} (${displayTime})`);
    } else if (type === 'mute') {
      state.muted[target] = duration || Infinity;
      addMessageToState('admin', `${target} has been MUTED by ${state.currentUser} (${displayTime})`);
    }
    
    saveState();
    showToast(`Action ${type} applied to ${target}`, 'success');
  }

  function removeMod(type, target) {
    if (!isMod(state.currentUser)) {
      showToast('Permission Denied.', 'error');
      return;
    }

    if (!target) return showToast(`Usage: /${type} <user>`, 'info');
    
    if (type === 'unmute' && state.muted[target]) {
      delete state.muted[target];
      addMessageToState('admin', `${target} has been UNMUTED`);
      showToast(`${target} unmuted`, 'success');
    } else if (type === 'unban' && state.banned[target]) {
      delete state.banned[target];
      addMessageToState('admin', `${target} has been UNBANNED`);
      showToast(`${target} unbanned`, 'success');
    } else {
      showToast(`User is not ${type === 'unmute' ? 'muted' : 'banned'}`, 'error');
    }
    saveState();
  }

  function setUserRole(user, role) {
    if (!isMod(state.currentUser)) {
      showToast('Permission Denied.', 'error');
      return;
    }

    if (!state.users[user]) return;
    state.users[user].role = role;
    saveState();
    addMessageToState('admin', `${user} is now ${ROLES[role].name}`);
    updateUIForRole();
    
    if (user === state.currentUser) {
        document.getElementById('myRoleTag').textContent = ROLES[role].name;
        document.getElementById('myRoleTag').style.color = ROLES[role].color;
    }
  }

  function isBanned(user) {
    if (!state.banned[user]) return false;
    if (state.banned[user] !== Infinity && Date.now() > state.banned[user]) {
      delete state.banned[user];
      saveState();
      return false;
    }
    return true;
  }

  function isMuted(user) {
    if (!state.muted[user]) return false;
    if (state.muted[user] !== Infinity && Date.now() > state.muted[user]) {
      delete state.muted[user];
      saveState();
      return false;
    }
    return true;
  }

  /**
   * TIMERS & REAL-TIME SYNC
   */
  function startMuteTimer() {
    if (muteInterval) clearInterval(muteInterval);
    muteInterval = setInterval(() => {
      if (!state.currentUser) return;
      if (state.muted[state.currentUser]) {
        const expiry = state.muted[state.currentUser];
        if (expiry !== Infinity && Date.now() > expiry) {
          delete state.muted[state.currentUser];
          saveState();
          showToast('You have been unmuted.', 'success');
        }
      }
    }, 1000);
  }

  function stopMuteTimer() {
    if (muteInterval) clearInterval(muteInterval);
  }

  window.addEventListener('storage', (e) => {
    if (e.key === DB_KEYS.USERS) {
      state.users = JSON.parse(e.newValue);
      if (state.currentUser) updateUIForRole();
    }
    if (e.key === DB_KEYS.MESSAGES) {
      const newMessages = JSON.parse(e.newValue);
      
      if (newMessages.length > state.messages.length) {
          const newOnes = newMessages.slice(state.messages.length);
          newOnes.forEach(msg => {
              checkNotification(msg);
          });
      }
      
      state.messages = newMessages;
      renderMessages();
    }
    if (e.key === DB_KEYS.MUTES) {
      state.muted = JSON.parse(e.newValue);
    }
    if (e.key === DB_KEYS.SUGGESTIONS) {
      state.suggestions = JSON.parse(e.newValue);
      if (!document.getElementById('suggestionsModal').classList.contains('hidden')) {
        renderSuggestionsList();
      }
    }
  });

  /**
   * DOM INTERACTIONS
   */
  
  const sidebar = document.getElementById('sidebarPanel');
  const overlay = document.getElementById('sidebarOverlay');
  function toggleSidebar(show) {
    if (show) {
      sidebar.classList.add('open');
      overlay.classList.add('open');
    } else {
      sidebar.classList.remove('open');
      overlay.classList.remove('open');
    }
  }
  document.getElementById('menuToggle').onclick = () => toggleSidebar(true);
  overlay.onclick = () => toggleSidebar(false);

  window.closeModals = () => {
    document.querySelectorAll('.modal-overlay').forEach(el => el.classList.remove('show'));
  };
  document.getElementById('menuRules').onclick = () => {
    toggleSidebar(false);
    document.getElementById('rulesModal').classList.add('show');
  };

  document.getElementById('menuSuggestions').onclick = () => {
    toggleSidebar(false);
    const modal = document.getElementById('suggestionsModal');
    const list = document.getElementById('suggestionsList');
    
    if (isMod(state.currentUser)) {
      list.classList.remove('hidden');
      renderSuggestionsList();
    } else {
      list.classList.add('hidden');
    }
    
    modal.classList.add('show');
  };

  document.getElementById('menuUsers').onclick = () => {
    renderUserList();
    toggleSidebar(false);
    document.getElementById('userListModal').classList.add('show');
  };

  function updateUIForRole() {
    const role = getUserRole(state.currentUser);
    const adminPanel = document.getElementById('adminPanel');
    const userList = document.getElementById('menuUsers');
    
    if (isMod(state.currentUser)) {
      adminPanel.classList.remove('hidden');
      userList.style.display = 'flex'; 
    } else {
      adminPanel.classList.add('hidden');
      userList.style.display = 'flex'; 
    }
  }

  document.getElementById('menuLogout').onclick = logout;

  document.getElementById('btnAssignRole').onclick = () => {
    const target = document.getElementById('roleTarget').value.trim();
    const role = document.getElementById('roleSelect').value;
    
    if (!target || !state.users[target]) return showToast('User not found', 'error');
    if (target === state.currentUser && role === 'member') return showToast('You cannot demote yourself', 'error');
    if (state.users[target].role === 'owner') return showToast('Cannot change Owner role', 'error');
    
    setUserRole(target, role);
    showToast(`${target} is now ${role}`, 'success');
  };
  
  document.getElementById('menuClearChat').onclick = () => {
     if(confirm("Are you sure you want to clear the chat for everyone?")) {
        state.messages = [];
        saveState();
        renderMessages();
        addMessageToState('admin', 'Chat cleared by ' + state.currentUser);
     }
  };

  document.getElementById('signupForm').onsubmit = (e) => {
    e.preventDefault();
    const u = document.getElementById('signupUsername').value.trim();
    const em = document.getElementById('signupEmail').value.trim().toLowerCase();
    const p = document.getElementById('signupPassword').value.trim();

    if (state.users[u]) return showToast('Username taken', 'error');
    
    // Check if email is already used
    const emailTaken = Object.values(state.users).some(user => user.email === em);
    if(emailTaken) return showToast('Email already in use.', 'error');

    const isFirstUser = Object.keys(state.users).length === 0;
    
    state.users[u] = { 
      password: p, 
      email: em,
      role: isFirstUser ? 'owner' : 'member',
      isOnline: true
    };
    
    state.currentUser = u;
    localStorage.setItem(DB_KEYS.SESSION, u);
    saveState();
    showToast('Account created!', 'success');
    enterChat();
  };

  document.getElementById('loginForm').onsubmit = (e) => {
    e.preventDefault();
    const u = document.getElementById('loginUsername').value.trim();
    const p = document.getElementById('loginPassword').value.trim();

    if (!state.users[u] || state.users[u].password !== p) {
      return showToast('Invalid credentials', 'error');
    }
    
    if (isBanned(u)) return showToast('You are banned.', 'error');

    state.currentUser = u;
    localStorage.setItem(DB_KEYS.SESSION, u);
    showToast('Welcome back!', 'success');
    enterChat();
  };

  initAuth();

</script>
</body>
</html>
