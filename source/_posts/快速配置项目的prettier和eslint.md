---
title: 快速配置项目的 prettier 和 eslint date: 2021-08-17 14:57:10 tags:

- 配置

---

# 快速配置项目的 prettier 和 eslint

- prettier：代码格式化
- eslint：代码检查

## `prettier` 配置

[@hhp-tools/prettier-config](https://tools.hhp1614.top/tools/libs/prettier-config.html)

### 安装

```sh
npm i -D prettier @hhp-tools/prettier-config
```

### 使用

在 `package.json` 中配置

```json
{
    "prettier": "@hhp-tools/prettier-config"
}
```

## `eslint` 配置

[@hhp-tools/eslint-config](https://tools.hhp1614.top/tools/libs/eslint-config.html)

### 安装

```sh
npm i -D eslint @hhp-tools/eslint-config
```

### 使用

在 `package.json` 中配置

```json
{
    "eslintConfig": {
        "extends": [
            "@hhp-tools/eslint-config"
        ]
    }
}
```

或者在 `.eslintrc.*` 中配置

```json
{
    "extends": "@hhp-tools/eslint-config"
}
```
