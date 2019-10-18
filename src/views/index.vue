<template>
  <div id="app" @click="isChangeMenu = []">
    <div class="sidebar">
      <i id="logo"></i>
      <label id="addfile">
        <input type="file" @change="upload">
      </label>
      <ul class="nav">
        <li><a href="#" @click.prevent="sort.onlyImportant = false">我的雲端</a></li>
        <li><a href="#" @click.prevent="alert">共用項目</a></li>
        <li><a href="#" @click.prevent="sort.onlyImportant = true">星號文件</a></li>
        <li>
          <a href="#" @clik.prevent>色彩文件</a>
          <div>
            <i class="red" @click="alert"></i>
            <i class="green" @click="alert"></i>
            <i class="purple" @click="alert"></i>
            <i class="yellow" @click="alert"></i>
            <i class="blue" @click="alert"></i>
            <i class="brown" @click="alert"></i>
          </div>
        </li>
        <li><a href="#" @click.prevent="alert">設定</a></li>
      </ul>
      <ul class="driveInfo">
        <li>雲端空間(5GB)：</li>
        <li class="memory"><i></i></li>
        <li>已使用0.01GB</li>
        <li><a href="#" @click.prevent="alert">升級空間容量！</a></li>
      </ul>
    </div>
    <div class="top">
      <h1>我的雲端</h1>
      <i class="itemDisplay" :class="{'block' : sort.display === 'block',
      'list' : sort.display === 'list'}"
      @click="sort.display = sort.display === 'block' ? 'list' : 'block'"></i>
      <input type="text" placeholder="此功能尚未完成">
      <i id="a"></i>
    </div>
    <div class="content" @dragenter.prevent.stop="openDrag">
      <!-- 上方工具列 -->
      <ul class="toolbar">
        <li>
          排序方式
            <select v-model="sort.sortBy">
              <option value="createTimeUp">依日期<i> ⬆</i></option>
              <option value="createTimeDown">依日期<i> ⬇</i></option>
              <option value="sizeUp">依大小<i> ⬆</i></option>
              <option value="sizeDown">依大小<i> ⬇</i></option>
              <option value="name">依名稱</option>
              <option value="type">依類型</option>
            </select>
        </li>
        <li v-if="selected.length === 0" @click="selectAll">全部選取</li>
        <li v-else @click="selected = []">已選取 ({{selected.length}})</li>
        <li @click="folder.isOpen = true">新增資料夾</li>
        <li><i id="shareUser" @click.prevent="alert"></i></li>
        <li><i id="delete" @click="removeFiles"></i></li>
        <li><i id="info" @click.prevent="alert"></i></li>
      </ul>
      <!-- 列表頭 -->
      <ul class="listHead" v-if="sort.display === 'list'">
        <li>檔名</li>
        <li>日期
          <i v-if="sort.sortBy === 'createTimeUp'"> ⬆</i>
          <i v-if="sort.sortBy === 'createTimeDown'"> ⬇</i>
        </li>
        <li>檔案大小
          <i v-if="sort.sortBy === 'sizeUp'"> ⬆</i>
          <i v-if="sort.sortBy === 'sizeDown'"> ⬇</i>
        </li>
        <li>擁有者</li>
      </ul>
      <!-- 檔案 -->
      <ul class="fileBox" :class="{'list' : sort.display === 'list'}">
        <li v-for="(item, key) in sortFile" :key="key"
        :class="[{'selected' : selected.includes(key), 'important': item.isImportant}, item.type]"
        @click.exact="selectItem(key)" @click.exact.meta.left="selectItembyCtrl(key)"
        @click.exact.ctrl.left="selectItembyCtrl(key)">
          <!-- 照片 & 圖像ICON -->
          <img class="imgSnap" :src="item.url" v-if="item.type === 'img'">
          <i class="fileIcon" v-else></i>
          <i class="favIcon"></i>
          <!-- 名稱 -->
          <h3>{{item.name}}</h3>
          <!-- 列表資訊 -->
          <div class="listInfo" v-if="sort.display === 'list'">
            <h4>{{item.createTime | timeFilter }}</h4>
            <h4>{{item.size | sizeFilter }}</h4>
            <h4>我</h4>
          </div>
          <!-- 檔案選單 -->
          <i class="menuSwitch" @click.stop="openMenu(key)"></i>
          <ul class="editMenu" v-if="isChangeMenu[0] === key">
            <li>重新命名</li>
            <li @click.stop="downloadItem(item.url)">下載</li>
            <li>共享</li>
            <li @click.stop="favFile(key)">加星號</li>
            <li>更改顏色</li>
            <li>移動到...</li>
            <li @click.stop="item.type !== 'file' ?
            removeFile(item.type, item.id, item.path):removeFile(item.type, item.id)">刪除</li>
          </ul>
        </li>
      </ul>
      <!-- 上傳遮罩 -->
      <div class="dropFile" @dragenter.prevent.stop @dragover.prevent.stop
      @drop.prevent="upload" v-if="isDrop">
        <i id="dropLogo"></i>
      </div>
      <!-- 提醒 -->
      <ul class="notice">
        <transition-group name="notice-list">
          <li v-for="(item, index) in notice" :key="item">
            <i class="noticeIcon">!</i>
            <h3>{{item}}</h3>
            <i class="closeNotice" @click="notice.splice(index, 1)">X</i>
          </li>
        </transition-group>
      </ul>
      <!-- 上傳進度 -->
      <div class="updateStatus" v-if="updateInfo.isOpen">
        <ul class="statusTitle">
          <li><h3>上傳中...</h3></li>
          <li><h4>約 {{updateInfo.minute}} 分鐘</h4></li>
          <li><i class="closeIcon" @click="updateInfo.isOpen = false"></i></li>
        </ul>
        <ul class="statusList">
          <li v-for="(item, key) in updateInfo.progress" :key="key">
            <i class="fileIcon"></i>
            <h4>{{item.name}}</h4>
            <div class="progress">
              <i :style="{width: item.success * 90 + 'px'}"></i>
            </div>
            <i class="closeIcon" @click="canceltask(key)"></i>
          </li>
        </ul>
      </div>
      <!-- 新增資料夾 -->
      <div class="addFolder" v-if="folder.isOpen">
        <div class="comfirm">
          <input type="text" placeholder="命名新資料夾" v-model="folder.newFolder"
          @keyup.enter="addFolder">
          <h4 @click="closeFolder">取消</h4>
          <h4 @click="addFolder">確定<i></i></h4>
        </div>
      </div>
      <!-- 檔案資訊 -->
      <!-- <div class="infoWrap" v-if="selected.length !== 0">
        <div class="infoHead">
          <h2>空間容量</h2>
          <i class="fileIcon"></i>
          <i class="favIcon"></i>
          <i class="closeIcon"></i>
        </div>
        <h3>詳細資料</h3>
        <h3>活動紀錄</h3>
        <ul class="fileinfo">
          <li>
            <i class="shareIcon"></i>|
            <i class="linkIcon"></i>
          </li>
          <li>類型 | Google雲端資料夾</li>
          <li>位置 | 我的雲端硬碟</li>
          <li>擁有者 | 我</li>
          <li>上次修改時間 | {{ drive[selected[0]].lastEditedTime | timeFilter }}</li>
          <li>上次開啟時間 | {{ drive[selected[0]].lastEditedTime | timeFilter }}</li>
          <li>建立日期 | {{ drive[selected[0]].createTime | timeFilter }}</li>
        </ul>
      </div> -->
    </div>
  </div>
