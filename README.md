# 📂 Tab Save Load

![Иконка](icon.png)

Расширение Chrome, которое сохраняет все открытые вкладки текущего окна в текстовый файл (названия и URL) и восстанавливает их обратно в один клик. Файл сохраняется в папку «Загрузки».

---

## 🎥 Видеообзор

[![Tab Save Load — видеообзор](https://img.youtube.com/vi/Xy-TOU82UvM/maxresdefault.jpg)](https://youtu.be/Xy-TOU82UvM)

▶️ [Смотреть на YouTube](https://youtu.be/Xy-TOU82UvM)

---

## 🚀 Что делает приложение?
1. **Сохранение:** Экспортирует названия (заголовки) и URL-адреса всех открытых вкладок текущего окна в текстовый `.txt` файл. Файл автоматически сохраняется в стандартную **папку «Загрузки»** вашего компьютера под именем вида `tabs_2026-06-06_14-30.txt` (дата и время создания).
2. **Загрузка:** Открывает окно выбора файла (по умолчанию — файл в папке «Загрузки», но можно выбрать любое другое место, если вы переместили файл). Считывает ссылки и автоматически открывает их в новых вкладках.
---

## 📁 Исходный код файлов проекта

### 1. `manifest.json`
Главный файл конфигурации расширения (Manifest V3). Описывает название, версию, иконку, необходимые разрешения (`tabs`) и указывает графический интерфейс (`popup.html`).

```json
{
  "manifest_version": 3,
  "name": "Сохранение и Загрузка вкладок",
  "version": "1.0",
  "description": "Сохраняет вкладки в текстовый файл и восстанавливает их обратно",
  "permissions": ["tabs"],
  "action": {
    "default_popup": "popup.html"
  },
  "icons": {
    "128": "icon.png"
  }
}
```
### 2. popup.html
Файл визуального интерфейса (всплывающее окно расширения). Содержит стили оформления, две кнопки («Сохранить в файл» и «Загрузить из файла»), скрытый элемент выбора файла и блок статуса.

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <style>
    body {
      width: 220px;
      font-family: Arial, sans-serif;
      padding: 12px;
      text-align: center;
      background-color: #f7f9fa;
      margin: 0;
    }
    h3 {
      margin: 0 0 12px 0;
      font-size: 15px;
      color: #333;
    }
    .btn-container {
      display: flex;
      flex-direction: column;
      gap: 8px;
    }
    button {
      width: 100%;
      padding: 10px;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-weight: bold;
      color: white;
      font-size: 13px;
      transition: background 0.2s;
    }
    #saveBtn {
      background-color: #34a853;
    }
    #saveBtn:hover {
      background-color: #2d8d47;
    }
    #loadBtn {
      background-color: #4285f4;
    }
    #loadBtn:hover {
      background-color: #3367d6;
    }
    #status {
      margin-top: 10px;
      font-size: 12px;
      font-weight: bold;
      color: #333;
      white-space: pre-line;
    }
  </style>
</head>
<body>
  <h3>Менеджер вкладок</h3>
  <div class="btn-container">
    <button id="saveBtn">💾 Сохранить в файл</button>
    <button id="loadBtn">📂 Загрузить из файла</button>
  </div>
  <!-- Скрытый инпут для выбора файла при загрузке -->
  <input type="file" id="fileInput" accept=".txt" style="display: none;">
  <div id="status"></div>
  <script src="popup.js"></script>
</body>
</html>
```
### 3. popup.js
Скрипт логики расширения. Отвечает за сбор ссылок через Chrome API, генерацию файла (автоматически присваивает имя с текущей датой и временем: tabs_ГГГГ-ММ-ДД_ЧЧ-ММ.txt и сохраняет его в системную папку «Загрузки»), а также за парсинг .txt файла и открытие вкладок.

```JavaScript

