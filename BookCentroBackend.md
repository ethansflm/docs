## 代码规范

- 规则： 据 eslint 和 prettier 规则进行强制约束和格式化，代码提交前，必须通过 lint 检测和 prettier 格式化。
- 约定： 无法进行基础语法规则检测部分，事先达成代码风格约定（可后续补充），保持代码风格统一，并以此为 Code Review 依据。

### Lint 和 Prettier 约束

- link 工具选择

  - @typescript-eslint/parser，原 typescript-eslint-parse, 用于解析 ts 代码

  - @typescript-eslint/eslint-plugin, 原 eslint-plugin-tyepscript, 提供 link 规则

    ```
    yarn add eslint -D
    yarn add @typescript-eslint/parser -D
    yarn add @typescript-eslint/eslint-plugin -D
    ```

- lint 规则参考

  - plugin:@typescript-eslint/recommended, 默认加载推荐配置, FYI: https://github.com/typescript-eslint/typescript-eslint/tree/main/packages/eslint-plugin/docs/rules

- prettier 工具选择

  - prettier, 默认加载所有推荐配置，

    ```
    yarn add prettier -D
    yarn add eslint-plugin-prettier -D
    yarn add eslint-config-prettier -D

    ```

- eslint 配置文件参考

  ```js
  module.exports = {
    parser: '@typescript-eslint/parser',
    parserOptions: {
      project: 'tsconfig.json',
      sourceType: 'module',
    },
    plugins: ['@typescript-eslint/eslint-plugin', 'unused-imports'],
    extends: [
      'plugin:@typescript-eslint/recommended',
      'prettier/@typescript-eslint',
      'plugin:prettier/recommended',
    ],
    root: true,
    env: {
      node: true,
      jest: true,
    },
    ignorePatterns: ['.eslintrc.js'],
    rules: {
      '@typescript-eslint/interface-name-prefix': 'off',
      '@typescript-eslint/explicit-function-return-type': 'off',
      '@typescript-eslint/explicit-module-boundary-types': 'off',
      '@typescript-eslint/no-explicit-any': 'off',
      '@typescript-eslint/no-var-requires': 'off',
      '@typescript-eslint/no-empty-function': 'off',
      'unused-imports/no-unused-imports-ts': 'error',
    },
  };
  ```

### 代码风格约定

- 文件命名规范

  - 所有文件均以驼峰法命名，如： clientUser.controller.ts
  - entity 文件以 <entityName>.entity.ts 结尾，对应一张具体的 mysql table, 不可乱用，以后配置可能根据 文件名中.entity 标识来做处理
  - dto 文件以 <dtoName>.dto.ts 结尾
  - 单元测试文件以 <fileName>.spec.ts 结尾
  - e2e 测试文件以 <fileName>.e2e-spec.ts 结尾
  - mongo schema 文件以 <schemaName>.schema.ts 结尾，对应一个 mongo collection
  - interface 文件， 以 <fileName>.interface.ts 结尾
  - enum 文件，以 <fileName>.enum.ts 结尾
  - config 文件，以 <fileName>.config.ts 结尾
  - controller 文件，以 <fileName>.controller.ts 结尾
  - service 文件，以 <fileName>.service.ts 结尾

  - 文件名尾缀部分已表明文件类型之后，文件名首部分，尽量不要重复表述，如： bookController.controller.ts, 其中 bookController 与 .controller.ts 尾缀表达重复

- 变量命名规范

  - interface 以 'I' 打头，如 ICreateProductRequest
  - enum 以 'E' 打头，如 EProductStatus
  - enum 的要根据实际场景分开，不能因为 enum val 相同而在多个场景下使用同一个，导致耦合
  - enum val 使用可读性强的字符串，避免使用 0，1，2 等枚举值
  - 配置性常量全大写，不同单词下划线链接，如： AWS_S3_BUCKET_NAME = 'test'
  - 运行时常量驼峰法书写，如：awsS3BucketName = 'test'
  - class 首字母大写，驼峰法表示

