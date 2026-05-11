# 六下英語 Unit 3-4 聽力測驗

這是一份單頁式線上聽力測驗，範圍對應六下翰林 Here We Go 8 的 Unit 3 與 Unit 4 主題與句型。題目為原創練習題，可直接用瀏覽器開啟 `index.html` 測試。

## 使用方式

1. 將 `index.html` 部署到 GitHub Pages、Netlify、Google Sites 可嵌入頁面，或其他靜態網頁空間。
2. 開啟部署後的網址。
3. 在右側「測驗網址」貼上正式網址，按「產生 QR」。
4. 學生掃 QR code 後填寫年級、班級、座號，格式會記錄為 `__年__班__號`。
5. 學生送出後會立即看到 100 分制分數。

## 作答歷程

目前頁面會把作答歷程保存在作答裝置的瀏覽器 `localStorage`，並可下載 CSV。

## 存到 Firebase Firestore

可以把學生作答紀錄集中存到 Firebase 的 Cloud Firestore。設定完成後，每位學生送出時會新增一筆文件到 `listeningTestSubmissions` collection。

### Firebase 設定步驟

1. 前往 [Firebase Console](https://console.firebase.google.com/)。
2. 建立新專案。
3. 在專案中新增「網頁應用程式」。
4. 複製 Firebase configuration，格式會像：

```javascript
const firebaseConfig = {
  apiKey: "......",
  authDomain: "......firebaseapp.com",
  projectId: "......",
  storageBucket: "......appspot.com",
  messagingSenderId: "......",
  appId: "......"
};
```

5. 打開 `index.html`，找到 `FIREBASE_CONFIG`，把設定值貼進去。
6. 在 Firebase Console 開啟 Firestore Database。
7. 建立資料庫時選擇正式模式或測試模式皆可，但正式使用前請調整安全規則。

### 測試用 Firestore Rules

如果只是短時間課堂測驗，可以先用日期限制開放新增資料。請把日期改成你的測驗結束日期。

```javascript
rules_version = '2';

service cloud.firestore {
  match /databases/{database}/documents {
    match /listeningTestSubmissions/{document} {
      allow create: if request.time < timestamp.date(2026, 12, 31);
      allow read, update, delete: if false;
    }
  }
}
```

這個規則讓學生只能新增作答紀錄，不能讀取、修改或刪除資料。老師要看成績時，請從 Firebase Console 或匯出資料查看。

若要讓全班紀錄集中到 Google Sheet：

1. 建立 Google Sheet。
2. 開啟「擴充功能」>「Apps Script」。
3. 貼上下方程式並部署成網路應用程式。
4. 將部署網址填入 `index.html` 裡的 `SUBMIT_ENDPOINT`。

```javascript
function doPost(e) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const data = JSON.parse(e.postData.contents);
  sheet.appendRow([
    data.title,
    data.score,
    data.correct,
    data.total,
    data.submittedAt,
    data.secondsUsed,
    data.autoSubmit ? "是" : "否",
    JSON.stringify(data.answers)
  ]);
  return ContentService.createTextOutput("ok");
}
```

部署 Apps Script 時，建議設定：

- 執行身分：我
- 誰可以存取：任何人

## 題目結構

- 共 20 題
- 每題 5 分，滿分 100 分
- 測驗時間 20 分鐘
- 題型為聽英文句子或短對話後選擇答案
- 使用瀏覽器 Web Speech API 播放英文聽力