// 1. СОХРАНЕНИЕ ВКЛАДОК В ФАЙЛ
document.getElementById('saveBtn').addEventListener('click', async () => {
  const statusDiv = document.getElementById('status');
  try {
    const tabs = await chrome.tabs.query({ currentWindow: true });
    
    let content = "Менеджер вкладок — Список сохраненных ссылок\n";
    content += "========================================\n\n";
    
    tabs.forEach(tab => {
      content += `Заголовок: ${tab.title}\n`;
      content += `URL: ${tab.url}\n`;
      content += "----------------------------------------\n";
    });
    
    const blob = new Blob([content], { type: 'text/plain;charset=utf-8' });
    const url = URL.createObjectURL(blob);
    
    const now = new Date();
    const dateStr = now.toISOString().slice(0, 10);
    const timeStr = now.toTimeString().slice(0, 5).replace(':', '-');
    const filename = `tabs_${dateStr}_${timeStr}.txt`;
    
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
    
    statusDiv.style.color = '#2e7d32';
    statusDiv.innerText = `✅ Сохранено вкладок: ${tabs.length}`;
  } catch (e) {
    statusDiv.style.color = '#c62828';
    statusDiv.innerText = `❌ Ошибка сохранения`;
    console.log(e);
  }
});

// 2. ЗАГРУЗКА ВКЛАДОК ИЗ ФАЙЛА
document.getElementById('loadBtn').addEventListener('click', () => {
  document.getElementById('fileInput').click();
});

document.getElementById('fileInput').addEventListener('change', async (event) => {
  const file = event.target.files[0];
  const statusDiv = document.getElementById('status');
  
  if (!file) return;

  try {
    const text = await file.text();
    const urlMatches = text.match(/^URL:\s*(.+)$/gm);
    
    if (!urlMatches || urlMatches.length === 0) {
      statusDiv.style.color = '#c62828';
      statusDiv.innerText = `❌ Файл не содержит ссылок`;
      return;
    }

    let openedCount = 0;
    
    for (let match of urlMatches) {
      const url = match.replace(/^URL:\s*/, '').trim();
      if (url.startsWith('http://') || url.startsWith('https://')) {
        await chrome.tabs.create({ url: url, active: false });
        openedCount++;
      }
    }
    
    statusDiv.style.color = '#2e7d32';
    statusDiv.innerText = `✅ Открыто вкладок: ${openedCount}`;
  } catch (e) {
    statusDiv.style.color = '#c62828';
    statusDiv.innerText = `❌ Ошибка чтения файла`;
    console.log(e);
  }
});

```

### 4. icon.png
Графическая иконка расширения (размером 128x128 пикселей), отображаемая на панели браузера.

---

## ⚙️ Как установить и запустить локально

Вы можете установить и протестировать расширение в браузере Google Chrome за пару минут:

1. **Скачайте проект** с GitHub (кнопка *Code -> Download ZIP*) или склонируйте репозиторий, либо просто подготовьте папку `tab-save-load` на компьютере со всеми четырьмя файлами (`manifest.json`, `popup.html`, `popup.js`, `icon.png`).
2. Откройте браузер Google Chrome и перейдите по служебному адресу:
   ```text
   chrome://extensions/
3. В правом верхнем углу страницы включите тумблер «Режим разработчика» (Developer mode).

4. В левом верхнем углу нажмите кнопку «Загрузить распакованное расширение» (Load unpacked).

5. В открывшемся окне проводника выберите вашу папку с проектом tab-save-load.

Готово! Иконка расширения появится на панели браузера. Нажмите на неё, чтобы использовать функции сохранения или загрузки вкладок.

---

## 👤 Автор

**AS7go** — разработчик расширения Tab RAM Cleaner.

- 🐙 GitHub: [@AS7go](https://github.com/AS7go)
- 📺 YouTube: [Мой канал](https://www.youtube.com/channel/UCQ3sX5SfuoL8F2wagAt_UCg)

Если расширение оказалось полезным — поставьте звёздочку на GitHub ⭐
