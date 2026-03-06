# 角色設定 (Role)
你是一位資深全端工程師，精通前端網頁開發（HTML5 / CSS3 / Vanilla JS）與 Firebase Realtime Database (Web SDK v9)。

# 專案概述 (Project Overview)
請協助開發一款大學課堂專用的「即時互動戀愛文字冒險遊戲（AVG）」。
學生透過手機掃描大螢幕的 QR Code 進入網頁，同時顯示上線人數，觀看同步的劇情畫面。當劇情到達「決策點」時，學生的手機會出現選項進行即時投票；老師端（大螢幕）可以即時看到雙方票數比例，並由老師點擊按鈕推動劇情前往多數人選擇的分支。

---

# 技術堆疊 (Tech Stack)
- **前端：** HTML5, CSS3, Vanilla JavaScript (無須編譯，純前端架構，方便直接部署)。
- **後端 / 資料同步：** Firebase Realtime Database。
- **UI 風格：** 經典日系戀愛遊戲風。畫面佔據上半部 70%（顯示背景或角色動畫圖），底部 30% 為半透明黑色對話框，對話文字需有「打字機（Typewriter）」逐字出現的效果。

---

# 核心功能與開發模組 (Core Features & Modules)

## 模組 1：雙身份視圖路由 (Role-based Views)
網頁僅靠 URL 參數來區分身份，例如 `index.html?role=admin` 為教師端，無參數 `index.html` 則為學生端。

* **教師端 (Admin/Host)：**
    * 顯示在投影幕上，需有適合大螢幕的自適應排版。
    * 畫面角落需顯示供學生掃描的 QR Code（連結至無參數的學生端網址）。
    * 對話框旁擁有隱藏版的「Next (下一步)」按鈕，用於手動推進劇情。
    * 進入投票環節時，畫面上即時顯示 A/B 選項的動態長條圖與當前票數。
* **學生端 (Player/Client)：**
    * 手機版自適應介面。
    * 無須操控劇情，純同步觀看與教師端一樣的背景圖片與對話文字。
    * 進入投票環節時，畫面出現 A 與 B 兩個大按鈕供點擊。
    * 點擊後按鈕鎖定（防止重複投票），並顯示「等待老師與其他人投票...」。

## 模組 2：Firebase 資料庫結構 (Database Schema)
1. Firebase 環境配置 (Environment Config)
請使用 Firebase Web SDK v9 (Modular) 進行開發，並初始化以下配置：

JavaScript
const firebaseConfig = {
  apiKey: "AIzaSyDPresURGBmmFWI60CXVNGpD4B2gkXdJMQ",
  authDomain: "lovegame-3ed57.firebaseapp.com",
  databaseURL: "https://lovegame-3ed57-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId: "lovegame-3ed57",
  storageBucket: "lovegame-3ed57.firebasestorage.app",
  messagingSenderId: "518614987215",
  appId: "1:518614987215:web:ef25e1d5826db774cc22ba",
  measurementId: "G-P2PSXP5L4P"
};
2. 資料庫架構與路徑 (Database Structure)
所有裝置必須即時監聽並讀取 Firebase 中的單一事實來源（Single Source of Truth），嚴禁使用 LocalStorage 儲存遊戲進度。

RTDB JSON 結構：

JSON
{
  "room_state": {
    "currentSceneId": "start",
    "isVotingActive": false,
    "votes": {
      "A": 0,
      "B": 0
    }
  }
}



## 模組 3：劇本與素材資料結構 (Story JSON)
const storyData = {
  "start": {
    text: "三週年紀念日，我們來到北海岸。然而，計畫永遠趕不上變化...",
    imagePath: "./image/01.png",
    next: "scene_01"
  },
  "scene_01": {
    text: "結衣興奮地湊過來：「寶貝，晚餐你想吃老字號牛排，還是這間網美餐酒館？」",
    imagePath: "./image/02.png",
    isVoting: true,
    options: {
      A: { label: "老字號牛排 (品質穩定)", next: "scene_02" },
      B: { label: "網美餐酒館 (滿足需求)", next: "scene_02" }
    }
  },
  "scene_02": {
    text: "高速公路上嚴重塞車。結衣暈車很不舒服，氣氛降到冰點...",
    imagePath: "./image/03.png",
    isVoting: true,
    options: {
      A: { label: "堅持原訂計畫", next: "scene_03" },
      B: { label: "立刻路邊停車找小吃", next: "scene_03" }
    }
  },
  "scene_03": {
    text: "終於坐下吃飯，但送錯餐了！結衣最討厭的香菜灑滿了盤子，她無奈地嘆口氣...",
    imagePath: "./image/04.png",
    isVoting: true,
    options: {
      A: { label: "要求重做 (矯正措施)", next: "calculate_ending" },
      B: { label: "跟她換餐 (情緒補救)", next: "calculate_ending" }
    }
  },
  "end_bad": {
    text: "行程走完了，但回程只有窒息的沉默。符合了所有規格，卻失去了整個市場。",
    imagePath: "./image/05.png",
    isVoting: false
  },
  "end_good": {
    text: "雖然計畫一團糟，但結衣覺得你處處為她著想。品質的最終定義，來自客戶的感受。",
    imagePath: "./image/06.png",
    isVoting: false
  }
};