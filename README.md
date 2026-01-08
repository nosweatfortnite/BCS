
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Battlecraft Studios – Document Support</title>

<!-- ========================= -->
<!--        STYLESHEET         -->
<!-- ========================= -->
<style>
* { box-sizing: border-box; }

body {
  margin: 0;
  height: 100vh;
  display: flex;
  font-family: "Segoe UI", system-ui, sans-serif;
  background: #eef1f7;
}

/* Sidebar */
#sidebar {
  width: 300px;
  padding: 16px;
  background: linear-gradient(180deg, #5f2cff, #3a0ca3);
  color: white;
  display: flex;
  flex-direction: column;
}

#brand {
  font-size: 18px;
  font-weight: 700;
  margin-bottom: 20px;
}

#brand span {
  font-size: 13px;
  opacity: 0.85;
}

button {
  background: #ffd166;
  border: none;
  border-radius: 8px;
  padding: 10px;
  font-weight: 600;
  cursor: pointer;
  margin-bottom: 12px;
}

#docs {
  flex: 1;
  overflow-y: auto;
}

.doc {
  background: rgba(255,255,255,0.15);
  border-radius: 8px;
  padding: 10px;
  margin-bottom: 8px;
  cursor: pointer;
  font-size: 14px;
}

.doc.locked::after {
  content: " 🔒";
}

/* Editor */
#editor {
  flex: 1;
  padding: 20px;
  display: flex;
  flex-direction: column;
}

input, textarea {
  font-size: 16px;
  padding: 10px;
  border-radius: 8px;
  border: 1px solid #ccc;
}

textarea {
  flex: 1;
  margin-top: 10px;
  resize: none;
}
</style>
</head>

<body>

<!-- ========================= -->
<!--           HTML            -->
<!-- ========================= -->
<div id="sidebar">
  <div id="brand">
    Battlecraft Studios<br>
    <span>Document Support</span>
  </div>

  <button onclick="newDoc()">＋ New Document</button>
  <div id="docs"></div>
</div>

<div id="editor">
  <input id="doc-title" placeholder="Document Title" disabled>
  <input id="doc-password" type="password" placeholder="Optional Password" disabled>
  <textarea id="content" placeholder="Write your document here..." disabled></textarea>
</div>

<!-- ========================= -->
<!--         JAVASCRIPT        -->
<!-- ========================= -->
<script>
/* ========= GLOBAL STATE ========= */
let db;
let currentDocId = null;

/* ========= DATABASE SETUP ========= */
const request = indexedDB.open("battlecraftDocsStable", 1);

request.onupgradeneeded = (e) => {
  db = e.target.result;
  db.createObjectStore("docs", {
    keyPath: "id",
    autoIncrement: true
  });
};

request.onsuccess = (e) => {
  db = e.target.result;
  loadDocs();
};

request.onerror = () => {
  alert("Failed to open database.");
};

/* ========= UI ELEMENTS ========= */
const titleInput = document.getElementById("doc-title");
const contentInput = document.getElementById("content");
const passwordInput = document.getElementById("doc-password");
const docsList = document.getElementById("docs");

function setEditorEnabled(enabled) {
  titleInput.disabled = !enabled;
  contentInput.disabled = !enabled;
  passwordInput.disabled = !enabled;
}

/* ========= LOAD DOCUMENT LIST ========= */
function loadDocs() {
  const tx = db.transaction("docs", "readonly");
  const store = tx.objectStore("docs");

  store.getAll().onsuccess = (e) => {
    docsList.innerHTML = "";

    e.target.result.forEach(doc => {
      const div = document.createElement("div");
      div.className = "doc" + (doc.password ? " locked" : "");
      div.textContent = doc.title || "Untitled";
      div.onclick = () => openDoc(doc.id);
      docsList.appendChild(div);
    });
  };
}

/* ========= CREATE DOCUMENT ========= */
function newDoc() {
  const tx = db.transaction("docs", "readwrite");
  const store = tx.objectStore("docs");

  store.add({
    title: "New Document",
    content: "",
    password: ""
  }).onsuccess = (e) => {
    openDoc(e.target.result);
  };
}

/* ========= OPEN DOCUMENT ========= */
function openDoc(id) {
  const tx = db.transaction("docs", "readonly");
  const store = tx.objectStore("docs");

  store.get(id).onsuccess = (e) => {
    const doc = e.target.result;
    if (!doc) return;

    if (doc.password) {
      const input = prompt("Enter password for this document:");
      if (input !== doc.password) {
        alert("Incorrect password.");
        return;
      }
    }

    currentDocId = id;
    titleInput.value = doc.title;
    contentInput.value = doc.content;
    passwordInput.value = doc.password || "";

    setEditorEnabled(true);
  };
}

/* ========= SAVE DOCUMENT ========= */
function saveDoc() {
  if (!currentDocId) return;

  const tx = db.transaction("docs", "readwrite");
  const store = tx.objectStore("docs");

  store.get(currentDocId).onsuccess = (e) => {
    const doc = e.target.result;
    if (!doc) return;

    doc.title = titleInput.value || "Untitled";
    doc.content = contentInput.value;
    doc.password = passwordInput.value || "";

    store.put(doc).onsuccess = loadDocs;
  };
}

/* ========= AUTO SAVE ========= */
titleInput.addEventListener("input", saveDoc);
contentInput.addEventListener("input", saveDoc);
passwordInput.addEventListener("input", saveDoc);

/* ========= INIT ========= */
setEditorEnabled(false);
</script>

</body>
</html>
