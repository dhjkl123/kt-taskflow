# 업무 관리 앱 구현 계획

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 업무 추가·삭제·상태변경이 가능한 단일 HTML 파일 업무 관리 앱을 만든다.

**Architecture:** 모든 HTML 구조·CSS 스타일·JS 로직을 `index.html` 한 파일에 포함한다. 업무 목록은 JS 배열(메모리)로 관리하며, 변경 시마다 DOM을 전체 재렌더링한다. 필터 상태는 별도 변수로 관리한다.

**Tech Stack:** Vanilla HTML5, CSS3, JavaScript (ES6+), 빌드 도구 없음

---

## 파일 구조

```
D:\taskflow\
└── index.html   ← 단일 파일 (HTML + <style> + <script> 포함)
```

---

### Task 1: HTML 뼈대 + CSS 기본 레이아웃

**Files:**
- Create: `index.html`

- [ ] **Step 1: index.html 파일 생성 — HTML 뼈대 작성**

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>업무 관리 앱</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      background: #f5f5f5;
      min-height: 100vh;
      padding: 24px 16px;
    }

    .container {
      max-width: 600px;
      margin: 0 auto;
    }

    h1 {
      font-size: 1.6rem;
      font-weight: 700;
      color: #1a1a1a;
      margin-bottom: 20px;
      text-align: center;
    }

    /* 입력 영역 */
    .input-area {
      display: flex;
      gap: 8px;
      margin-bottom: 16px;
    }

    .input-area input {
      flex: 1;
      padding: 10px 14px;
      border: 1px solid #ddd;
      border-radius: 8px;
      font-size: 0.95rem;
      outline: none;
      transition: border-color 0.2s;
    }

    .input-area input:focus { border-color: #4a90e2; }

    .input-area button {
      padding: 10px 18px;
      background: #4a90e2;
      color: white;
      border: none;
      border-radius: 8px;
      font-size: 0.95rem;
      cursor: pointer;
      white-space: nowrap;
      transition: background 0.2s;
    }

    .input-area button:hover { background: #357abd; }

    /* 필터 탭 */
    .filter-tabs {
      display: flex;
      gap: 4px;
      margin-bottom: 16px;
      background: #fff;
      border-radius: 10px;
      padding: 4px;
      border: 1px solid #eee;
    }

    .filter-tabs button {
      flex: 1;
      padding: 8px 4px;
      border: none;
      background: transparent;
      border-radius: 7px;
      font-size: 0.85rem;
      cursor: pointer;
      color: #666;
      transition: all 0.2s;
    }

    .filter-tabs button.active {
      background: #4a90e2;
      color: white;
      font-weight: 600;
    }

    /* 업무 목록 */
    #task-list { display: flex; flex-direction: column; gap: 10px; }

    /* 업무 카드 */
    .task-card {
      background: white;
      border-radius: 10px;
      padding: 14px 16px;
      display: flex;
      align-items: center;
      gap: 12px;
      border: 1px solid #eee;
      box-shadow: 0 1px 3px rgba(0,0,0,0.05);
    }

    .task-title {
      flex: 1;
      font-size: 0.95rem;
      color: #1a1a1a;
    }

    .task-title.done {
      text-decoration: line-through;
      color: #aaa;
    }

    /* 상태 뱃지 */
    .status-badge {
      display: inline-block;
      padding: 2px 8px;
      border-radius: 12px;
      font-size: 0.75rem;
      font-weight: 600;
      white-space: nowrap;
    }

    .badge-todo       { background: #f0f0f0; color: #666; }
    .badge-inProgress { background: #e8f0fe; color: #4a90e2; }
    .badge-done       { background: #e6f4ea; color: #34a853; }

    /* 상태 선택 드롭다운 */
    .status-select {
      padding: 5px 8px;
      border: 1px solid #ddd;
      border-radius: 6px;
      font-size: 0.82rem;
      cursor: pointer;
      background: white;
      outline: none;
    }

    /* 삭제 버튼 */
    .delete-btn {
      padding: 5px 10px;
      background: transparent;
      border: 1px solid #ffcdd2;
      border-radius: 6px;
      color: #e57373;
      font-size: 0.82rem;
      cursor: pointer;
      transition: all 0.2s;
      white-space: nowrap;
    }

    .delete-btn:hover { background: #ffebee; }

    /* 빈 상태 */
    .empty-state {
      text-align: center;
      color: #bbb;
      font-size: 0.9rem;
      padding: 40px 0;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>업무 관리 앱</h1>

    <div class="input-area">
      <input type="text" id="task-input" placeholder="새 업무를 입력하세요..." />
      <button onclick="addTask()">추가</button>
    </div>

    <div class="filter-tabs">
      <button class="active" onclick="setFilter('all')"   id="tab-all">전체 (0)</button>
      <button onclick="setFilter('todo')"       id="tab-todo">할 일 (0)</button>
      <button onclick="setFilter('inProgress')" id="tab-inProgress">진행 중 (0)</button>
      <button onclick="setFilter('done')"       id="tab-done">완료 (0)</button>
    </div>

    <div id="task-list"></div>
  </div>

  <script>
    // 데이터
    let tasks = [];
    let currentFilter = 'all';

    // 업무 추가
    function addTask() {
      const input = document.getElementById('task-input');
      const title = input.value.trim();
      if (!title) return;
      tasks.push({ id: Date.now(), title, status: 'todo' });
      input.value = '';
      render();
    }

    // 업무 삭제
    function deleteTask(id) {
      tasks = tasks.filter(t => t.id !== id);
      render();
    }

    // 상태 변경
    function changeStatus(id, status) {
      const task = tasks.find(t => t.id === id);
      if (task) task.status = status;
      render();
    }

    // 필터 변경
    function setFilter(filter) {
      currentFilter = filter;
      document.querySelectorAll('.filter-tabs button').forEach(btn => btn.classList.remove('active'));
      document.getElementById('tab-' + filter).classList.add('active');
      render();
    }

    // 상태 한글 라벨
    const STATUS_LABEL = { todo: '할 일', inProgress: '진행 중', done: '완료' };
    const STATUS_BADGE  = { todo: 'badge-todo', inProgress: 'badge-inProgress', done: 'badge-done' };

    // 탭 카운트 갱신
    function updateTabCounts() {
      const counts = { todo: 0, inProgress: 0, done: 0 };
      tasks.forEach(t => counts[t.status]++);
      document.getElementById('tab-all').textContent        = `전체 (${tasks.length})`;
      document.getElementById('tab-todo').textContent       = `할 일 (${counts.todo})`;
      document.getElementById('tab-inProgress').textContent = `진행 중 (${counts.inProgress})`;
      document.getElementById('tab-done').textContent       = `완료 (${counts.done})`;
    }

    // 렌더링
    function render() {
      updateTabCounts();

      const filtered = currentFilter === 'all'
        ? tasks
        : tasks.filter(t => t.status === currentFilter);

      const list = document.getElementById('task-list');

      if (filtered.length === 0) {
        list.innerHTML = '<div class="empty-state">업무가 없습니다.</div>';
        return;
      }

      list.innerHTML = filtered.map(task => `
        <div class="task-card">
          <span class="task-title ${task.status === 'done' ? 'done' : ''}">${escapeHtml(task.title)}</span>
          <span class="status-badge ${STATUS_BADGE[task.status]}">${STATUS_LABEL[task.status]}</span>
          <select class="status-select" onchange="changeStatus(${task.id}, this.value)">
            <option value="todo"       ${task.status === 'todo'       ? 'selected' : ''}>할 일</option>
            <option value="inProgress" ${task.status === 'inProgress' ? 'selected' : ''}>진행 중</option>
            <option value="done"       ${task.status === 'done'       ? 'selected' : ''}>완료</option>
          </select>
          <button class="delete-btn" onclick="deleteTask(${task.id})">삭제</button>
        </div>
      `).join('');
    }

    // XSS 방지
    function escapeHtml(str) {
      return str.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
    }

    // Enter 키로 추가
    document.getElementById('task-input').addEventListener('keydown', e => {
      if (e.key === 'Enter') addTask();
    });

    // 초기 렌더링
    render();
  </script>
</body>
</html>
```

- [ ] **Step 2: 브라우저에서 열어 동작 확인**

`D:\taskflow\index.html`을 브라우저에서 열고 아래 항목을 확인한다:
- 업무 입력 후 "추가" 버튼 클릭 → 카드 생성 확인
- Enter 키로 업무 추가 확인
- 공백만 입력 후 추가 → 추가 안 됨 확인
- 드롭다운으로 상태 변경 → 뱃지 색상·텍스트 변경 확인
- 완료 상태로 변경 → 제목 취소선 확인
- 삭제 버튼 → 카드 제거 확인
- 필터 탭 클릭 → 해당 상태 업무만 표시 확인
- 탭 카운트 숫자가 실시간 갱신되는지 확인
- 업무 없을 때 "업무가 없습니다." 빈 상태 메시지 확인

- [ ] **Step 3: 커밋**

```bash
git add index.html
git commit -m "feat: add task management app (single HTML file)"
```

---

### Task 2: .gitignore 업데이트 및 최종 커밋

**Files:**
- Modify: `.gitignore`

- [ ] **Step 1: .gitignore에 .superpowers 추가 확인**

`.gitignore`에 아래 줄이 없으면 추가한다:

```
# 브레인스토밍 임시 파일
.superpowers/
```

- [ ] **Step 2: 커밋**

```bash
git add .gitignore
git commit -m "chore: ignore .superpowers directory"
```

- [ ] **Step 3: 원격 저장소에 푸쉬**

```bash
git push origin master
```