</template>

<script>
const storage = window.firebase.storage();
const db = window.firebase.firestore();

export default {
  name: 'index',
  data() {
    return {
      // db根目錄資料
      drive: [],
      selected: [],
      // 右鍵選單
      isChangeMenu: [],
      isDrop: false,
      // 左下提醒事項
      notice: [],
      // 右下上傳小視窗
      updateInfo: {
        isOpen: false,
        minute: 3,
        progress: [],
      },
      // storage上傳值(取消上傳用)
      task: '',
      // 資料夾
      folder: {
        isOpen: false,
        newFolder: '',
      },
      sort: {
        sortBy: 'createTimeUp',
        onlyImportant: false,
        display: 'block',
      },
    };
  },
  firestore: {
    drive: db.collection('BUCKET'),
  },
  filters: {
    timeFilter(value) {
      const dt = new Date(value);
      return `${dt.getFullYear()} 年 ${dt.getMonth() + 1} 月 ${dt.getDate()} 號`;
    },
    sizeFilter(value) {
      if (String(value).length <= 6) {
        return `${(value / 1024).toFixed(1)} KB`;
      } else if (String(value).length <= 9) {
        return `${(value / 1024 / 1024).toFixed(1)} MB`;
      } else if (String(value).length > 9) {
        return `${(value / 1024 / 1024).toFixed(1)} GB`;
      }
      return false;
    },
  },
  methods: {
    selectItem(value) {
      if (!this.selected.includes(value)) {
        this.selected = [value];
      } else {
        this.selected = [];
      }
    },
    selectItembyCtrl(value) {
      if (!this.selected.includes(value)) {
        this.selected.push(value);
      } else {
        this.selected.splice(this.selected.indexOf(value), 1);
      }
    },
    openMenu(value) {
      if (this.isChangeMenu[0] === value) {
        this.isChangeMenu = [];
        return;
      }
      this.isChangeMenu = [value];
    },
    openDrag() {
      const vm = this;
      this.isDrop = true;
      setTimeout(() => { vm.isDrop = false; }, 1000);
    },
    upload(e) {
      const fileList = e.type === 'drop' ? e.dataTransfer.files[0] : e.target.files[0];
      // 不接受資料夾
      if (fileList.type === '') { return; }
      const vm = this;
      // firestore 設定
      const config = {
        name: String(fileList.name.split('.', 1)),
        type: vm.convertorFileType(fileList.name.split('.').pop()),
        createTime: Number(new Date()),
        lastEditedTime: fileList.lastModified,
        isImportant: false,
        color: 'primary',
        size: fileList.size,
        path: fileList.name,
        id: vm.drive.length + 1,
      };
      // 傳上 storage : ref(檔案名稱)
      let newLen = 0;
      this.updateInfo.isOpen = true;
      this.task = storage.ref(fileList.name).put(fileList);
      this.task.on('state_changed', (snap) => {
        if (snap.bytesTransferred === 0) {
          newLen = this.updateInfo.progress.length;
          vm.updateInfo.progress.push({
            name: config.name,
            success: 0,
          });
        }
        vm.updateInfo.progress[newLen].success = snap.bytesTransferred / snap.totalBytes;
      }, () => {
        // 失敗訊息
        const newText = `${config.name}上傳失敗`;
        vm.notice.push(newText);
        setTimeout(() => {
          vm.notice.splice(vm.notice.indexOf(newText), 1);
        }, 3000);
        return false;
      }, () => {
        // 上傳成功關閉上傳視窗
        vm.updateInfo.progress.splice(newLen, 1);
        if (vm.updateInfo.progress.length === 0) {
          vm.updateInfo.isOpen = false;
        }
      });
      this.task.then((item) => {
        item.ref.getDownloadURL().then((value) => {
          config.url = value;
          // 上傳到 firestore
          const path = this.drive.length === 0 ? this.convertorCol(this.drive.length + 1) :
            this.convertorCol(Number(this.drive[this.drive.length - 1].id.substr(4, 7)) + 1);
          db.collection('BUCKET').doc(path).set(config).then(() => {
            // 上傳完畢提示
            const newText = `已將${config.name}移動至我的雲端`;
            vm.notice.push(newText);
            setTimeout(() => {
              vm.notice.splice(vm.notice.indexOf(newText), 1);
            }, 3000);
          });
        });
      });
      // 關閉上傳遮罩
      this.isDrop = false;
    },
    canceltask(key) {
      this.task.cancel();
      this.updateInfo.progress.splice(key, 1);
      this.updateInfo.isOpen = false;
    },
    addFolder() {
      const vm = this;
      const config = {
        name: this.folder.newFolder,
        type: 'file',
        createTime: Number(new Date()),
        lastEditedTime: Number(new Date()),
        isImportant: false,
        color: 'primary',
        size: 0,
        id: this.drive.length + 1,
      };
      const path = this.drive.length === 0 ? this.convertorCol(this.drive.length + 1) :
        this.convertorCol(Number(this.drive[this.drive.length - 1].id.substr(4, 7)) + 1);
      db.collection('BUCKET').doc(path).set(config).then(() => {
        // 上傳完畢提示
        const newText = '已新增一個資料夾！';
        vm.notice.push(newText);
        setTimeout(() => {
          vm.notice.splice(vm.notice.indexOf(newText), 1);
        }, 3000);
      });
      this.closeFolder();
    },
    closeFolder() {
      this.folder.isOpen = false;
      this.folder.newFolder = '';
    },
    convertorCol(value) {
      // 轉換集合名稱
      if (value < 10) {
        return `file00${String(value)}`;
      } else if (value < 100) {
        return `file0${String(value)}`;
      }
      return `file${value}`;
    },
    convertorFileType(value) {
      // 轉換檔案類型
      const newValue = value.toUpperCase();
      if (newValue === 'JPG' || newValue === 'JPEG' || newValue === 'PNG' ||
      newValue === 'SVG' || newValue === 'GIF' || newValue === 'BMP') {
        return 'img';
      } else if (newValue === 'MP4') {
        return 'video';
      } else if (newValue === 'ZIP' || newValue === 'RAR') {
        return 'zip';
      }
      return value;
    },
    removeFile(type, id, path) {
      if (type !== 'file') { storage.ref(path).delete(); }
      db.collection('BUCKET').doc(id).delete().then(() => {
        this.selected.splice(0, 1);
        if (this.selected.length === 0) { this.openMenu(); }
      });
    },
    removeFiles() {
      let item = '';
      const vm = this;
      for (let i = 0; i < this.selected.length; i += 1) {
        item = this.drive[this.selected[i]];
        if (item.type !== 'file') {
          vm.removeFile(item.type, item.id, item.path);
        } else {
          vm.removeFile(item.type, item.id);
        }
      }
    },
    favFile(key) {
      db.collection('BUCKET').doc(this.drive[key].id).update({
        isImportant: !this.drive[key].isImportant,
      });
      this.openMenu();
    },
    selectAll() {
      for (let i = 0; i < this.drive.length; i += 1) {
        this.selected.push(i);
      }
    },
    alert() {
      alert('此功能尚未完成哦！');
    },
    downloadItem(url) {
      window.open(url, 'download');
    },
  },
  computed: {
    sortFile() {
      const newArr = this.sort.onlyImportant ?
        this.drive.filter(item => item.isImportant === true) :
        [...this.drive];
      switch (this.sort.sortBy) {
        case 'createTimeUp':
          return newArr.sort((a, b) => a.createTime - b.createTime);
        case 'createTimeDown':
          return newArr.sort((a, b) => b.createTime - a.createTime);
        case 'sizeUp':
          return newArr.sort((a, b) => a.size - b.size);
        case 'sizeDown':
          return newArr.sort((a, b) => b.size - a.size);
        case 'name':
          return newArr.sort((a, b) => a.name - b.name);
        case 'type':
          return newArr.sort((a, b) => a.type - b.type);
        // no default
      }
      return false;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped src="@/sass/index.scss" lang="scss">
</style>
