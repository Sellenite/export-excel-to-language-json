<template>
  <div class="container">
    <div class="select">
      <h2>选择.xlsx文件</h2>
      <b-form-file
        v-model="file"
        ref="file-input"
        class="mb-2"
        @input="handle"
        accept=".xlsx"
      ></b-form-file>
      <b-button @click="clearFiles" class="mr-2">清空</b-button>
      <!-- <b-button variant="success">转换</b-button> -->
      <p class="mt-2">
        Selected file: <b>{{ file ? file.name : 'null' }}</b>
      </p>
    </div>
    <div class="view">
      <h2>预览</h2>
      <b v-if="items.length === 0">未选择文件</b>
      <b-table hover responsive :items="items" v-else></b-table>
    </div>
    <div class="view">
      <h2>校验空值（第二行必须不能为空）</h2>
      <b v-if="items.length === 0">未选择文件</b>
      <b-table hover responsive :items="errorSet" v-else>
        <!-- 这是什么神奇的用法 v-slot:cell(emptyKeys)="data" -->
        <template v-slot:cell(emptyKeys)="data">
          <div v-for="(item, index) in data.value" :key="index">{{ item }}</div>
        </template>
      </b-table>
    </div>
    <div class="save">
      <h2>生成JSON</h2>
      <h5>（如无反应，点击下载后记得在右上角允许多个文件下载）</h5>
      <b-button variant="success" @click="save">点击下载</b-button>
    </div>
  </div>
</template>

<script>
const LANGUAGE_KEY = 'LANGUAGE_KEY';

import {
  BFormFile,
  BButton,
  BTable
} from 'bootstrap-vue';
import XLSX from 'xlsx';
import fileSaver from 'file-saver';
import JSZip from 'jszip'

export default {
  name: '',

  mixins: [],

  components: {
    'b-form-file': BFormFile,
    'b-button': BButton,
    'b-table': BTable
  },

  props: {},

  data() {
    return {
      file: null,
      items: [],
      key: '',
      errorSet: {},
      dataArr: [],
      languageKeys: []
    };
  },

  computed: {},

  watch: {},

  created() {},

  mounted() {},

  destroyed() {},

  methods: {
    handle(file) {
      if (file == null) {
        return;
      }
      const fileReader = new FileReader();

      fileReader.onload = ev => {
        try {
          const data = ev.target.result;
          // 以二进制流方式读取得到整份excel表格对象
          const workbook = XLSX.read(data, {
            type: 'binary',
          });
          // 存储获取到的数据
          let persons = [];
          // 遍历每张表读取
          for (const sheet in workbook.Sheets) {
            // eslint-disable-next-line no-prototype-builtins
            if (workbook.Sheets.hasOwnProperty(sheet)) {
              persons = persons.concat(
                XLSX.utils.sheet_to_json(workbook.Sheets[sheet]),
              );
              break; // 如果只取第一张表，就取消注释这行
            }
          }
          this.items = persons;
          this.prepare();
        } catch (e) {
          alert(e.message);
          return;
        }
      };

      fileReader.readAsBinaryString(file);
    },

    clearFiles() {
      this.file = null;
      this.items = [];
    },

    // 处理文件
    trans() {
      const zip = new JSZip();
      this.dataArr.forEach((obj, index) => {
        let name = String(this.languageKeys[index]).toLowerCase();
        name = String(name).replace(/\s/g, '-');
        const blob = this.create(obj);
        zip.file(`lang-${name}.json`, blob);
      });
      zip.generateAsync({ type: 'blob' }).then((blob) => {
        saveAs(blob, 'language.zip');
      });
    },

    // 准备数据格式并校验空值
    prepare() {
      this.errorSet = [];
      const item = {...this.items[0]};
      delete item[LANGUAGE_KEY];
      const languageKeys = Object.keys(item);
      const arr = languageKeys.map((lang, langIndex) => {
        const obj = {};
        const emptyKeys = [];
        this.items.forEach((row, index) => {
          const key = row[LANGUAGE_KEY] && String(row[LANGUAGE_KEY]).trim();
          const value = row[lang] && String(row[lang]).trim();
          if (key && value) {
            obj[key] = value || 'illegal value';
          }
          if (!value) {
            emptyKeys.push(key);
          }
        });
        if (emptyKeys.length) {
          this.errorSet.push({ lang, emptyKeys });
        }
        return obj;
      });
      this.languageKeys = languageKeys
      this.dataArr = arr;
    },

    // 生成文件
    create(obj) {
      const data = JSON.stringify(obj, null, 2);
      const blob = new Blob([data], {
        type: 'text/plain;charset=utf-8',
      });
      return blob;
    },

    save() {
      try {
        if (this.items.length === 0) {
          throw new Error('请先选择.xlsx文件');
        }
        this.trans();
      } catch (error) {
        // eslint-disable-next-line no-alert
        alert(error.message);
      }
    }
  },
};
</script>

<style lang="less" scoped>
.container {
  > div {
    margin-bottom: 30px;
  }
  .setkey {
    .set {
      display: flex;
      justify-content: space-between;
      margin-bottom: 20px;
    }
  }
  .input-group {
    margin-bottom: 10px;
  }
}
</style>
