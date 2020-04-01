<template>
  <div>
    <!-- 引用插件 -->
    <VueDataEditor
      @input="codeChange"
      :value="code"
      :disable="false"
      :width="'100%'"
      :height="'300px'"
    ></VueDataEditor>
    <hr />
    <el-button @click="update" type="primary">提交新代码块</el-button>
  </div>
</template>
<script>
import VueDataEditor from "@/page/vue_data_editor/vue_data_editor.vue";

export default {
  data() {
    return {
      // 双向绑定的值
      code:'<?xml version="1.0" encoding="UTF-8"?><project xmlns="http://maven.apache.org/POM/4.0.0"         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">    <build>        <plugins>            <plugin>                <groupId>org.mybatis.generator</groupId>                <artifactId>mybatis-generator-maven-plugin</artifactId>                <version>1.3.2</version>                <configuration>                    <configurationFile>src/main/resources/generatorConfig.xml</configurationFile>                    <verbose>true</verbose>                    <overwrite>true</overwrite>                </configuration>            </plugin>        </plugins>    </build></project>',
      disable: false,
    };
  },
  components: {
    VueDataEditor
  },
  methods: {
    // 子组件传递过来的最新值
    codeChange(event) {
      this.code = event;
    },
    // 打印
    update() {
      console.log(this.code);
    }
  }
};
</script>