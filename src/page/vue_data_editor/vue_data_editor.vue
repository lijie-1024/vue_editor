<template>
  <div>
    <el-card class="box-card">
      <!-- 操作栏 -->
      <el-row slot="header" class="clearfix" v-if="toolbar == true">
        <el-col :span="5">
          <el-button type="primary" @click="toolsBarLeft">格式化</el-button>
          <el-button type="primary" @click="toolsBarRight">恢复</el-button>
        </el-col>
        <el-col :span="6">
          <el-select v-model="value_type">
            <el-option label="JSON" value="JSON"></el-option>
            <el-option label="TEXT" value="TEXT"></el-option>
            <el-option label="XML" value="XML"></el-option>
            <el-option label="HTML" value="HTML"></el-option>
          </el-select>
        </el-col>
        <el-col :span="2" style="float:right">
          <el-button type="primary" v-clipboard:copy="contentBackup" @click="copy_value">复制</el-button>
        </el-col>
      </el-row>
      <!-- 编辑器 -->
      <div ref="vue_editor" :style="{height:height,width:width}"></div>
    </el-card>
  </div>
</template>
<style>
.box-card {
  margin: 20px;
}
.btn-hover {
  padding-left: 6px;
  padding-right: 6px;
}

.btn-hover:hover {
  background: #e0e0e0 !important;
}
.ace-xcode .ace_gutter {
  border-right: none !important;
  background: #fafafa !important;
}
.ace_content_disable {
  background: #fafafa !important;
}
</style>
<script>
// 引入ace代码编辑器
import ace from "brace/index";
import "brace/ext/emmet";
import "brace/ext/language_tools";
import "brace/mode/html";
import "brace/mode/json";
import "brace/mode/text";
import "brace/mode/xml";
import "brace/mode/javascript";
import "brace/theme/xcode";
import "brace/theme/terminal";
import "brace/snippets/javascript";
// 代码格式化方法
import {
  string_to_json_wrap,
  json_wrap_to_string,
  string_to_xml_wrap,
  check_string_type,
  wrap_to_string,
  string_to_wrap
} from "./data_format_utils";

