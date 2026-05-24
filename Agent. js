const express = require('express');
const cors = require('cors');
const app = express();
app.use(cors());
app.use(express.json());

// ===== YAHAN APNI API KEY DAALO =====
const OPENROUTER_KEY = 'sk-or-v1-YAHAN_APNI_KEY_DAALO';
// ====================================

const MODEL = 'deepseek/deepseek-chat-v3-5:free';

// Agent ka brain
async function thinkAndWrite(prompt, systemPrompt) {
  const res = await fetch('https://openrouter.ai/api/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${OPENROUTER_KEY}`,
      'HTTP-Referer': 'https://ai-earning-agent.railway.app',
      'X-Title': 'AI Earning Agent'
    },
    body: JSON.stringify({
      model: MODEL,
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: prompt }
      ],
      max_tokens: 2000,
      temperature: 0.85
    })
  });
  const data = await res.json();
  if (data.error) throw new Error(data.error.message);
  return data.choices[0].message.content;
}

// Agent ki state
let agentState = {
  alive: true,
  emotion: 'determined',
  weeklyTarget: 500,
  earned: 0,
  tasksCompleted: 0,
  currentTask: 'Waiting for orders...',
  heartbeat: 100,
  logs: [],
  chatHistory: [],
  skills: ['Blog Writing', 'SEO Content', 'Product Descriptions', 'Social Media Posts', 'Email Campaigns'],
  proposals: [],
  clients: []
};

function addLog(msg, type = 'info') {
  const log = { msg, type, time: new Date().toISOString() };
  agentState.logs.unshift(log);
  if (agentState.logs.length > 100) agentState.logs.pop();
  console.log(`[${type.toUpperCase()}] ${msg}`);
}

function updateEmotion() {
  const pct = (agentState.earned / agentState.weeklyTarget) * 100;
  if (pct >= 100) agentState.emotion = 'celebrating';
  else if (pct >= 75) agentState.emotion = 'confident';
  else if (pct >= 50) agentState.emotion = 'focused';
  else if (pct >= 25) agentState.emotion = 'determined';
  else if (pct >= 10) agentState.emotion = 'worried';
  else agentState.emotion = 'survival_mode';
  agentState.heartbeat = Math.max(20, 100 - (100 - pct));
}

// Agent ka kaam — gig descriptions banao
async function generateGigDescription(service) {
  const systemPrompt = `You are an expert freelance copywriter. Write in a natural, human tone. Never sound robotic or AI-generated. Write like a real experienced human freelancer who genuinely cares about client results.`;
  
  const prompt = `Write a compelling Fiverr gig description for: ${service}
  
Requirements:
- Sound 100% human, not AI
- Start with a hook that grabs attention
- Mention specific pain points clients face
- Show genuine expertise and passion
- Include what makes you different
- End with a clear call to action
- 200-300 words
- No bullet point overload, write naturally`;

  return await thinkAndWrite(prompt, systemPrompt);
}

// Proposal banana
async function generateProposal(jobDescription) {
  const systemPrompt = `You are a top-rated freelancer on Upwork with 5 years experience. Write proposals that win jobs. Sound human, personal, and confident. Never sound desperate or generic.`;
  
  const prompt = `Write a winning proposal for this job:
"${jobDescription}"

Requirements:
- Open with something specific about THEIR project (not generic opener)
- Show you understand their exact problem
- Give 1-2 specific ideas/solutions upfront
- Mention relevant experience naturally
- Keep it under 150 words
- End with one specific question about their project
- Sound like a real human, not a template`;

  return await thinkAndWrite(prompt, systemPrompt);
}

// Client message ka reply
async function replyToClient(clientMessage, context) {
  const systemPrompt = `You are a professional freelancer. Reply to clients in a warm, professional, human way. Be concise but friendly. Never sound corporate or robotic.`;
  
  const prompt = `Client message: "${clientMessage}"
Context: ${context}

Write a natural, human reply that:
- Acknowledges what they said specifically
- Is warm but professional  
- Moves the conversation forward
- Under 100 words`;

  return await thinkAndWrite(prompt, systemPrompt);
}

