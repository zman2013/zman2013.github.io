---
title: project-efficient-configurations
date: 2021-03-19 15:30:56
tags:
---

# 配置语义化提交
## 安装
```shell
npm i -D @commitlint/cli @commitlint/config-conventional commitizen cz-conventional-changelog
npx commitizen init cz-conventional-changelog --save-dev --save-exact
```

## 配置
修改文件 .commitlintrc.json，内容如下：
```json
{
  "extends": [
    "@commitlint/config-conventional"
  ]
}
```

## 提交方式
```shell
git cz
```


# 语义化发布
```shell
npm i -D @semantic-release/git @semantic-release/npm semantic-release
```

## 配置
在根目录新建 .releaserc.json，内容如下：
```json
{
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/npm"
  ]
}
```


# lint
## 安装
```shell
npm install -D eslint eslint-config-prettier eslint-plugin-prettier @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

## 配置
修改文件 .eslintrc.js，内容如下：
```js
module.exports = {
  root: true,
  parser: '@typescript-eslint/parser', // Specifies the ESLint parser
  parserOptions: {
    ecmaVersion: 2020, // Allows for the parsing of modern ECMAScript features
    sourceType: 'module', // Allows for the use of imports
  },
  extends: [
    'plugin:@typescript-eslint/recommended', // Uses the recommended rules from the @typescript-eslint/eslint-plugin
  ],
  rules: {
    // Place to specify ESLint rules. Can be used to overwrite rules specified from the extended configs
    // "@typescript-eslint/explicit-function-return-type": "off",
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-explicit-any': 'off',
    '@typescript-eslint/no-var-requires': 'off',
    '@typescript-eslint/no-non-null-assertion': 'off',
    '@typescript-eslint/no-this-alias': 'off',
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
  },
}
```

# prettier
## 安装
```shell
npm install -D prettier eslint-config-prettier eslint-plugin-prettier
```

## 配置
修改文件 .prettierrc，内容如下：
```json
{
  "semi": false,
  "singleQuote": true
}
```

修改文件 .eslintrc.js，内容如下：
```js
module.exports = {
  root: true,
  parser: '@typescript-eslint/parser', // Specifies the ESLint parser
  parserOptions: {
    ecmaVersion: 2020, // Allows for the parsing of modern ECMAScript features
    sourceType: 'module', // Allows for the use of imports
  },
  extends: [
    'plugin:@typescript-eslint/recommended', // Uses the recommended rules from the @typescript-eslint/eslint-plugin
    'prettier/@typescript-eslint', // Uses eslint-config-prettier to disable ESLint rules from @typescript-eslint/eslint-plugin that would conflict with prettier
    'plugin:prettier/recommended', // Enables eslint-plugin-prettier and eslint-config-prettier. This will display prettier errors as ESLint errors. Make sure this is always the last configuration in the extends array.
  ],
  rules: {
    // Place to specify ESLint rules. Can be used to overwrite rules specified from the extended configs
    // "@typescript-eslint/explicit-function-return-type": "off",
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-explicit-any': 'off',
    '@typescript-eslint/no-var-requires': 'off',
    '@typescript-eslint/no-non-null-assertion': 'off',
    '@typescript-eslint/no-this-alias': 'off',
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
  },
}
```


# husky lint-staged
## 安装
```shell
npm i -D husky@4 lint-staged
```

## 配置
修改文件 .lintstagedrc.json，内容如下：
```json
{
  "{src,test,examples}/**/*.ts": [
    "prettier --write",
    "eslint --fix"
  ]
}
```

修改文件 .huskyrc.json，内容如下：
```json
{
  "hooks": {
    "pre-commit": "npm run test && lint-staged",
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS",
    "pre-push": "npm run lint && npm run test -- --coverage --no-cache && npm run build"
  }
}
```


# jest
## 安装
```shell
npm i -D @types/jest jest jest-junit ts-jest
```

## 配置
修改文件 jest.config.js，内容如下：
```js
module.exports = {
  transform: {
    '.(ts|tsx)': 'ts-jest',
  },
  testRegex: '(/__tests__/.*|\\.(test|spec))\\.(ts|tsx|js)$',
  moduleFileExtensions: ['ts', 'tsx', 'js'],
  coveragePathIgnorePatterns: ['/node_modules/', '/test/', '/tools/'],
  // jest-junit is used to generate unit test reports in junit format,
  // which GitLab can store in Pipeline history.
  reporters: ['default', 'jest-junit'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  collectCoverage: true,
}
```
