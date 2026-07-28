> 模板版本：v0.4.0

<p align="center">
  <h1 align="center"> <code>watermelondb</code> </h1>
</p>


本项目基于 [watermelondb@[v0.28.1-0](https://github.com/Nozbe/WatermelonDB)]开发。

该第三方库的仓库已迁移至 Gitcode，且支持直接从 npm 下载，新的包名为：`@react-native-ohos/watermelondb` 版本所属关系如下：
| 三方库名称    | 三方库版本    | 发布信息     | 支持RN版本    | Autolink     | 编译API版本     | 社区基线版本    | npm地址                |
| ------------ | ------------ | ------------------------------ | ------------- | ------------- |------------------------ | ------------- | ------------- |
| @react-native-ohos/watermelondb | ~0.29.0 | [@react-native-ohos/watermelondb Releases](https://gitcode.com/OpenHarmony-RN/rntpc_watermelondb/tags) | 0.82.*  | 是 | API12+ | v0.28.1-0 | [Npm Address](https://www.npmjs.com/package/@react-native-ohos/watermelondb) |
| @react-native-ohos/watermelondb | ~0.28.2 | [@react-native-ohos/watermelondb Releases](https://gitcode.com/OpenHarmony-RN/rntpc_watermelondb/tags) | 0.72.* / 0.77.*  | 否 | API12+ | v0.28.1-0 | [Npm Address](https://www.npmjs.com/package/@react-native-ohos/watermelondb) |


## 1. 安装与使用

进入到工程目录并输入以下命令：

#### npm

```bash
npm install @react-native-ohos/watermelondb
```

#### yarn

```bash
yarn add @react-native-ohos/watermelondb
```

快速使用：
babel.config.js文件
```js
module.exports = {
  presets: ["module:metro-react-native-babel-preset"],
  plugins: [
    ['@babel/plugin-proposal-decorators', { legacy: true }], ['@babel/plugin-proposal-class-properties', { loose: true }],
  ],
};
```

安装babel相关依赖:

```bash

npm install @babel/plugin-proposal-decorators --save-dev

npm install @babel/plugin-transform-class-properties --save-dev
```

下面的代码展示了这个库的基本使用场景：

> [!WARNING] 使用时 import 的库名不变。

### watermelondb example

```js
//  jsi: false
import React, { useState } from 'react';
import { View, Text, Button, StyleSheet, ScrollView } from 'react-native';
import {
  appSchema,
  tableSchema,
  tableName,
  columnName,
  validateColumnSchema,
} from '@nozbe/watermelondb/Schema';
import { Database } from '@nozbe/watermelondb';
import SQLiteAdapter from '@nozbe/watermelondb/adapters/sqlite';
import { schemaMigrations } from '@nozbe/watermelondb/Schema/migrations';

const userTable = tableName('users');

const userColumns = {
  name: columnName('name'),
  email: columnName('email'),
  age: columnName('age'),
};

const userTableSchema = tableSchema({
  name: userTable,
  columns: [
    { name: userColumns.name, type: 'string' },
    { name: userColumns.email, type: 'string', isIndexed: true },
    { name: userColumns.age, type: 'number', isOptional: true },
  ],
});

const userInfoTable = tableName('user_info');
const userInfoColumns = {
  userId: columnName('user_id'),
  address: columnName('address'),
  phone: columnName('phone'),
};
const userInfoTableSchema = tableSchema({
  name: userInfoTable,
  columns: [
    { name: userInfoColumns.userId, type: 'string', isIndexed: true },
    { name: userInfoColumns.address, type: 'string' },
    { name: userInfoColumns.phone, type: 'string', isOptional: true },
  ],
});

const orderTable = tableName('order');
const orderTableSchema = tableSchema({
  name: orderTable,
  columns: [
    { name: columnName('order_no'), type: 'string', isIndexed: true },
    { name: columnName('user_id'), type: 'string', isIndexed: true },
    { name: columnName('amount'), type: 'number' },
    { name: columnName('status'), type: 'string', defaultValue: 'pending' },
  ],
});

const productTable = tableName('product');
const productTableSchema = tableSchema({
  name: productTable,
  columns: [
    { name: columnName('product_name'), type: 'string' },
    { name: columnName('price'), type: 'number' },
    { name: columnName('stock'), type: 'number', defaultValue: 0 },
  ],
});

const tableSchemaMap = {
  tableName: userTableSchema,
  columnName: userInfoTableSchema,
  tableSchema: orderTableSchema,
  appSchema: productTableSchema,
};


const getAppDatabaseSchema = (tableSchema) => {
  return appSchema({
    version: 1,
    tables: [tableSchema],
  });
};

const validateUserColumns = () => {
  const results = [];
  userTableSchema.columnArray.forEach(column => {
    try {
      validateColumnSchema(column);
      results.push({ column: column.name, valid: true });
    } catch (error) {
      results.push({ column: column.name, valid: false, error: error.message });
    }
  });
  console.log(results);
  return results;
};

const initializeDatabase = async (tableType) => {
  const targetTableSchema = tableSchemaMap[tableType] || userTableSchema;
  const adapter = new SQLiteAdapter({
    dbName: `watermelon_${tableType}_db`,
    schema: getAppDatabaseSchema(targetTableSchema),
    jsi: false,
    migrations: schemaMigrations({ migrations: [] }),
  });

  await adapter.initializingPromise; 
  return new Database({ adapter, modelClasses: [] });
};

const SchemaExample = () => {
  const [validationResult, setValidationResult] = useState(null);
  const [createResult, setCreateResult] = useState(null);
  const [database, setDatabase] = useState({}); 


  const handleValidateSchema = () => {
    const results = validateUserColumns();
    setValidationResult(results);
    setCreateResult(null);
  };

  const handleCreateTables = async (tableType) => {
    try {
   
      if (database[tableType]) {
        const tableName = tableSchemaMap[tableType]?.name || userTable;
        setCreateResult({
          success: true,
          message: `数据库(${tableType})已存在`,
          tables: [tableName],
        });
        return;
      }

      const db = await initializeDatabase(tableType);
   
      setDatabase({ ...database, [tableType]: db });
      const tableName = tableSchemaMap[tableType]?.name || userTable;
      setCreateResult({
        success: true,
        message: `数据库表(${tableType})创建成功`,
        tables: [tableName],
      });
    } catch (error) {
      setCreateResult({
        success: false,
        message: `数据库表(${tableType})创建失败`,
        error: error.message,
      });
    }
  };


  const handleClear = () => {
    setValidationResult(null);
    setCreateResult(null);
    setDatabase({}); 
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>WatermelonDB 表创建示例</Text>

      <View style={styles.buttons}>
        <Button
          title="使用tableName创建用户表"
          onPress={() => handleCreateTables('tableName')} 
          color="#4CAF50"
        />
        <Button
          title="使用columnName创建用户表"
          onPress={() => handleCreateTables('columnName')} 
          color="#4CAF50"
        />
        <Button
          title="使用tableSchema创建用户表"
          onPress={() => handleCreateTables('tableSchema')} 
          color="#4CAF50"
        />
        <Button
          title="使用appSchema创建用户表"
          onPress={() => handleCreateTables('appSchema')} 
          color="#4CAF50"
        />
        <Button
          title="使用validateColumnSchema验证字段定义"
          onPress={handleValidateSchema}
          color="#2196F3"
        />
        <Button title="清空状态" onPress={handleClear} color="#f44336" />
      </View>

   
      {validationResult && (
        <View style={styles.result}>
          <Text style={styles.resultTitle}>字段验证结果</Text>
          {validationResult.every(r => r.valid) ? (
            <Text style={styles.success}>所有字段定义均有效</Text>
          ) : (
            <View>
              <Text style={styles.error}>发现无效字段定义</Text>
              <Text style={styles.error}>
                {' '}
                WatermelonDB 版本过旧，该版本未导出 validateColumnSchema
                函数（该函数是较新版本才对外暴露的
              </Text>
            </View>
          )}
          {validationResult.map((item, i) => (
            <View key={i} style={styles.validationItem}>
              <Text>
                字段 {item.column}:{' '}
                {item.valid ? '有效' : `无效 (${item.error})`}
              </Text>
            </View>
          ))}
        </View>
      )}


      {createResult && (
        <View style={styles.result}>
          <Text
            style={[
              styles.resultTitle,
              { color: createResult.success ? 'green' : 'red' },
            ]}>
            {createResult.success ? '操作成功' : '操作失败'}
          </Text>
          <Text>{createResult.message}</Text>
          {createResult.error && (
            <Text style={styles.error}>{createResult.error}</Text>
          )}
          {createResult.tables && (
            <View>
              <Text>已创建表:</Text>
              {createResult.tables.map((table, i) => (
                <Text key={i}>- {table}</Text>
              ))}
            </View>
          )}
        </View>
      )}
    </ScrollView>
  );
};


const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  buttons: {
    flexDirection: 'row',
    gap: 10,
    marginBottom: 20,
    flexWrap: 'wrap',
  },
  result: {
    padding: 15,
    backgroundColor: 'white',
    borderRadius: 8,
    elevation: 2,
    marginBottom: 20,
  },
  resultTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  validationItem: {
    marginVertical: 4,
    padding: 4,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  success: {
    color: 'green',
    marginBottom: 10,
  },
  error: {
    color: 'red',
  },
});

export default SchemaExample;
```

```js
//  jsi: ture
import React, { useState } from 'react';
import { View, Text, Button, StyleSheet, ScrollView } from 'react-native';
import {
  appSchema,
  tableSchema,
  tableName,
  columnName,
  validateColumnSchema,
} from '@nozbe/watermelondb/Schema';
import { Database } from '@nozbe/watermelondb';
import SQLiteAdapter from '@nozbe/watermelondb/adapters/sqlite';
import { schemaMigrations } from '@nozbe/watermelondb/Schema/migrations';


const userTable = tableName('users');


const userColumns = {
  name: columnName('name'),
  email: columnName('email'),
  age: columnName('age'),
};

const userTableSchema = tableSchema({
  name: userTable,
  columns: [
    { name: userColumns.name, type: 'string' },
    { name: userColumns.email, type: 'string', isIndexed: true },
    { name: userColumns.age, type: 'number', isOptional: true },
  ],
});


const userInfoTable = tableName('user_info');
const userInfoColumns = {
  userId: columnName('user_id'),
  address: columnName('address'),
  phone: columnName('phone'),
};
const userInfoTableSchema = tableSchema({
  name: userInfoTable,
  columns: [
    { name: userInfoColumns.userId, type: 'string', isIndexed: true },
    { name: userInfoColumns.address, type: 'string' },
    { name: userInfoColumns.phone, type: 'string', isOptional: true },
  ],
});


const orderTable = tableName('order');
const orderTableSchema = tableSchema({
  name: orderTable,
  columns: [
    { name: columnName('order_no'), type: 'string', isIndexed: true },
    { name: columnName('user_id'), type: 'string', isIndexed: true },
    { name: columnName('amount'), type: 'number' },
    { name: columnName('status'), type: 'string', defaultValue: 'pending' },
  ],
});


const productTable = tableName('product');
const productTableSchema = tableSchema({
  name: productTable,
  columns: [
    { name: columnName('product_name'), type: 'string' },
    { name: columnName('price'), type: 'number' },
    { name: columnName('stock'), type: 'number', defaultValue: 0 },
  ],
});


const tableSchemaMap = {
  tableName: userTableSchema,
  columnName: userInfoTableSchema,
  tableSchema: orderTableSchema,
  appSchema: productTableSchema,
};


const getAppDatabaseSchema = tableSchema => {
  return appSchema({
    version: 1,
    tables: [tableSchema],
  });
};


const validateUserColumns = () => {
  const results = [];
 
  userTableSchema.columnArray.forEach(column => {
    try {
      validateColumnSchema(column);
      results.push({ column: column.name, valid: true });
    } catch (error) {
      results.push({ column: column.name, valid: false, error: error.message });
    }
  });
  console.log(results);
  return results;
};


const initializeDatabase = tableType => {

  const targetTableSchema = tableSchemaMap[tableType] || userTableSchema;
  const adapter = new SQLiteAdapter({
    dbName: `watermelonT_${tableType}_db`, 
    schema: getAppDatabaseSchema(targetTableSchema),
    jsi: true,
    migrations: schemaMigrations({ migrations: [] }),
  });

  adapter.initializingPromise; 
  return new Database({ adapter, modelClasses: [] });
};

const SchemaExampleT = () => {
  const [validationResult, setValidationResult] = useState(null);
  const [createResult, setCreateResult] = useState(null);
  const [database, setDatabase] = useState({}); 


  const handleValidateSchema = () => {
    const results = validateUserColumns();
    setValidationResult(results);
    setCreateResult(null);
  };


  const handleCreateTables = tableType => {
    try {
      if (database[tableType]) {
        const tableName = tableSchemaMap[tableType]?.name || userTable;
        setCreateResult({
          success: true,
          message: `数据库(${tableType})已存在`,
          tables: [tableName],
        });
        return;
      }

      const db = initializeDatabase(tableType);
      setDatabase({ ...database, [tableType]: db });
      const tableName = tableSchemaMap[tableType]?.name || userTable;
      setCreateResult({
        success: true,
        message: `数据库表(${tableType})创建成功`,
        tables: [tableName],
      });
    } catch (error) {
      setCreateResult({
        success: false,
        message: `数据库表(${tableType})创建失败`,
        error: error.message,
      });
    }
  };

  const handleClear = () => {
    setValidationResult(null);
    setCreateResult(null);
    setDatabase({}); 
  };

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>WatermelonDB 表创建示例</Text>

      <View style={styles.buttons}>
        <Button
          title="使用tableName创建用户表"
          onPress={() => handleCreateTables('tableName')} 
          color="#4CAF50"
        />
        <Button
          title="使用columnName创建用户表"
          onPress={() => handleCreateTables('columnName')} 
          color="#4CAF50"
        />
        <Button
          title="使用tableSchema创建用户表"
          onPress={() => handleCreateTables('tableSchema')} 
          color="#4CAF50"
        />
        <Button
          title="使用appSchema创建用户表"
          onPress={() => handleCreateTables('appSchema')} 
          color="#4CAF50"
        />
        <Button
          title="使用validateColumnSchema验证字段定义"
          onPress={handleValidateSchema}
          color="#2196F3"
        />
        <Button title="清空状态" onPress={handleClear} color="#f44336" />
      </View>

      {validationResult && (
        <View style={styles.result}>
          <Text style={styles.resultTitle}>字段验证结果</Text>
          {validationResult.every(r => r.valid) ? (
            <Text style={styles.success}>所有字段定义均有效</Text>
          ) : (
            <View>
              <Text style={styles.error}>发现无效字段定义</Text>
              <Text style={styles.error}>
                {' '}
                WatermelonDB 版本过旧，该版本未导出 validateColumnSchema
                函数（该函数是较新版本才对外暴露的
              </Text>
            </View>
          )}
          {validationResult.map((item, i) => (
            <View key={i} style={styles.validationItem}>
              <Text>
                字段 {item.column}:{' '}
                {item.valid ? '有效' : `无效 (${item.error})`}
              </Text>
            </View>
          ))}
        </View>
      )}

      {createResult && (
        <View style={styles.result}>
          <Text
            style={[
              styles.resultTitle,
              { color: createResult.success ? 'green' : 'red' },
            ]}>
            {createResult.success ? '操作成功' : '操作失败'}
          </Text>
          <Text>{createResult.message}</Text>
          {createResult.error && (
            <Text style={styles.error}>{createResult.error}</Text>
          )}
          {createResult.tables && (
            <View>
              <Text>已创建表:</Text>
              {createResult.tables.map((table, i) => (
                <Text key={i}>- {table}</Text>
              ))}
            </View>
          )}
        </View>
      )}
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  title: {
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  buttons: {
    flexDirection: 'row',
    gap: 10,
    marginBottom: 20,
    flexWrap: 'wrap',
  },
  result: {
    padding: 15,
    backgroundColor: 'white',
    borderRadius: 8,
    elevation: 2,
    marginBottom: 20,
  },
  resultTitle: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  validationItem: {
    marginVertical: 4,
    padding: 4,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  success: {
    color: 'green',
    marginBottom: 10,
  },
  error: {
    color: 'red',
  },
});

export default SchemaExampleT;
```

## 2. Link
|                           | 是否支持autolink | RN框架版本 |
|---------------------------|-----------------|------------|
| ~0.29.0                    |  Yes             |  0.82     |
| ~0.28.2                    |  No              |  0.77 / 0.72     |

使用AutoLink的工程需要根据该文档配置，Autolink框架指导文档：https://gitcode.com/openharmony-sig/ohos_react_native/blob/master/docs/zh-cn/Autolinking.md

如您使用的版本支持 Autolink，并且工程已接入 Autolink，可跳过ManualLink配置。

<details>
  <summary>ManualLink: 此步骤为手动配置原生依赖项的指导</summary>

首先需要使用 DevEco Studio 打开项目里的 HarmonyOS 工程 `harmony`。

### 2.1.在工程根目录的 `oh-package.json` 添加 overrides字段

为了让工程依赖同一个版本的 RN SDK，需要在工程根目录的 `oh-package.json5` 添加 overrides 字段，指向工程需要使用的 RN SDK 版本。替换的版本既可以是一个具体的版本号，也可以是一个模糊版本，还可以是本地存在的 HAR 包或源码目录。

关于该字段的作用请阅读[官方说明](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-oh-package-json5-V5#zh-cn_topic_0000001792256137_overrides)：

```json
{
  "overrides": {
    "@rnoh/react-native-openharmony" : "./react_native_openharmony"
  }
}
```

### 2.2. 引入原生端代码

目前有两种方法：

1. 通过 har 包引入（在 IDE 完善相关功能后该方法会被遗弃，目前首选此方法）；
2. 直接链接源码。

方法一：通过 har 包引入（推荐）

> [!TIP] har 包位于三方库安装路径的 `harmony` 文件夹下。

打开 `entry/oh-package.json5`，添加以下依赖

```json
"dependencies": {
    "@rnoh/react-native-openharmony": "0.72.32",
    "@react-native-ohos/watermelondb": "file:../../node_modules/@react-native-ohos/watermelondb/harmony/watermelondb.har"
  }
```

点击右上角的 `sync` 按钮

或者在终端执行：

```bash
cd entry
ohpm install
```

方法二：直接链接源码

> [!TIP] 如需使用直接链接源码，请参考[直接链接源码说明](./link-source-code.md)

### 2.3. 配置 CMakeLists 和引入 WatermelonDBPackage

打开 `entry/src/main/cpp/CMakeLists.txt`，添加：

```diff
project(rnapp)
cmake_minimum_required(VERSION 3.4.1)
set(CMAKE_SKIP_BUILD_RPATH TRUE)
set(RNOH_APP_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(NODE_MODULES "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../node_modules")
set(OH_MODULE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules")
set(RNOH_CPP_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules/@rnoh/react-native-openharmony/src/main/cpp")
set(RNOH_GENERATED_DIR "${CMAKE_CURRENT_SOURCE_DIR}/generated")
set(LOG_VERBOSITY_LEVEL 1)
set(CMAKE_ASM_FLAGS "-Wno-error=unused-command-line-argument -Qunused-arguments")
set(CMAKE_CXX_FLAGS "-fstack-protector-strong -Wl,-z,relro,-z,now,-z,noexecstack -s -fPIE -pie")
set(OH_MODULES "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules")

set(WITH_HITRACE_SYSTRACE 1) # for other CMakeLists.txt files to use
add_compile_definitions(WITH_HITRACE_SYSTRACE)

add_subdirectory("${RNOH_CPP_DIR}" ./rn)

# RNOH_BEGIN: manual_package_linking_1
# RNOH_END: manual_package_linking_1

file(GLOB GENERATED_CPP_FILES "${CMAKE_CURRENT_SOURCE_DIR}/generated/*.cpp") # this line is needed by codegen v1

+ add_subdirectory("${OH_MODULES}/@react-native-ohos/watermelondb/src/main/cpp" ./watermelondb)



add_library(rnoh_app SHARED
    ${GENERATED_CPP_FILES}
    "./PackageProvider.cpp"
    "${RNOH_CPP_DIR}/RNOHAppNapiBridge.cpp"
)
target_link_libraries(rnoh_app PUBLIC rnoh)

# RNOH_BEGIN: manual_package_linking_2
+ target_link_libraries(rnoh_app PUBLIC rnoh_watermelondb)


# RNOH_END: manual_package_linking_2
```

打开 `entry/src/main/cpp/PackageProvider.cpp`，添加：

```diff
#include "RNOH/PackageProvider.h"
#include "generated/RNOHGeneratedPackage.h"
+ #include "WatermelondbPackage.h"

using namespace rnoh;

std::vector<std::shared_ptr<Package>> PackageProvider::getPackages(
    Package::Context ctx) {
  return {
      std::make_shared<RNOHGeneratedPackage>(ctx), // generated by codegen v1
+         std::make_shared<WatermelondbPackage>(ctx)
  };
}
```

### 2.4. 在 ArkTs 侧引入 watermelondb 组件

找到 `entry/src/main/ets/RNPackagesFactory.ets`，添加：

```diff
...
import type {RNPackageContext, RNPackage} from '@rnoh/react-native-openharmony/ts';
+ import { WatermelonDBPackage } from '@react-native-ohos/watermelondb';

export function createRNPackages(ctx: RNPackageContext): RNPackage[] {
return [
+    new WatermelonDBPackage(ctx)
  ];
 }
...
...
```
</details>


### 2.5.运行

点击右上角的 `sync` 按钮

或者在终端执行：

```bash
cd entry
ohpm install
```

然后编译、运行即可。


## 3.约束与限制

### 3.1.兼容性

本文档内容基于以下版本验证通过：

1. RNOH：0.72.90; SDK：HarmonyOS NEXT Developer DB3; IDE: DevEco Studio: 5.0.5.220; ROM：NEXT.0.0.105;
2. RNOH：0.77.18; SDK：HarmonyOS 6.0.0 Release; IDE: DevEco Studio 6.0.0.858; ROM：6.0.0.112;
3. RNOH: 0.82.7; SDK: HarmonyOS 6.0.2 Release SDK; IDE: DevEco Studio 6.0.2.642; ROM: 6.0.0.328;

### 3.2.权限要求

#### 在 entry 目录下添加申请以上权限的原因

打开 `entry/src/main/resources/base/element/string.json`，添加：

```diff
...
{
  "string": [
+    {
+      "name": "module_desc",
+      "value": "moudle description"
+    },
+    {
+      "name": "EntryAbility_desc",
+      "value": "description"
+    },
+    {
+      "name": "EntryAbility_label",
+      "value": "WatermelonDB"
+    },
  ]
}
```

## 4.属性

> [!TIP] "Platform"列表示该属性在原三方库上支持的平台。

> [!TIP] "HarmonyOS Support"列为 yes 表示 HarmonyOS 平台支持该属性；no 则表示不支持；partially 表示部分支持。使用方法跨平台一致，效果对标 iOS 或 Android 的效果。

**Collection 类**


| Name                      | Description                            | Type     | Required | Platform    | HarmonyOS Support |
| ------------------------- | -------------------------------------- | -------- | -------- | ----------- | ----------------- |
| db                        | 获取该集合所属的数据库实例引用         | function | no       | iOS/Android | yes               |
| find                      | 根据指定条件查询集合中匹配的记录       | function | yes      | iOS/Android | yes               |
| findAndObserve            | 查询记录并建立响应式观察以监听数据变更 | function | yes      | iOS/Android | yes               |
| create                    | 在集合中创建并保存新纪录               | function | yes      | iOS/Android | yes               |
| prepareCreate             | 预创建记录对象但不立即持久化到数据库   | function | yes      | iOS/Android | yes               |
| prepareCreateFromDirtyRaw | 从原始未经验证的数据预创建记录对象     | function | yes      | iOS/Android | yes               |
| disposableFromDirtyRaw    | 基于原始脏数据创建一次性使用的临时记录 | function | yes      | iOS/Android | yes               |
| table                     | 获取该集合对应的数据库表名称           | function | no       | iOS/Android | yes               |
| schema                    | 获取该集合对应的数据表结构定义         | function | no       | iOS/Android | yes               |
| experimentalSubscribe     | 订阅集合级别的数据变更通知             | function | yes      | iOS/Android | yes               |

**Database 类**


| Name                       | Description                              | Type     | Required | Platform    | HarmonyOS Support |
| -------------------------- | ---------------------------------------- | -------- | -------- | ----------- | ----------------- |
| get&lt;T extends Model&gt; | 根据模型类型和 ID 从数据库获取特定记录   | function | yes      | iOS/Android | yes               |
| localStorage               | 获取数据库的本地存储适配器用于键值对数据 | function | yes      | iOS/Android | yes               |
| batch                      | 创建批量操作以高效执行多个数据库写入     | function | yes      | iOS/Android | yes               |
| write&lt;T&gt;             | 在写入事务中执行数据库修改操作           | function | yes      | iOS/Android | yes               |
| read&lt;T&gt;              | 在只读事务中执行数据库查询操作           | function | yes      | iOS/Android | yes               |
| action&lt;T&gt;            | 执行需要读写权限的数据库动作             | function | yes      | iOS/Android | yes               |
| withChangesForTables       | 创建监听指定表数据变更的观察者           | function | yes      | iOS/Android | yes               |
| experimentalSubscribe      | 数据库注册表变更监听变化                 | function | yes      | iOS/Android | yes               |
| unsafeResetDatabase        | 完全重置数据库并清除所有数据             | function | no       | iOS/Android | yes               |
| \_ensureInWriter           | 确保当前在写入线程中执行操作             | function | yes      | iOS/Android | yes               |
| \_fataError                | 处理数据库致命错误并终止连接             | function | yes      | iOS/Android | yes               |

**Relation 类**


| Name          | Description                                   | Type     | Required | Platform    | HarmonyOS Support |
| ------------- | --------------------------------------------- | -------- | -------- | ----------- | ----------------- |
| get id        | 获取关联目标模型的唯一标识符                  | function | no       | iOS/Android | yes               |
| set id        | 设置关联目标模型的标识符值                    | function | yes      | iOS/Android | yes               |
| fetch         | 异步获取关联记录的完整数据承诺                | function | no       | iOS/Android | yes               |
| then&lt;U&gt; | 支持 Promise 链式调用，处理异步获取的关联数据 | function | yes      | iOS/Android | yes               |
| set           | 建立与另一个模型实例的关联关系                | function | yes      | iOS/Android | yes               |
| observe       | 监听关联模型数据的实时变化                    | function | no       | iOS/Android | yes               |

**Model 类**


| Name                                          | Description                            | Type     | Required | Platform    | HarmonyOS Support |
| --------------------------------------------- | -------------------------------------- | -------- | -------- | ----------- | ----------------- |
| associations                                  | 定义模型与其他数据表之间的关联关系映射 | function | no       | iOS/Android | yes               |
| prepareMarkAsDeleted                          | 标记为删除状态                         | function | no       | iOS/Android | yes               |
| \_getChanges                                  | 获取模型变更观察对象                   | function | no       | iOS/Android | yes               |
| get id                                        | 获取实例的唯一标识符                   | function | no       | iOS/Android | yes               |
| get syncStatus                                | 获取数据同步状态                       | function | no       | iOS/Android | yes               |
| update                                        | 更新模型并立即提交                     | function | no       | iOS/Android | yes               |
| prepareUpdate                                 | 准备更新                               | function | no       | iOS/Android | yes               |
| prepareDestroyPermanently                     | 准备永久销毁但不执行                   | function | no       | iOS/Android | yes               |
| markAsDeleted                                 | 删除模型并提交                         | function | no       | iOS/Android | yes               |
| destroyPermanently                            | 永久删除模型记录                       | function | no       | iOS/Android | yes               |
| experimentalMarkAsDeleted                     | 软删除功能                             | function | no       | iOS/Android | yes               |
| experimentalDestroyPermanently                | 永久删除功能                           | function | no       | iOS/Android | yes               |
| observe                                       | 监听模型数据变化                       | function | no       | iOS/Android | yes               |
| collection                                    | 获取所属数据集合                       | function | no       | iOS/Android | yes               |
| get collections                               | 获取所有集合映射                       | function | no       | iOS/Android | yes               |
| get database                                  | 获取数据库实例                         | function | no       | iOS/Android | yes               |
| get db                                        | 获取数据库实例                         | function | no       | iOS/Android | yes               |
| get asModel                                   | 返回自身实例                           | function | no       | iOS/Android | yes               |
| callWriter&lt;T&gt;                           | 在写入线程执行操作                     | function | yes      | iOS/Android | yes               |
| callReader&lt;T&gt;                           | 在读取线程执行查询                     | function | yes      | iOS/Android | yes               |
| subAction&lt;T&gt;                            | 在事务中执行子操作                     | function | yes      | iOS/Android | yes               |
| get table                                     | 获取对应数据表名                       | function | no       | iOS/Android | yes               |
| prepareCreate                                 | 准备创建模型                           | function | yes      | iOS/Android | yes               |
| \_prepareCreateFromDirtyRaw                   | 从脏数据创建模型                       | function | yes      | iOS/Android | yes               |
| \_disposableFromDirtyRaw                      | 创建一次性模型                         | function | yes      | iOS/Android | yes               |
| experimentalSubscribe                         | 实验性订阅模型变更                     | function | yes      | iOS/Android | yes               |
| \_notifyChanged                               | 内部通知模型变更                       | function | no       | iOS/Android | yes               |
| \_notifyDestroyed                             | 内部通知模型销毁                       | function | no       | iOS/Android | yes               |
| \_getRaw                                      | 内部获取原始字段值                     | function | yes      | iOS/Android | yes               |
| \_setRaw                                      | 内部设置字段值并标记变更               | function | yes      | iOS/Android | yes               |
| \_dangerouslySetRawWithoutMarkingColumnChange | 内部不安全设置字段值                   | function | yes      | iOS/Android | yes               |
| \_ensureCanSetRaw                             | 内部确保可设置原始值                   | function | no       | iOS/Android | yes               |
| \_ensureNotDisposable                         | 内部确保非一次性实例                   | function | yes      | iOS/Android | yes               |

**Query 类**


| Name                             | Description                   | Type     | Required | Platform    | HarmonyOS Support |
| -------------------------------- | ----------------------------- | -------- | -------- | ----------- | ----------------- |
| pipe&lt;T&gt;                    | 对查询结果进行链式转换操作    | function | no       | iOS/Android | yes               |
| fetch                            | 立即执行查询获取结果          | function | no       | iOS/Android | yes               |
| then&lt;U&gt;                    | 支持 Promise 链式调用查询结果 | function | no       | iOS/Android | yes               |
| observe                          | 监听查询结果的实时变化        | function | no       | iOS/Android | yes               |
| observeWithColumns               | 监听特定字段变化的查询结果    | function | yes      | iOS/Android | yes               |
| fetchCount                       | 获取查询结果的数量统计        | function | yes      | iOS/Android | yes               |
| get count                        | 获取查询结果总数              | function | no       | iOS/Android | yes               |
| observeCount                     | 监听查询结果数量的变化        | function | no       | iOS/Android | yes               |
| fetchIds                         | 获取查询结果的 ID 列表        | function | no       | iOS/Android | yes               |
| unsafeFetchRaw                   | 不安全地获取原始查询数据      | function | no       | iOS/Android | yes               |
| experimentalSubscribe            | 实验性查询订阅功能            | function | yes      | iOS/Android | yes               |
| experimentalSubscribeWithColumns | 实验性字段变化订阅            | function | yes      | iOS/Android | yes               |
| experimentalSubscribeToCount     | 实验性数量变化订阅            | function | yes      | iOS/Android | yes               |
| markAllAsDeleted                 | 标记所有查询结果为软删除      | function | no       | iOS/Android | yes               |
| destroyAllPermanently            | 永久删除所有查询结果          | function | no       | iOS/Android | yes               |
| get modelClass                   | 获取查询对应的模型类          | function | no       | iOS/Android | yes               |
| get table                        | 获取查询的主表名称            | function | no       | iOS/Android | yes               |
| get secondaryTables              | 获取查询涉及的关联表          | function | no       | iOS/Android | yes               |
| get allTables                    | 获取查询所有相关表            | function | no       | iOS/Android | yes               |
| get associations                 | 获取查询的关联关系            | function | no       | iOS/Android | yes               |
| serialize                        | 序列化查询为可传输格式        | function | no       | iOS/Android | yes               |

**Schema 类**


| Name                             | Description                    | Type     | Required | Platform    | HarmonyOS Support |
| -------------------------------- | ------------------------------ | -------- | -------- | ----------- | ----------------- |
| tableName&lt;T extends Model&gt; | 定义数据库表的名称             | function | yes      | iOS/Android | yes               |
| columnName                       | 定义数据库字段的名称           | function | yes      | iOS/Android | yes               |
| appSchema                        | 定义整个应用数据库的结构模式   | function | yes      | iOS/Android | yes               |
| validateColumnSchema            | 验证字段模式定义的合法性       | function | yes      | iOS/Android | yes               |
| tableSchema                      | 定义单个数据表的结构和字段配置 | function | yes      | iOS/Android | yes               |

## 5.遗留问题

## 6.其他
•get id中get为JavaScript基础语法，获取关联目标模型的唯一标识符，通过 .id 形式调用<br>
•set id中set为JavaScript基础语法，设置关联目标模型的标识符值，通过 .id = 'xxx' 形式调用<br>

## 7.开源协议

本项目基于 [MIT License (MIT)](https://github.com/Nozbe/WatermelonDB/blob/master/LICENSE) ，请自由地享受和参与开源。
