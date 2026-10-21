/\* sFlowForge Pro \- Multi-Gigabit Sampling Dark UI \*/  
:root {  
  \--bg-primary: \#020617;  
  \--bg-card: \#0b152d;  
  \--bg-card-alt: \#070e20;  
  \--border-subtle: \#192749;  
  \--border-focus: \#38bdf8;  
  \--text-main: \#f8fafc;  
  \--text-muted: \#94a3b8;  
  \--text-dim: \#64748b;  
  \--accent-cyan: \#06b6d4;  
  \--accent-blue: \#3b82f6;  
  \--accent-emerald: \#10b981;  
  \--accent-amber: \#f59e0b;  
  \--accent-rose: \#f43f5e;  
  \--font-sans: \-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;  
  \--font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;  
}

\* {  
  box-sizing: border-box;  
  margin: 0;  
  padding: 0;  
}

body {  
  background-color: var(--bg-primary);  
  color: var(--text-main);  
  font-family: var(--font-sans);  
  line-height: 1.5;  
  padding: 20px;  
}

.app-container {  
  max-width: 1440px;  
  margin: 0 auto;  
}

/\* Header \*/  
.app-header {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  background: var(--bg-card);  
  border: 1px solid var(--border-subtle);  
  border-radius: 12px;  
  padding: 16px 24px;  
  margin-bottom: 16px;  
}

.brand {  
  display: flex;  
  align-items: center;  
  gap: 16px;  
}

.brand-badge {  
  background: linear-gradient(135deg, var(--accent-cyan), var(--accent-blue));  
  color: \#020617;  
  font-weight: 800;  
  font-size: 13px;  
  padding: 6px 12px;  
  border-radius: 8px;  
  letter-spacing: 0.05em;  
}

.brand-title h1 {  
  font-size: 20px;  
  font-weight: 800;  
  color: var(--text-main);  
}

.version-tag {  
  font-size: 13px;  
  font-weight: 600;  
  color: var(--accent-cyan);  
}

.subtitle {  
  font-size: 13px;  
  color: var(--text-muted);  
}

.header-badges {  
  display: flex;  
  gap: 10px;  
}

.badge {  
  font-size: 11px;  
  font-weight: 700;  
  padding: 4px 10px;  
  border-radius: 20px;  
  text-transform: uppercase;  
  letter-spacing: 0.05em;  
}

.badge-cyan {  
  background: rgba(6, 182, 212, 0.15);  
  color: \#22d3ee;  
  border: 1px solid rgba(6, 182, 212, 0.3);  
}

.badge-emerald {  
  background: rgba(16, 185, 129, 0.15);  
  color: \#34d399;  
  border: 1px solid rgba(16, 185, 129, 0.3);  
}

/\* Preset Bar \*/  
.preset-bar {  
  display: flex;  
  align-items: center;  
  gap: 12px;  
  background: var(--bg-card);  
  border: 1px solid var(--border-subtle);  
  border-radius: 10px;  
  padding: 12px 20px;  
  margin-bottom: 20px;  
}

.preset-label {  
  font-size: 13px;  
  font-weight: 600;  
  color: var(--text-muted);  
}

.studio-grid {  
  display: grid;  
  grid-template-columns: 1fr 1.15fr;  
  gap: 20px;  
}

@media (max-width: 1024px) {  
  .studio-grid {  
    grid-template-columns: 1fr;  
  }  
}

/\* Cards \*/  
.card {  
  background: var(--bg-card);  
  border: 1px solid var(--border-subtle);  
  border-radius: 12px;  
  margin-bottom: 20px;  
  overflow: hidden;  
}

.card-header {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  padding: 14px 20px;  
  background: rgba(255, 255, 255, 0.02);  
  border-bottom: 1px solid var(--border-subtle);  
}

.card-header h2 {  
  font-size: 15px;  
  font-weight: 700;  
  color: var(--text-main);  
}

.header-stat {  
  font-size: 12px;  
  color: var(--text-muted);  
}

.card-body {  
  padding: 18px 20px;  
}

/\* Form inputs \*/  
.form-row {  
  display: flex;  
  gap: 14px;  
  margin-bottom: 12px;  
}

.form-group {  
  margin-bottom: 12px;  
}

.flex-1 {  
  flex: 1;  
}

label {  
  display: block;  
  font-size: 12px;  
  font-weight: 600;  
  color: var(--text-muted);  
  margin-bottom: 6px;  
}

.select-input, .text-input {  
  width: 100%;  
  background: var(--bg-primary);  
  border: 1px solid var(--border-subtle);  
  border-radius: 6px;  
  padding: 8px 12px;  
  color: var(--text-main);  
  font-size: 13px;  
  transition: border-color 0.15s ease;  
}

.select-input:focus, .text-input:focus {  
  outline: none;  
  border-color: var(--border-focus);  
}

.font-mono {  
  font-family: var(--font-mono);  
}

.highlight-mono {  
  font-family: var(--font-mono);  
  font-weight: 700;  
  color: \#38bdf8;  
}

.help-text {  
  display: block;  
  font-size: 11px;  
  color: var(--text-dim);  
  margin-top: 4px;  
}

.calc-formula-box {  
  background: var(--bg-card-alt);  
  border: 1px solid var(--border-subtle);  
  border-radius: 8px;  
  padding: 10px 14px;  
  margin-bottom: 14px;  
  font-size: 12px;  
}

