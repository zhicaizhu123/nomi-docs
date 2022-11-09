# VS Code配置
> 前端项目统一使用VS Code 进行项目开发，便于统一开发规范

## 常用插件

| 名称                         | 描述                                   |
| ---------------------------- | -------------------------------------- |
| 【强制】Pettier              | 代码规范                               |
| 【强制】Stylelint            | style 规范                             |
| 【强制】Volar                | vue3 提示                              |
| 【推荐】Git Graph            | Git 可视化提交分支                     |
| 【推荐】GitLens              | Git 可视化提交分支                     |
| 【推荐】Iconify IntelliSense | Iconify 图标可视化展示                 |
| 【推荐】DotENV               | .env 文件高亮提示                      |
| 【推荐】TODO Highlight       | 注释类型高亮提示                       |
| 【推荐】Auto Rename Tag      | HTML 收尾标签同步编辑                  |
| 【推荐】npm Intellisense     | 引入npm 提示                           |
| 【推荐】JSON to TS           | 将JSON 转化为TS接口                    |
| 【推荐】Code Spell Checker   | 单词拼写错误校验                       |
| 【推荐】TSDoc Comment        | 快速将普通注释转化为TS可识别的智能提示 |
| 【推荐】Less IntelliSense    | less相关                               |
| 【推荐】Headwind             | Tailwind css语法提示                   |
| ...                          |                                        |

## .vscode配置

### settings.json

```json
{
  "[vue]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "editor.formatOnSave": true,
  "editor.wordWrap": "on",
  "eslint.validate": ["vue", "javascript", "html"],
  "eslint.lintTask.enable": true,
  "vetur.format.defaultFormatter.html": "js-beautify-html",
  "vetur.format.defaultFormatter.js": "none",
  "vetur.experimental.templateInterpolationService": true,
  "docthis.includeDescriptionTag": true,
  "docwriter.style": "JSDoc"
}
```

### extensions.json

```json
{
  "recommendations": [
    "octref.vetur",
    "dbaeumer.vscode-eslint",
    "stylelint.vscode-stylelint",
    "esbenp.prettier-vscode",
    "mrmlnc.vscode-less",
    "mikestead.dotenv"
  ]
}
```

### vue.code-spnippets

```json
{
  // 创建组件模板
	"vue3 component template": {
    "prefix": "vue3Less",
    "body": [
      "<template>",
      "  <div class=\"$1\">",
      "  </div>",
      "</template>\n",
      "<script lang=\"ts\">",
      "import { defineComponent, onMounted } from 'vue'\n",
      "export default defineComponent({",
      "  name: '$1',\n",
      "  components: {},\n",
      "  props: {},\n",
      "  setup() {",
      "    onMounted(() => {})",
      "    return {}",
      "  },",
      "})",
      "</script>\n",
      "<style lang=\"less\" scoped></style>\n",
    ]
  },
}
```

