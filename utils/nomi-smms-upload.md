# nomi-smms-upload
> 基于 SM.MS 的图床脚手架

⚠️注意：如果没有SM.MS 账号，需要在[官网](https://smms.app)注册一个账号

## 安装CLI
```bash
npm i -g nomi-smms-upload
```

## 使用
```bash
# 上传
nomi-smms-upload [options]

# 帮助文档
nomi-smms-upload --help
```

## options 参数说明
#### config
> CLI 上传配置文件
> 
> 默认值：`nomi.smms.config.js`

使用方式：`-c`, `--config`
```bash
nomi-smms-upload -c nomi.smms.config.js
```

配置文件参数：

| 参数  | 描述            | 类型   | 默认值 |
| ----- | --------------- | ------ | ------ |
| token | SM.MS API Token | string | -      |

配置文件示例：
```javascript
module.exports = {
    token: 'xxxx'
}
```
通过[SM.MS](https://smms.app/home/apitoken)生成 API Token
![smms api token](https://s2.loli.net/2023/01/05/dWUSM8mf5FpCR3P.png)

#### token
> SM.MS API Token

使用方式：`-t`, `--token`
```bash
nomi-smms-upload --token xxx
```

#### file
> 待上传本地文件路径或目录路径（相对路径）

使用方式：`-f`, `--file`
```bash
nomi-smms-upload --file path/to/example.jpg
```

#### debug
> 是否开启调试模式

使用方式：`-d`, `--debug`
```bash
nomi-smms-upload --debug
```

## 本地生产文件
![nomi.smms.uploaded.json](https://s2.loli.net/2023/01/05/SRniUDgkQPjoF8c.png)

## SM.MS 图片列表
![pic list](https://s2.loli.net/2023/01/05/s2HmJOlW7PkuFic.png)