/\* Toggle Grid \*/  
.toggle-grid {  
  display: grid;  
  grid-template-columns: 1fr 1fr;  
  gap: 12px;  
  margin-top: 12px;  
}

.toggle-item {  
  display: flex;  
  align-items: flex-start;  
  gap: 10px;  
  background: var(--bg-card-alt);  
  border: 1px solid var(--border-subtle);  
  border-radius: 8px;  
  padding: 10px 12px;  
  cursor: pointer;  
}

.toggle-label strong {  
  display: block;  
  font-size: 12px;  
  color: var(--text-main);  
}

.toggle-sub {  
  font-size: 10px;  
  color: var(--text-dim);  
  display: block;  
}

/\* Scorecard \*/  
.scorecard-header {  
  display: flex;  
  align-items: center;  
  gap: 20px;  
  padding: 18px 20px;  
  background: rgba(255, 255, 255, 0.02);  
  border-bottom: 1px solid var(--border-subtle);  
}

.grade-badge {  
  width: 64px;  
  height: 64px;  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  border-radius: 12px;  
  font-size: 28px;  
  font-weight: 800;  
  font-family: var(--font-mono);  
}

.grade-a {  
  background: rgba(16, 185, 129, 0.2);  
  color: \#34d399;  
  border: 2px solid \#10b981;  
}

.grade-b {  
  background: rgba(59, 130, 246, 0.2);  
  color: \#60a5fa;  
  border: 2px solid \#3b82f6;  
}

.grade-f {  
  background: rgba(244, 63, 94, 0.2);  
  color: \#fb7185;  
  border: 2px solid \#f43f5e;  
}

.scorecard-summary {  
  flex: 1;  
}

.scorecard-summary h3 {  
  font-size: 15px;  
  font-weight: 700;  
  margin-bottom: 8px;  
}

.score-progress-bar {  
  height: 8px;  
  background: rgba(255, 255, 255, 0.08);  
  border-radius: 4px;  
  overflow: hidden;  
  margin-bottom: 8px;  
}

.score-fill {  
  height: 100%;  
  background: linear-gradient(90deg, var(--accent-cyan), var(--accent-emerald));  
  transition: width 0.3s ease;  
}

.score-metrics {  
  display: flex;  
  justify-content: space-between;  
  font-size: 12px;  
  color: var(--text-muted);  
}

.rubric-list {  
  display: flex;  
  flex-direction: column;  
  gap: 8px;  
}

.rubric-item {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  background: var(--bg-card-alt);  
  border: 1px solid var(--border-subtle);  
  border-radius: 6px;  
  padding: 8px 12px;  
  font-size: 12px;  
}

.rubric-status {  
  font-weight: 700;  
  margin-right: 8px;  
}

.status-pass { color: var(--accent-emerald); }  
.status-warn { color: var(--accent-amber); }  
.status-fail { color: var(--accent-rose); }

/\* Tabs & Code preview \*/  
.tab-group {  
  display: flex;  
  gap: 6px;  
  flex-wrap: wrap;  
}

.tab-btn {  
  background: transparent;  
  border: 1px solid var(--border-subtle);  
  color: var(--text-muted);  
  border-radius: 6px;  
  padding: 4px 10px;  
  font-size: 11px;  
  font-weight: 600;  
  cursor: pointer;  
  transition: all 0.15s ease;  
}

.tab-btn.active {  
  background: rgba(6, 182, 212, 0.15);  
  color: \#38bdf8;  
  border-color: var(--accent-cyan);  
}

.code-container {  
  position: relative;  
  background: \#020409;  
  border: 1px solid var(--border-subtle);  
  border-radius: 8px;  
  padding: 14px;  
}

.code-container pre {  
  margin: 0;  
  overflow-x: auto;  
  max-height: 280px;  
}

.code-container code {  
  font-size: 12px;  
  line-height: 1.5;  
  color: \#38bdf8;  
}

.copy-btn {  
  position: absolute;  
  top: 10px;  
  right: 10px;  
}

.code-actions {  
  display: flex;  
  justify-content: flex-end;  
  gap: 10px;  
  margin-top: 12px;  
}

/\* Buttons \*/  
.btn {  
  padding: 8px 16px;  
  border-radius: 6px;  
  font-size: 13px;  
  font-weight: 600;  
  cursor: pointer;  
  transition: all 0.15s ease;  
  border: none;  
}

.btn-sm {  
  padding: 5px 10px;  
  font-size: 11px;  
}

.btn-primary {  
  background: linear-gradient(135deg, var(--accent-cyan), var(--accent-blue));  
  color: \#020617;  
}

.btn-primary:hover {  
  filter: brightness(1.1);  
}

.btn-secondary {  
  background: var(--bg-card-alt);  
  border: 1px solid var(--border-subtle);  
  color: var(--text-main);  
}

.btn-secondary:hover {  
  background: rgba(255, 255, 255, 0.05);  
}

/\* Utilities \*/  
.text-emerald { color: var(--accent-emerald); }  
.text-amber { color: var(--accent-amber); }  
.text-rose { color: var(--accent-rose); }

/\* Footer \*/  
.app-footer {  
  text-align: center;  
  padding: 24px 0 10px;  
  font-size: 12px;  
  color: var(--text-dim);  
  border-top: 1px solid var(--border-subtle);  
  margin-top: 20px;  
}  