- 数据库表定义规范

  - 字符编码集需要一致，统一使用 CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci
  - 表名不要使用数据库关键字，如 order，与 order by 重复，导致 sql 查询时需要特殊处理
  - 表名、字段名小写，下划线链接
  - 包含公共部分，created_at, updated_at, deleted_at
  - 没有值的字段直接给 null， 不要给''(string), 或者{}(json), 或者 0 (number), 这些非 null 的默认值，在条件查询的时候，造成很多干扰。很多时候，ifnull 函数能完成的事情，因为这些默认值的存在，不得已只能通过 case when，很不友好。
  - 同 module 之间，表与表通过 id 作为外键，不同 module 之间，只能以其他 unordered unique id 关联

- 关于模块、可注入类，以及方法等定义区分

  - 尽量减少不必要的注入和依赖，有利于单元测试的书写和测试驱动开发(TDD), 及减少代码耦合。

  - 无依赖的工具类方法直接 export， 如
    ```ts
     export function getCurrentTimestamp = ()=>{}
    ```
  - 有依赖的 工具类 handle， 才需要新增一个 Injectable util class， 如
    ```ts
    class S3Util {
      constructor(
        @Inject('S3Client')
        private connection: S3Client,
      ) {}
    }
    ```
  - 无依赖的工具类 provider, 直接在 common/services 下添加 service，不新增 module
  - 有依赖的工具类，视逻辑复杂度，新增 module

- 关于 dto、entity、shema

  - dto、mysql entity 、mongo schema 的 class 尽量不要共用，比如下面这个文件中，ApiProperty 是定义 dto 文件用的，Prop 是定义 mongo schema 用的，不要混用。
  - 关于上面一条，也可选择 ApiProperty 和 Column、Prop 等公用，但 dto 中需要通过 PickType， PartialType 等工具，达到 class 上的区分

  ```ts
  // -.schema.ts
  @ApiProperty({
    description: '產品分類父級_id',
    type: String,
  })
  @Prop()
  parentId: Types.ObjectId;
  ```

- 关于 flyway 管理数据库版本

  - 不能变更老的.sql 文件,新的 DDL 操作需要新增 .sql 文件， 文件名小写 && 下划线拼接, 内容为具体的 DDL 语句
  - 变更字段累加小版本，如 v1.0.1 -> v1.0.2
  - 新增表累加中间版本, 如 v1.0.1 -> v1.1.0

- 关于环境变量的设置

  - 项目根目录下存在.env 和 .env.local 两个文件
  - .env.local 不提交至 github, 仅在本地开发时用，若是不存在，服务启动时将读取.env 的设定。
  - .env 设定为变量的全集，请保证新增的环境变量在 .env 中有体现，这很重要.
  - 单元测试会读取 .env 的设定，测试完会有清除数据的动作，请不要在 .env 中配置你认为存量数据有用的环境.

- 关于金额的存储与显示

  - 与金额相关的字段，后端默认以分为单位，由于场景不同，可能会传递 number 和 string 两种类型
    - 凡 number 类型，均默认以分为单位, 同时携带币种， 如 curreny = 'hkd' && amount=2003402, amount 不应该单独被传递
    - 凡 string 类型的，均是转换好之后的类型，如: 'HK$ 20,034.02'， 供显示用。

- 关于前后端日期交互

  - 在前后端请求参数交互过程中，只能使用两类日期，number 和 Date String（"2022-02-19T13:25:19.854Z"），这两者都能精确到具体时区，不要使用时区不明的时间表达格式。如 "2022-02-19 13:25:19", 当在不同时区使用产品时，会引发歧义。

- 安全相关

  - 使用 sql 时，要用参数查询，避免使用拼接 SQL 字符串，主要是为了防止被前端传的非法字符串非法注入
  - 如涉及短信、邮件、文件上传付费资源的 api 设计上，一定要做好权限控制，或使用必要的限制手段，不能无限制放开使用
