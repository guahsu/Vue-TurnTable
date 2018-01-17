<template lang="pug">
.container
  //- 提示文字
  .alert.alert-info.text-center.mx-auto.w-75.mt-5.fade(v-if="config.showAlert", :class="{'alert--show': currentResult}") {{ currentResult }}
  //- 轉盤本體
  .turnTable(v-show="true")
    .turnTable__body
      canvas.turnTable__canvas(:class="animitionType")
    .turnTable__button(@click="animitionProc")
      .turnTable__arrow
      .turnTable__button__content
        span.turnTable__button__text {{ config.buttonText }}
  //- 設定按鈕
  .buttonArea
    input#showButtons.mr-2(type="checkbox", :checked="!showButtons", v-model="showButtons")
    label.text-sm-right(for="showButtons") 顯示設定
    br
    .buttons(v-show="showButtons")
      button.btn.btn-outline-danger.mr-4(data-toggle="modal", data-target="#resultList", v-if="config.autoStop") 轉出結果
      button.btn.btn-outline-info.mr-4(data-toggle="modal", data-target="#sector") 區塊編輯
      button.btn.btn-outline-warning(data-toggle="modal", data-target="#config") 轉盤設定
  //- 轉出結果視窗
  #resultList.modal.fade(v-if="config.autoStop")
    .modal-dialog(role="document")
      .modal-content
        .modal-header
          h5.modal-title 轉出結果
          button.close(data-dismiss="modal")
            span &times;
        .modal-body
          .table-responsive
            table.table.table-bordered.table-sm
              thead.text-center
                tr
                  th.text-center(scope="col") No
                  th.text-center(scope="col") 內容
              tbody.text-center
                tr(v-for="result, index in resultList")
                  td.text-center {{ index + 1 }}
                  td.text-center {{ result }}
        .modal-footer
          button.btn.btn-sm.btn-outline-secondary.ml-2.px-3(@click.prevent="resultList = []") 清空資訊
          button.btn.btn-sm.btn-secondary.ml-2.px-3(data-dismiss="modal") 關閉
  //- 區塊設定視窗
  #sector.modal.fade(:style="{'opacity': editAreaOpacity}")
    .modal-dialog(role="document")
      .modal-content
        .modal-header
          h5.modal-title 區塊編輯
          label.text-sm.ml-3 | 背景透明度:
          input(type="range", min="0.1", max="1", step="0.1", v-model="editAreaOpacity")
          button.close(data-dismiss="modal")
            span &times;
        .modal-body
          form.form-inline(@submit.prevent="addGiftData")
            .form-group.w-100.mb-2
              label.mr-2 編輯模式:
              .form-check.ml-4.ml-sm-0.mr-sm-2
                input#editBySet.form-check-input(type="radio", name="editMode", :value="true", v-model="sectorEditMode", checked)
                label(for="editBySet") 單塊新增
              .form-check.ml-4.ml-sm-0
                input#editByData.form-check-input(type="radio", name="editMode", :value="false", v-model="sectorEditMode")
                label(for="editByData") 資料編輯(Array-Object)
            //- 單塊新增模式
            .editBySet.w-100(v-if="sectorEditMode")
              .form-group.w-100.mb-2
                label.mr-2 背景顏色:
                input.mr-2(v-if="!isMobile", type="color", v-model="sector.backgroundColor")
                input.form-control.form-control-sm(type="text", v-model="sector.backgroundColor", :placeholder='getDefaultSectorColor()')
              .form-group.w-100.mb-2
                label.mr-2 文字顏色:
                input.mr-2(v-if="!isMobile", type="color", v-model="sector.textColor")
                input.form-control.form-control-sm(type="text", v-model="sector.textColor", :placeholder='config.textColor')
              .form-group.w-100.mb-2
                label.mr-2 文字尺寸:
                input.form-control.form-control-sm(type="number", placeholder="請輸入文字尺寸", v-model="sector.textSize")
              .form-group.w-100.mb-2
                label.mr-2 文字內容:
                input.form-control.form-control-sm(type="text", placeholder="請輸入文字內容", v-model="sector.text")
              .form-group.w-100.mb-2
                label.mr-2 出現機率:
                input.form-control.form-control-sm(type="number", min="1", step="0.1", v-model.number="sector.chance")
              .form-group.w-100.mb-2
                button.btn.btn-sm.btn-success.px-5(type="submit") 新增
            //- 資料編輯模式
            .editByDate.w-100.my-2(v-else)
              textarea.form-control.form-control-sm.w-100(:value="showGiftsToTextArea", @input="setGiftsByTextArea")
            //- 設定結果顯示區
            .table-responsive
              table.table.table-bordered.table-sm.mt-3
                thead.text-center
                  tr
                    th(scope="col") 區塊占比
                    th(scope="col") 背景顏色
                    th(scope="col") 文字顏色
                    th(scope="col") 文字內容
                    th(scope="col") 移除
                tbody.text-center
                  tr(v-for="gift, index in gifts")
                    td {{ getChance(gift.chance) }}
                    td(:style="{'background-color': gift.backgroundColor || getDefaultSectorColor(index)}")
                      | {{ gift.backgroundColor || getDefaultSectorColor(index) }}
                    td(:style="{'background-color': gift.textColor || config.textColor }")
                      | {{ gift.textColor || config.textColor }}
                    td {{ gift.text }}
                    td
                      button.btn.btn-sm.btn-outline-danger(@click.prevent="removeGiftData(index)") 移除
        .modal-footer
          button.btn.btn-sm.btn-outline-success(@click.prevent="setGiftDataToLocalStorage") 儲存設定
          button.btn.btn-sm.btn-outline-secondary.ml-2.px-3(@click.prevent="gifts = getDefaultGfits()") 清空重設
          button.btn.btn-sm.btn-secondary.ml-2.px-3(data-dismiss="modal") 關閉
  //- 轉盤設定視窗
  #config.modal.fade(:style="{'opacity': editAreaOpacity}")
    .modal-dialog(role="document")
      .modal-content
        .modal-header
          h5.modal-title 轉盤設定
          label.text-sm.ml-3 | 背景透明度:
          input(type="range", min="0.1", max="1", step="0.1", v-model="editAreaOpacity")
          button.close(data-dismiss="modal")
            span &times;
        .modal-body
          form.form-inline(@submit.prevent="")
            .form-group.w-100.mb-2
              label.mr-2 停止模式:
              .form-check.ml-4.ml-sm-0.mr-sm-2
                input#autoStopTrue.form-check-input(type="radio", name="autoStop", :value="true", v-model="config.autoStop", checked)
                label(for="autoStopTrue") 自動
              .form-check.ml-4.ml-sm-0
                input#autoStopFalse.form-check-input(type="radio", name="autoStop", :value="false", v-model="config.autoStop")
                label(for="autoStopFalse") 手動
            .form-group.w-100.mb-2(v-if="config.autoStop")
              label.mr-2 轉出資訊:
              .form-check.ml-4.ml-sm-0.mr-sm-2
                input#showAlertTrue.form-check-input(type="radio", name="showAlert", :value="true", v-model="config.showAlert", checked)
                label(for="showAlertTrue") 顯示
              .form-check.ml-4.ml-sm-0
                input#showAlertFalse.form-check-input(type="radio", name="showAlert", :value="false", v-model="config.showAlert")
                label(for="showAlertFalse") 關閉
            .form-group.w-100.mb-2
              label.mr-2 {{ config.autoStop ? '動畫時間長度(s):' : '轉10圈的轉速(s):' }}
              input.form-control.form-control-sm(type="number", min="1", step="1", v-model.number="config.runTime")
            .form-group.w-100.mb-2
              label.mr-2 轉盤尺寸設定(px):
              input.form-control.form-control-sm(type="number", step="1", min="150", v-model.number="config.baseSize")
            .form-group.w-100.mb-2(v-if="config.autoStop")
              label.mr-2 回彈角度設定(百分比）:
              input.form-control.form-control-sm(type="number", step="0.01", min="0.1", max="1", v-model.number="config.rollBackRange", :readonly="!config.autoStop")
            .form-group.w-100.mb-2
              label.mr-2 區塊預設色(單數):
              input.mr-2(v-if="!isMobile", type="color", v-model="config.singleColor")
              input.form-control.form-control-sm(type="text", v-model="config.singleColor")
            .form-group.w-100.mb-2
              label.mr-2 區塊預設色(雙數):
              input.mr-2(v-if="!isMobile", type="color", v-model="config.doubleColor")
              input.form-control.form-control-sm(type="text", v-model="config.doubleColor")
            .form-group.w-100.mb-2
              label.mr-2 區塊邊框寬度
              input.form-control.form-control-sm(type="number", min="1", step="1", v-model.number="config.borderWidth")
            .form-group.w-100.mb-2
              label.mr-2 按鈕顏色:
              input.mr-2(v-if="!isMobile", type="color", v-model="config.buttonColor")
              input.form-control.form-control-sm(type="text", v-model="config.buttonColor")
            .form-group.w-100.mb-2
              label.mr-2 按鈕文字:
              input.form-control.form-control-sm(type="text", v-model="config.buttonText")
        .modal-footer
          button.btn.btn-sm.btn-outline-success(@click.prevent="setConfigToLocalStorage") 儲存設定
          button.btn.btn-sm.btn-outline-secondary.ml-2.px-3(@click.prevent="config = getDefaultConfig()") 重新設定
          button.btn.btn-sm.btn-secondary.ml-2.px-3(data-dismiss="modal") 關閉
</template>

<script>
  export default {
    name: 'TurnTable',
    data() {
      return {
        /** 轉動狀態 */
        isRunning: false,
        /** 手機瀏覽器，用來開關input-color */
        isMobile: false,
        /** 編輯模式: true=區塊設定模式、false="資料編輯模式" */
        sectorEditMode: true,
        /** 是否顯示設定按鈕 */
        showButtons: true,
        /** 編輯視窗背景透明度 */
        editAreaOpacity: 1,
        /** 獎項中獎角度儲存 */
        giftsDeg: [],
        /** 轉動狀態 */
        turnStatus: {
          currentDeg: 0,
          targetDeg: 0,
          rollBackDeg: 0,
        },
        /** 獎項設定資料 */
        gifts: [],
        defaultGifts: [
          { chance: 10, text: 'RED', textColor: '', textSize: '', backgroundColor: '' },
          { chance: 10, text: 'BLUE', textColor: '', textSize: '', backgroundColor: '' },
        ],
        /** 中獎清單 */
        currentResult: '',
        resultList: [],
        /** 區塊設定 */
        sector: {},
        defaultSector: {
          chance: 10,
          text: '',
          textColor: '',
          textSize: this.defaultSectorTextSize,
          backgroundColor: this.defaultSectorColor,
        },
        /** 轉盤設定 */
        config: {},
        defaultConfig: {
          autoStop: true,
          runTime: 6,
          rollBackRange: 0.25,
          showAlert: true,
          baseSize: 500,
          singleColor: '#b0e0e6',
          doubleColor: '#e4c6d0',
          borderWidth: 1,
          borderColor: '#ffffff',
          textColor: '#ffffff',
          buttonColor: '#f19393',
          buttonText: 'GO',
        },
      };
    },
    watch: {
      isRunning(boolean) {
        // 手動模式要將動畫停止
        if (!this.config.autoStop) {
          const state = boolean ? 'running' : 'paused';
          document.documentElement.style.setProperty('--animitionState', state);
        }
      },
      /** 偵聽當資料變更就重建轉盤 */
      gifts() {
        this.buideTurnTable();
      },
      config: {
        handler() {
          this.buideTurnTable();
        },
        deep: true,
      },
      turnStatus: {
        handler() {
          this.updateturnStatus();
        },
        deep: true,
      },
    },
    computed: {
      /** 轉盤資料字串 */
      showGiftsToTextArea() {
        return JSON.stringify(this.gifts);
      },
      /** 動畫Class判斷 */
      animitionType() {
        return {
          'turnTable__canvas--manualTrun': !this.config.autoStop,
          'turnTable__canvas--autoTrun': this.isRunning && this.config.autoStop,
        };
      },
      /** 計算區塊機率值總和 */
      countDataChance() {
        let totalChance = 0;
        this.gifts.forEach((data) => {
          totalChance += data.chance;
        });
        return totalChance;
      },
      /** 預設區塊色 */
      defaultSectorColor() {
        return this.getDefaultSectorColor();
      },
      /** 預設區塊文字大小 */
      defaultSectorTextSize() {
        return Math.floor(this.config.baseSize / 15);
      },
    },
    methods: {
      /** 檢查手機瀏覽器（關閉input-color) */
      checkMobile() {
        return (/(iPhone|iPad|iPod|iOS|Android)/i.test(navigator.userAgent));
      },
      /** 計算區塊機率% */
      getChance(data) {
        return `${((data / this.countDataChance) * 100).toFixed(1)}%`;
      },
      /** 取得預設區塊色彩 */
      getDefaultSectorColor(index) {
        const number = index || this.gifts.length;
        return number % 2 === 0 ? this.config.doubleColor : this.config.singleColor;
      },
      /** 新增區塊資料 */
      addGiftData() {
        this.gifts.push(this.sector);
        this.setDefaultSector();
      },
      /** 移除區塊資料 */
      removeGiftData(index) {
        this.gifts.splice(index, 1);
      },
      /** 取得區塊資料預設值 */
      getDefaultGfits() {
        return Array.from(this.defaultGifts);
      },
      /** 設定區塊資料(透過區塊編輯Object) */
      setGiftsByTextArea(e) {
        this.gifts = JSON.parse(e.target.value);
      },
      /** 儲存區塊設定值(LocalStorage) */
      setGiftDataToLocalStorage() {
        const localDatas = JSON.stringify(this.gifts);
        localStorage.setItem('turnTableGiftDatas', localDatas);
      },
      /** 讀取區塊設定值(LocalStorage) */
      getGiftDataFromLocalStorage() {
        const localDatas = JSON.parse(localStorage.getItem('turnTableGiftDatas'));
        return localDatas;
      },
      /** 設定區塊預設值 */
      setDefaultSector() {
        this.sector = Object.assign({}, this.defaultSector);
      },
      /** 取得轉盤預設值 */
      getDefaultConfig() {
        // 計算當前視窗寬高，取低值*0.6當基準值設定轉盤大小
        const innerHeight = window.innerHeight;
        const innerWidth = window.innerWidth;
        this.defaultConfig.baseSize = innerHeight > innerWidth ?
          Math.floor(innerWidth * 0.8) : Math.floor(innerHeight * 0.6);
        return Object.assign({}, this.defaultConfig);
      },
      /** 儲存轉盤設定值(LocalStorage) */
      setConfigToLocalStorage() {
        const localConfig = JSON.stringify(this.config);
        localStorage.setItem('turnTableConfig', localConfig);
      },
      /** 讀取轉盤設定值(LocalStorage) */
      getConfigFromLocalStorage() {
        const localConfig = JSON.parse(localStorage.getItem('turnTableConfig'));
        return localConfig;
      },
      /** 轉盤內容繪製 */
      drawCanvas() {
        const centerPoint = this.config.baseSize / 2;
        const turnTable = document.querySelector('.turnTable__canvas');
        const ctx = turnTable.getContext('2d');
        turnTable.setAttribute('width', this.config.baseSize);
        turnTable.setAttribute('height', this.config.baseSize);
        this.giftsDeg = [];
        let lastAngle = 0;
        // 內部區塊繪製
        this.gifts.forEach((data, index) => {
          // 計算角度(全部資料機率 / 單片機率 * 360)
          const deg = (data.chance / this.countDataChance) * 360;
          // 計算弧度(角度 * PI / 180)，
          const angle = deg * (Math.PI / 180);
          // 儲存角度範圍
          this.giftsDeg[index] = {
            from: index === 0 ? 0 : this.giftsDeg[index - 1].to,
            to: index === 0 ? deg : this.giftsDeg[index - 1].to + deg,
            name: data.text,
          };
          // 開始繪製
          ctx.save();
          ctx.beginPath();
          ctx.translate(centerPoint, centerPoint);
          ctx.moveTo(0, 0);
          // 旋轉弧度 = 上次的區塊弧度(初始0)
          ctx.rotate(lastAngle);
          // 繪製區塊，半徑-外框線 避免 框線擠到canvas邊框
          ctx.arc(0, 0, centerPoint - this.config.borderWidth, 0, angle, false);
          // 更新最後一次的結束角度
          lastAngle += angle;
          // 區塊底色填充
          ctx.fillStyle = data.backgroundColor || this.getDefaultSectorColor(index);
          ctx.fill();
          /** 邊框繪製 */
          ctx.lineWidth = this.config.borderWidth;
          ctx.strokeStyle = this.config.borderColor;
          ctx.stroke();
          // 內容文字繪製
          ctx.rotate(angle / 2);
          ctx.fillStyle = data.textColor || this.config.textColor;
          ctx.font = data.textSize ?
            `${data.textSize}px Microsoft JhengHei` : `${this.defaultSectorTextSize}px Microsoft JhengHei`;
          ctx.textBaseline = 'middle';
          ctx.fillText(data.text, centerPoint / 1.8, 0);
          //
          ctx.restore();
        });
        // 自動停止模式動畫結束時要跑autoTurnStop function
        if (this.config.autoStop) {
          turnTable.addEventListener('animationend', this.autoTurnStop);
        } else {
          turnTable.removeEventListener('animationend', this.autoTurnStop);
        }
      },
      /** 轉動模式 */
      animitionProc() {
        if (this.config.autoStop) {
          this.autoTurnStart();
        } else {
          this.manualTrun();
        }
      },
      /** 手動模式(手動停止) */
      manualTrun() {
        this.isRunning = !this.isRunning;
      },
      /** 自動模式(自動停止) */
      autoTurnStart() {
        // 如果正在執行中不動作
        if (this.isRunning) return;
        // 取得隨機角度(預設至少跑10圈)
        const randomDeg = Math.floor(Math.random() * (360 - 0)) + 3600;
        // 取得隨機回彈角度
        const randomRollBackDeg = (Math.random() * this.config.rollBackRange) + 1;
        // 設定回彈角度
        this.turnStatus.rollBackDeg = randomRollBackDeg < 1.01 ? 1 : randomRollBackDeg;
        // 設置動畫的目標角度
        this.turnStatus.targetDeg = randomDeg;
        // 開始旋轉動畫
        this.isRunning = true;
      },
      /** 自動模式停止事件(由繪製區的addEventListener('animationend')觸發) */
      autoTurnStop() {
        // 關閉動畫
        this.isRunning = false;
        // 紀錄本次角度
        this.turnStatus.currentDeg = Math.floor(this.turnStatus.targetDeg % 360);
        // 計算中獎角度
        const nowDeg = 360 - this.turnStatus.currentDeg;
        // 顯示獎品資料
        this.giftsDeg.forEach((gift) => {
          if (nowDeg >= gift.from && nowDeg <= gift.to) {
            this.resultList.push(gift.name);
            this.currentResult = gift.name;
            setTimeout(() => {
              this.currentResult = '';
            }, 3500);
          }
        });
      },
      /** 建立轉盤 */
      buideTurnTable() {
        // CSS值設定
        document.documentElement.style.setProperty('--turnTableSize', `${this.config.baseSize + 20}px`);
        document.documentElement.style.setProperty('--buttonSize', `${this.config.baseSize / 3}px`);
        document.documentElement.style.setProperty('--buttonFontSize', `${this.config.baseSize / 10}px`);
        document.documentElement.style.setProperty('--arrowHeight', `${this.config.baseSize / 7}px`);
        document.documentElement.style.setProperty('--arrowWidth', `${this.config.baseSize / 5}px`);
        document.documentElement.style.setProperty('--buttonColor', `${this.config.buttonColor}`);
        document.documentElement.style.setProperty('--runTime', `${this.config.runTime}s`);
        document.documentElement.style.setProperty('--animitionState', 'paused');
        // 清空轉出結果
        this.resultList = [];
        // 繪製轉盤
        this.drawCanvas();
      },
      /** 更新轉盤角度資訊 */
      updateturnStatus() {
        document.documentElement.style.setProperty('--targetDeg', `${this.turnStatus.targetDeg}deg`);
        document.documentElement.style.setProperty('--currentDeg', `${this.turnStatus.currentDeg}deg`);
        document.documentElement.style.setProperty('--rollBackDeg', `${this.turnStatus.rollBackDeg}`);
      },
    },
    beforeMount() {
      // 檢查手機模式
      this.isMobile = this.checkMobile();
      // 區塊新增的基礎預設值設定
      this.setDefaultSector();
      // 若沒有儲存在LocalStorage中的資料就用預設值
      this.config = this.getConfigFromLocalStorage() || this.getDefaultConfig();
      this.gifts = this.getGiftDataFromLocalStorage() || this.getDefaultGfits();
    },
    mounted() {
      // 建立轉盤
      this.buideTurnTable();
      // 偵測空白鍵，使空白鍵可觸發轉動及停止
      document.addEventListener('keypress', (e) => {
        if (e.keyCode === 32) this.animitionProc();
      });
    },
  };
</script>

<style lang="scss">
  //圖形樣式變數
  $turnTableSize: var(--turnTableSize);
  $buttonSize: var(--buttonSize);
  $buttonColor: var(--buttonColor);
  $buttonFontSize: var(--buttonFontSize);
  $arrowWidth: var(--arrowWidth);
  $arrowHeight: var(--arrowHeight);
  //轉動執行用變數
  $runTime: var(--runTime);
  $targetDeg: var(--targetDeg);
  $currentDeg: var(--currentDeg);
  $rollBackDeg: var(--rollBackDeg);
  $animitionState: var(--animitionState);

  @mixin positionCenter($args:null) {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%) $args;
  }

  * {
    font-family: Microsoft JhengHei;
  }

  html {
    font-size: 14px;
  }

  .alert {
    font-size: 20px;
    &--show {
      z-index: 1;
      opacity: 0.7;
      transition: opacity 0.5s linear;
    }
  }

  .buttonArea {
    position: fixed;
    display: inline-block;
    right: 3vh;
    bottom: 3vh;
  }

  .turnTable {
    @include positionCenter();
    width: $turnTableSize;
    height: $turnTableSize;
    border-radius: 50%;
    box-shadow: 0px 2px 10px rgba(0, 0, 0, 0.5);
    user-select: none;
    &__body {
      @include positionCenter();
      font-size: 0px;
    }
    &__canvas {
      transform: rotate($targetDeg);
    }
    &__canvas--autoTrun {
      animation: AutoTrun $runTime forwards cubic-bezier(0.1, 0, 0, $rollBackDeg);
    }
    &__canvas--manualTrun {
      animation: ManualTrun $runTime infinite linear forwards;
      animation-play-state: $animitionState;
    }
    &__button {
      @include positionCenter();
      padding: 10px;
      border-radius: 50%;
      background-color: #fff;
      box-shadow: 0px 1px 5px rgba(0, 0, 0, 0.5);
      color: #fff;
      cursor: pointer;
    }
    &__button__content {
      width: $buttonSize;
      height: $buttonSize;
      border-radius: 50%;
      background-color: $buttonColor;
    }
    &__button__text {
      @include positionCenter();
      color: #fff;
      font-weight: 900;
      font-size: $buttonFontSize;
    }
    &__arrow {
      @include positionCenter(rotate(270deg));
      left: 100%;
      border: $arrowHeight solid transparent;
      border-top: $arrowWidth solid $buttonColor;
    }
  }

  @keyframes AutoTrun {
    from {
      transform: rotate($currentDeg);
    }
    to {
      transform: rotate($targetDeg);
    }
  }

  @keyframes ManualTrun {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(7200deg);
    }
  }
</style>
