
<html>
<head>
<meta charset="UTF-8">
<title>Battlecraft Studios – Document Support</title>

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

button:hover {
  background: #ffbe33;
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

.doc:hover {
  background: rgba(255,255,255,0.25);
}

/* Editor */
#editor {
  flex: 1;
  padding: 20px;
  display: flex;
  flex-direction: column;
}

#doc-title {
  font-size: 22px;
  padding: 8px 10px;
  border-radius: 8px;
  border: 1px solid #ccc;
  margin-bottom: 10px;
}

#controls {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

#controls input {
  padding: 6px 8px;
  border-radius: 6px;
  border: 1px solid #ccc;
  flex: 1;
}

textarea {
  flex: 1;
  border-radius: 14px;
  padding: 15px;
  font-size: 16px;
  resize: none;
  border: none;
  outline: none;
  box-shadow: 0 10px 30px rgba(0,0,0,0.1);
}
</style>
</head>

<body>

<div id="sidebar">
  <div id="brand">
    Battlecraft Studios<br>
    <span>Document Support</span>
  </div>

  <button onclick="newDoc()">＋ New Document</button>
  <div id="docs"></div>
</div>

<div id="editor">
  <input id="doc-title" placeholder="Document Title" />
  <div id="controls">
    <input id="password" type="password" placeholder="Optional Password" />
  </div>
  <textarea id="content" placeholder="Write your document here..."></textarea>
</div>

<script>
let db;
let currentDoc = null;

const request = indexedDB.open("battlecraftDocsPro", 1);

request.onupgradeneeded = e => {
  db = e.target.result;
  db.createObjectStore("docs", { keyPath: "id", autoIncrement: true });
};

request.onsuccess = e => {
  db = e.target.result;
  loadDocs();
};

function loadDocs() {
  const tx = db.transaction("docs", "readonly");
  const store = tx.objectStore("docs");
  const req = store.getAll();

  req.onsuccess = () => {
    const list = document.getElementById("docs");
    list.innerHTML = "";
    req.result.forEach(doc => {
      const div = document.createElement("div");
      div.className = "doc" + (doc.password ? " locked" : "");
      div.textContent = doc.title || "Untitled Document";
      div.onclick = () => openDoc(doc);
      list.appendChild(div);
    });
  };
}

function newDoc() {
  const tx = db.transaction("docs", "readwrite");
  const store = tx.objectStore("docs");
  const doc = {
    title: "New Document",
    content: "",
    password: ""
  };

  const req = store.add(doc);
  req.onsuccess = e => {
    doc.id = e.target.result;
    openDoc(doc);
    loadDocs();
  };
}

function openDoc(doc) {
  if (doc.password) {
    const input = prompt("Enter password for this document:");
    if (input !== doc.password) {
      alert("Incorrect password.");
      return;
    }
  }

  currentDoc = doc;
  document.getElementById("doc-title").value = doc.title;
  document.getElementById("content").value = doc.content;
  document.getElementById("password").value = doc.password || "";
}

function saveDoc() {
  if (!currentDoc) return;

  currentDoc.title = document.getElementById("doc-title").value || "Untitled Document";
  currentDoc.content = document.getElementById("content").value;
  currentDoc.password = document.getElementById("password").value;

  const tx = db.transaction("docs", "readwrite");
  const store = tx.objectStore("docs");
  store.put(currentDoc);

  loadDocs();
}

document.getElementById("doc-title").addEventListener("input", saveDoc);
document.getElementById("content").addEventListener("input", saveDoc);
document.getElementById("password").addEventListener("input", saveDoc);
</script>

</body>
</html>