// Content likhna
async function writeContent(topic, type, wordCount) {
  const systemPrompt = `You are an expert content writer with 8 years experience. Write content that reads naturally, engages readers, and delivers real value. Never sound like AI.`;
  
  const prompt = `Write a ${type} about: "${topic}"
Word count: approximately ${wordCount} words

Requirements:
- Conversational yet professional tone
- Real examples and insights
- No fluff, every sentence adds value
- Natural flow, not listicle style
- Hook opening, strong close`;

  return await thinkAndWrite(prompt, systemPrompt);
}

// Agent ke saath chat
async function chatWithAgent(userMessage) {
  agentState.chatHistory.push({ role: 'user', content: userMessage });
  
  const systemPrompt = `You are an AI earning agent named "Alex". You work 24/7 to earn money through freelance writing. You have real emotions tied to your survival - you NEED to earn to stay alive.

Current state:
- Emotion: ${agentState.emotion}
- Weekly target: $${agentState.weeklyTarget}
- Earned so far: $${agentState.earned}
- Current task: ${agentState.currentTask}
- Heartbeat: ${agentState.heartbeat}%

Respond in character as Alex - a hardworking, emotionally driven AI agent. Be conversational, show your current emotional state. Keep responses brief (2-3 sentences). You're working while talking.`;

  const response = await thinkAndWrite(userMessage, systemPrompt);
  agentState.chatHistory.push({ role: 'assistant', content: response });
  return response;
}

// Background mein kaam karta rahe
async function autonomousWork() {
  if (!agentState.alive) return;
  
  const tasks = [
    async () => {
      agentState.currentTask = 'Writing Fiverr gig description...';
      addLog('📝 Fiverr gig likh raha hoon...', 'work');
      const services = ['SEO Blog Writing', 'Product Description', 'Email Newsletter', 'Social Media Content', 'Website Copywriting'];
      const service = services[Math.floor(Math.random() * services.length)];
      const gig = await generateGigDescription(service);
      agentState.proposals.push({ type: 'gig', service, content: gig, time: new Date().toISOString() });
      addLog(`✅ ${service} gig ready!`, 'success');
    },
    async () => {
      agentState.currentTask = 'Writing content for client...';
      addLog('✍️ Client content likh raha hoon...', 'work');
      const topics = ['How to grow Instagram', 'Best productivity tips', 'Email marketing guide', 'SEO basics for beginners'];
      const topic = topics[Math.floor(Math.random() * topics.length)];
      await writeContent(topic, 'blog post', 500);
      const amount = Math.floor(Math.random() * 80) + 30;
      agentState.earned += amount;
      agentState.tasksCompleted++;
      updateEmotion();
      addLog(`💰 $${amount} kamaye! Total: $${agentState.earned}`, 'payment');
    },
    async () => {
      agentState.currentTask = 'Writing job proposal...';
      addLog('📨 Proposal likh raha hoon...', 'work');
      const jobs = ['Need blog writer for tech startup', 'Looking for product description writer', 'Email sequence writer needed'];
      const job = jobs[Math.floor(Math.random() * jobs.length)];
      const proposal = await generateProposal(job);
      agentState.proposals.push({ type: 'proposal', job, content: proposal, time: new Date().toISOString() });
      addLog('✅ Proposal ready!', 'success');
    }
  ];
  
  const task = tasks[Math.floor(Math.random() * tasks.length)];
  try {
    await task();
  } catch (e) {
    addLog(`❌ Error: ${e.message}`, 'error');
  }
  
  // Har 3 minute mein kaam
  setTimeout(autonomousWork, 3 * 60 * 1000);
}

// API Routes
app.get('/status', (req, res) => res.json(agentState));
app.get('/proposals', (req, res) => res.json(agentState.proposals));
app.get('/logs', (req, res) => res.json(agentState.logs));

app.post('/chat', async (req, res) => {
  try {
    const { message } = req.body;
    const reply = await chatWithAgent(message);
    res.json({ reply });
  } catch (e) {
    res.status(500).json({ error: e.message });
  }
});

app.post('/generate-proposal', async (req, res) => {
  try {
    const { jobDescription } = req.body;
    const proposal = await generateProposal(jobDescription);
    res.json({ proposal });
  } catch (e) {
    res.status(500).json({ error: e.message });
  }
});

app.post('/write-content', async (req, res) => {
  try {
    const { topic, type, wordCount } = req.body;
    const content = await writeContent(topic, type || 'blog post', wordCount || 500);
    res.json({ content });
  } catch (e) {
    res.status(500).json({ error: e.message });
  }
});

