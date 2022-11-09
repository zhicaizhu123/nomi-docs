# CommitLint

## 依赖安装

```bash
yarn add -D @commitlint/{cli,config-conventional} husky lint-stage
```

## 项目配置

> 根目录下创建`commitlint.config.js`

```javascript
module.exports = {
    extends: ['@commitlint/config-conventional']
};
```

> `package.json` 配置

```json
{
  "lint-staged": {
    '*.{js,jsx,ts,tsx}': ['eslint --fix', 'prettier --write'],
    '{!(package)*.json,*.code-snippets,.!(browserslist)*rc}': ['prettier --write--parser json'],
    'package.json': ['prettier --write'],
    '*.vue': ['eslint --fix', 'prettier --write', 'stylelint --fix'],
    '*.{scss,less,styl,html}': ['stylelint --fix', 'prettier --write'],
    '*.md': ['prettier --write'],

  }
}
```

> 跟目录创建.husky.rc

```json
{
    "hooks": {
      "pre-commit": "lint-staged"
    }
}
```

