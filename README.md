# Vue.js 大～～轉～～盤
![](https://guahsu.io/2018/01/vue-turntable/1.png) 

## 功能
### 1. 區塊設定
設定每個區塊的資料，機率可以隨意數字最後會加總去算成百分比，  
單塊設定可拖曳編輯(使用[Vue.Draggable](https://github.com/SortableJS/Vue.Draggable)製作的拖曳功能)。
![](https://guahsu.io/2018/01/vue-turntable/2.png) 

### 2. 轉盤設定
手動模式：就是單純下個`paused`把動畫停下來，  
自動模式：可以設定回彈角度(轉盤動畫結束前的位置反彈)
![](https://guahsu.io/2018/01/vue-turntable/3.png) 

### 3. 設定檔
可以將不同的區塊＆轉盤設定存檔，因應各種場景切換
![](https://guahsu.io/2018/01/vue-turntable/4.png) 

### 開發
分別都設定在data中，方便配合各處傳值來作設定：）

### TODO
有機會有時間想在套個firebase儲存設定資訊，  
練習登入功能且可以讓朋友作設定分享出去轉～

### 其他
因為仍在學習中，有錯誤的或是寫很怪的地方希望能在多多指教了：）

### 使用
PUG / SCSS / Vue.js / Bootstrap4 / [Vue.Draggable](https://github.com/SortableJS/Vue.Draggable)

#### 更新紀錄
2018/01/13 修正排版，新增轉盤設定檔LocalStorage儲存功能，增修部分備註。  
2018/01/17 修正轉盤未置中問題，新增區塊編輯文字大小功能。  
2018/01/18 更改canvas比例依據裝置解析度調整，調整預設文字隨字數控制比例。
2018/01/19 合併轉盤設定與排版＆修改區塊編輯方式，新增設定存檔功能。