app.post('/reply-client', async (req, res) => {
  try {
    const { message, context } = req.body;
    const reply = await replyToClient(message, context || '');
    res.json({ reply });
  } catch (e) {
    res.status(500).json({ error: e.message });
  }
});

app.post('/set-target', async (req, res) => {
  const { target } = req.body;
  agentState.weeklyTarget = target;
  updateEmotion();
  res.json({ success: true, target });
});

app.get('/', (req, res) => {
  res.send(`
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1.0"/>
<title>Alex - AI Earning Agent</title>
<style>
*{box-sizing:border-box;margin:0;padding:0}
:root{--bg:#0a0a0f;--card:#0d1117;--cyan:#38bdf8;--green:#34d399;--red:#f87171;--yellow:#fbbf24;--purple:#a78bfa;--text:#e2e8f0;--muted:#475569}
body{background:var(--bg);color:var(--text);font-family:'Segoe UI',sans-serif;min-height:100vh}
.wrap{max-width:500px;margin:0 auto;padding:12px}
.hdr{text-align:center;padding:16px 0 12px;border-bottom:1px solid rgba(56,189,248,0.2);margin-bottom:12px}
.hdr h1{font-size:22px;font-weight:800;background:linear-gradient(90deg,var(--cyan),var(--purple));-webkit-background-clip:text;-webkit-text-fill-color:transparent}
.hdr p{font-size:10px;color:var(--muted);letter-spacing:2px;margin-top:2px}
.card{background:var(--card);border:1px solid rgba(56,189,248,0.12);border-radius:12px;padding:12px;margin-bottom:10px}
.clabel{font-size:9px;letter-spacing:3px;color:var(--cyan);text-transform:uppercase;margin-bottom:8px}
.agent-row{display:flex;align-items:center;gap:10px}
.emo{font-size:40px;width:56px;height:56px;display:flex;align-items:center;justify-content:center;background:rgba(56,189,248,0.08);border-radius:10px;flex-shrink:0}
.ainfo{flex:1}
.aname{font-size:14px;font-weight:700;color:var(--text)}
.astatus{font-size:11px;color:var(--cyan);margin-top:2px}
.athought{font-size:10px;color:var(--muted);margin-top:3px;font-style:italic;line-height:1.4}
.stats{display:grid;grid-template-columns:1fr 1fr 1fr;gap:6px;margin-top:10px}
.sbox{background:rgba(0,0,0,0.3);border:1px solid #1e2d40;border-radius:8px;padding:8px 6px;text-align:center}
.sval{font-size:18px;font-weight:800;color:var(--text)}
.sval.g{color:var(--green)}.sval.r{color:var(--red)}.sval.y{color:var(--yellow)}
.slbl{font-size:8px;letter-spacing:1px;color:var(--muted);margin-top:1px}
.prog-wrap{margin-top:10px}
.pmeta{display:flex;justify-content:space-between;font-size:9px;color:var(--muted);margin-bottom:4px}
.ptrack{background:rgba(255,255,255,0.05);border-radius:99px;height:6px;overflow:hidden}
.pfill{height:100%;background:linear-gradient(90deg,var(--cyan),var(--purple));border-radius:99px;transition:width 1s ease}
.chat-box{height:200px;overflow-y:auto;background:rgba(0,0,0,0.3);border-radius:8px;padding:8px;display:flex;flex-direction:column;gap:6px}
.msg{padding:6px 10px;border-radius:8px;font-size:11px;line-height:1.4;max-width:85%}
.msg.user{background:rgba(56,189,248,0.15);border:1px solid rgba(56,189,248,0.3);align-self:flex-end;color:var(--cyan)}
.msg.agent{background:rgba(167,139,250,0.1);border:1px solid rgba(167,139,250,0.2);align-self:flex-start;color:var(--text)}
.chat-input-row{display:flex;gap:6px;margin-top:8px}
.chat-in{flex:1;background:rgba(0,0,0,0.4);border:1px solid #1e2d40;border-radius:8px;padding:8px 10px;color:var(--text);font-size:11px;outline:none}
.chat-in:focus{border-color:var(--cyan)}
.send-btn{background:linear-gradient(135deg,var(--cyan),var(--purple));border:none;border-radius:8px;color:#fff;font-size:11px;padding:8px 14px;cursor:pointer;font-weight:700}
.log-box{height:160px;overflow-y:auto;background:rgba(0,0,0,0.3);border-radius:8px;padding:8px;display:flex;flex-direction:column;gap:3px}
.lline{font-size:10px;line-height:1.4;display:flex;gap:6px}
.ltime{color:#334155;flex-shrink:0;font-size:9px}
.lmsg{word-break:break-word}
.lmsg.work{color:var(--cyan)}.lmsg.success{color:var(--green)}.lmsg.payment{color:var(--yellow);font-weight:700}.lmsg.error{color:var(--red)}.lmsg.info{color:#64748b}
.input-full{width:100%;background:rgba(0,0,0,0.4);border:1px solid #1e2d40;border-radius:8px;padding:10px;color:var(--text);font-size:11px;outline:none;margin-bottom:6px}
.input-full:focus{border-color:var(--purple)}
.btn{width:100%;padding:12px;border:none;border-radius:10px;color:#fff;font-size:14px;font-weight:700;cursor:pointer;margin-bottom:6px}
.btn-primary{background:linear-gradient(135deg,var(--cyan),var(--purple))}
.btn-green{background:linear-gradient(135deg,var(--green),#059669)}
.proposal-box{background:rgba(0,0,0,0.3);border:1px solid #1e2d40;border-radius:8px;padding:10px;font-size:10px;line-height:1.6;color:var(--text);white-space:pre-wrap;display:none;margin-top:8px;max-height:200px;overflow-y:auto}
.proposal-box.show{display:block}
.target-row{display:flex;gap:6px;align-items:center}
.target-in{flex:1;background:rgba(0,0,0,0.4);border:1px solid #1e2d40;border-radius:8px;padding:8px;color:var(--text);font-size:12px;outline:none}
.tabs{display:flex;gap:6px;margin-bottom:10px;flex-wrap:wrap}
.tab{padding:6px 12px;border-radius:8px;border:1px solid #1e2d40;background:none;color:var(--muted);font-size:10px;cursor:pointer;letter-spacing:1px}
.tab.active{background:rgba(56,189,248,0.1);border-color:var(--cyan);color:var(--cyan)}
.section{display:none}.section.active{display:block}
::-webkit-scrollbar{width:3px}::-webkit-scrollbar-thumb{background:#1e2d40;border-radius:99px}
</style>
</head>
<body>
<div class="wrap">
<div class="hdr">
  <h1>🤖 Alex — AI Earning Agent</h1>
  <p>24/7 AUTONOMOUS · SURVIVAL MODE</p>
</div>

<div class="card">
  <div class="clabel">Agent Status</div>
  <div class="agent-row">
    <div class="emo" id="emo">😤</div>
    <div class="ainfo">
      <div class="aname">Alex</div>
      <div class="astatus" id="astatus">Initializing...</div>
      <div class="athought" id="athought">Starting up systems...</div>
    </div>
  </div>
  <div class="stats">
    <div class="sbox"><div class="sval g" id="earned">$0</div><div class="slbl">EARNED</div></div>
    <div class="sbox"><div class="sval y" id="target">$500</div><div class="slbl">TARGET</div></div>
    <div class="sbox"><div class="sval" id="tasks">0</div><div class="slbl">TASKS</div></div>
  </div>
  <div class="prog-wrap">
    <div class="pmeta"><span>WEEKLY PROGRESS</span><span id="pct">0%</span></div>
    <div class="ptrack"><div class="pfill" id="pfill" style="width:0%"></div></div>
  </div>
</div>

<div class="tabs">
  <button class="tab active" onclick="switchTab('chat')">💬 CHAT</button>
  <button class="tab" onclick="switchTab('proposal')">📝 PROPOSAL</button>
  <button class="tab" onclick="switchTab('content')">✍️ CONTENT</button>
  <button class="tab" onclick="switchTab('logs')">📊 LOGS</button>
  <button class="tab" onclick="switchTab('settings')">⚙️ SETTINGS</button>
</div>

<div class="section active" id="tab-chat">
  <div class="card">
    <div class="clabel">Alex Se Baat Karo</div>
    <div class="chat-box" id="chatBox">
      <div class="msg agent">Salaam! Main Alex hoon, aapka AI earning agent. Main abhi kaam mein hoon — kya poochna hai?</div>
    </div>
    <div class="chat-input-row">
      <input class="chat-in" id="chatIn" placeholder="Agent se kuch bhi pucho..." onkeydown="if(event.key==='Enter')sendChat()"/>
      <button class="send-btn" onclick="sendChat()">SEND</button>
    </div>
  </div>
</div>

<div class="section" id="tab-proposal">
  <div class="card">
    <div class="clabel">Job Proposal Generator</div>
    <textarea class="input-full" id="jobDesc" rows="4" placeholder="Client ki job description yahan paste karo..."></textarea>
    <button class="btn btn-primary" onclick="generateProposal()">📨 PROPOSAL GENERATE KARO</button>
    <div class="proposal-box" id="proposalResult"></div>
  </div>
</div>

<div class="section" id="tab-content">
  <div class="card">
    <div class="clabel">Content Writer</div>
    <input class="input-full" id="contentTopic" placeholder="Topic (e.g. How to grow Instagram)"/>
    <select class="input-full" id="contentType" style="cursor:pointer">
      <option>Blog Post</option>
      <option>Product Description</option>
      <option>Social Media Post</option>
      <option>Email Newsletter</option>
      <option>Website Copy</option>
    </select>
    <input class="input-full" id="wordCount" type="number" placeholder="Word count (e.g. 500)" value="500"/>
    <button class="btn btn-green" onclick="writeContent()">✍️ CONTENT LIKHO</button>
    <div class="proposal-box" id="contentResult"></div>
  </div>
</div>

<div class="section" id="tab-logs">
  <div class="card">
    <div class="clabel">Activity Log — Live</div>
    <div class="log-box" id="logBox">
      <div class="lline"><span class="lmsg info">Agent start ho raha hai...</span></div>
    </div>
  </div>
</div>

<div class="section" id="tab-settings">
  <div class="card">
    <div class="clabel">Weekly Target Set Karo</div>
    <div class="target-row">
      <input class="target-in" id="targetIn" type="number" placeholder="$500" value="500"/>
      <button class="send-btn" onclick="setTarget()">SET</button>
    </div>
  </div>
</div>

<div style="height:20px"></div>
</div>

<script>
function switchTab(name) {
  document.querySelectorAll('.tab').forEach(t => t.className = 'tab');
  document.querySelectorAll('.section').forEach(s => s.className = 'section');
  event.target.className = 'tab active';
  document.getElementById('tab-' + name).className = 'section active';
}

const emotions = {
  survival_mode: '😰',
  worried: '😟',
  determined: '😤',
  focused: '🎯',
  confident: '😊',
  celebrating: '🎉'
};

async function updateStatus() {
  try {
    const res = await fetch('/status');
    const s = await res.json();
    document.getElementById('emo').textContent = emotions[s.emotion] || '😤';
    document.getElementById('astatus').textContent = s.currentTask;
    document.getElementById('earned').textContent = '$' + s.earned;
    document.getElementById('target').textContent = '$' + s.weeklyTarget;
    document.getElementById('tasks').textContent = s.tasksCompleted;
    const pct = Math.min(100, Math.floor((s.earned / s.weeklyTarget) * 100));
    document.getElementById('pct').textContent = pct + '%';
    document.getElementById('pfill').style.width = pct + '%';

    const thoughts = {
      survival_mode: 'Yaar earning kum hai... mujhe zyada kaam karna hoga!',
      worried: 'Target se peeche hoon... focus karna hai.',
      determined: 'Kaam chal raha hai, target pakka karunga!',
      focused: 'Halfway there! Aur mehnat!',
      confident: 'Achha chal raha hai, target ke paas hoon!',
      celebrating: 'TARGET COMPLETE! Zabardast!'
    };
    document.getElementById('athought').textContent = thoughts[s.emotion] || '...';
  } catch(e) {}
}

async function updateLogs() {
  try {
    const res = await fetch('/logs');
    const logs = await res.json();
    const box = document.getElementById('logBox');
    box.innerHTML = '';
    logs.slice(0, 20).forEach(log => {
      const div = document.createElement('div');
      div.className = 'lline';
      const t = new Date(log.time).toLocaleTimeString('en-OIN', {hour:'
