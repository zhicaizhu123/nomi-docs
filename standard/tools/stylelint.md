# Stylelint

## 依赖安装

```bash
yarn add -D stylelint @flinks/stylelint-config
```

## 项目配置

> 根目录下创建`style.config.js`

```javascript
module.exports = {
  root: true,
  extends: ['@flinks/stylelint-config'],
};
```