// 主要代码
export default {
  name: "vue_editor",
  /**
   * 参数介绍：
   * value：（必填）双向绑定的数据；
   * theme：（非必填）ace编辑器主题默认xcode，可根据官网自行更换；
   * height：（必填）高度；
   * width：（必填）宽度；
   * options：（非必填）ace编辑器的设置
   * toolbar: （非必填）操作栏；
   * disable：（必填）是否启用编辑功能；
   * type：（非必填）json/xml/html/text，也支持更多，自行引用
   *
   */
  props: {
    value: {
      required: true
    },
    theme: {
      type: String,
      default: "xcode",
      required: false
    },
    height: {
      required: true
    },
    width: {
      required: true
    },
    options: Object,
    toolbar: {
      required: false,
      default: true,
      type: Boolean
    },
    disable: {
      required: false,
      type: Boolean,
      default: false
    },
    type: {
      required: false,
      type: String
    }
  },
  data() {
    return {
      editor: null,
      contentBackup: "",
      value_type: null,
      internalChange: false
    };
  },
  watch: {
    theme(v) {
      this.editor.setTheme("ace/theme/" + v);
    },
    options(v) {
      this.editor.setOptions(v);
    },
    height() {
      this.$nextTick(function() {
        this.editor.resize();
      });
    },
    width() {
      this.$nextTick(function() {
        this.editor.resize();
      });
    },
    value(v) {
      if (this.editor && !this.internalChange) {
        v = v && v !== null ? v : "";
        typeof v === "object" && (v = JSON.stringify(v));
        this.contentBackup = string_to_wrap(v);
        this.value_type = this.type || check_string_type(this.contentBackup);
        this.editor.session.setValue(this.contentBackup);
      }
    },
    value_type(nv) {
      switch (nv) {
        case "JSON": {
          this.contentBackup = string_to_json_wrap(this.contentBackup);
          this.editor.getSession().setMode("ace/mode/" + nv.toLowerCase());
          this.editor.session.setValue(this.contentBackup);
          break;
        }
        case "TEXT": {
          this.contentBackup = json_wrap_to_string(this.contentBackup);
          this.editor.getSession().setMode("ace/mode/" + nv.toLowerCase());
          this.editor.session.setValue(this.contentBackup);
          break;
        }
        case "XML": {
          this.contentBackup = string_to_xml_wrap(this.contentBackup);
          this.editor.getSession().setMode("ace/mode/" + nv.toLowerCase());
          this.editor.session.setValue(this.contentBackup);
          break;
        }
        case "HTML": {
          this.contentBackup = string_to_xml_wrap(this.contentBackup);
          this.editor.getSession().setMode("ace/mode/" + nv.toLowerCase());
          this.editor.session.setValue(this.contentBackup);
          break;
        }
        // 新增类别
        case "javascript": {
          this.editor.getSession().setMode("ace/mode/" + nv.toLowerCase());
          this.editor.session.setValue(this.contentBackup);
          break;
        }
      }
    },
    disable(v) {
      if (this.editor) {
        this.editor.setReadOnly(v);
        v
          ? this.$refs.vue_editor.classList.add("ace_content_disable")
          : this.$refs.vue_editor.classList.remove("ace_content_disable");
      }
    }
  },
  methods: {
    // 单位校验
    px(n) {
      if (/^\d*$/.test(n)) {
        return n + "px";
      }
      return n;
    },
    // 格式化
    toolsBarLeft() {
      this.contentBackup = string_to_wrap(this.contentBackup, this.value_type);
      this.editor.session.setValue(this.contentBackup);
    },
    // 数据转字符串
    toolsBarRight() {
      this.contentBackup = wrap_to_string(this.contentBackup, this.value_type);
      this.editor.session.setValue(this.contentBackup);
    },
    copy_value() {
      this.$copyText(this.contentBackup).then(
        () => {
          this.$message.success("已经复制到粘贴板！");
        },
        () => {
          this.$message.error("复制失败！");
        }
      );
    },

    onChange() {
      let error = false;
      let v;
      try {
        v = this.editor.getValue();
        error = false;
      } catch (err) {
        error = true;
      }
      if (error) {
        this.$emit("error");
      } else {
        if (this.editor) {
          this.internalChange = true;
          this.contentBackup = v;
          this.$emit("input", v);
          this.$nextTick(() => {
            this.internalChange = false;
          });
        }
      }
    },
    // 编辑器
    initView() {
      this.contentBackup = this.value && this.value !== null ? this.value : "";
      this.value_type = check_string_type(this.value);
      let vm = this;
      let lang = this.lang || "text";
      let theme = this.theme && this.theme !== "xcode" ? this.theme : "xcode";
      let editor_div = this.$refs.vue_editor;
      let editor = (vm.editor = ace.edit(editor_div));

      this.$emit("init", editor);

      editor.$blockScrolling = Infinity;
      editor.setOption("enableEmmet", false);
      editor.getSession().setMode("ace/mode/" + lang);
      editor.setTheme("ace/theme/" + theme);
      editor.getSession().setUseWrapMode(true);
      editor.setShowPrintMargin(false);
      editor.setValue(this.contentBackup);

      editor.on("change", vm.onChange);
      if (vm.options) editor.setOptions(vm.options);

      if (vm.disable) {
        editor.setReadOnly(true);
      }
      ace.require("ace/ext/language_tools")
      // 启用提示
      editor.setOption({
        enableBasicAutocompletion: true,
        enableSnippets: true,
        enableLiveAutocompletion: true
      });
    }
  },

  beforeDestroy() {
    this.editor.destroy();
    this.editor.container.remove();
  },
  mounted() {
    this.initView();
    this.height = this.height ? this.px(this.height) : "100%";
    this.width = this.width ? this.px(this.width) : "100%";
  }
};
</script>