---
title: Android-安全性-安全风险-SQL 注入
date: 2025-12-23 15:21:24 +0800
categories:
  - Android
  - Security
  - Risks
  - OWASP
tags:
  - Android
  - Security
  - Risks
  - OWASP
description: SQL注入可以暴露敏感的用户或应用程序数据，绕过身份验证和授权限制，并使数据库容易受到损坏或删除。
math: true
---
**OWASP 类别：**[MASVS-CODE：代码质量](https://mas.owasp.org/MASVS/10-MASVS-CODE)

## 概览

SQL注入利用应用程序的漏洞，通过在SQL语句中插入代码来访问底层数据库，从而绕过应用程序有意暴露的接口。该攻击可能导致私人数据泄露、数据库内容损坏，甚至后端基础设施遭到破坏。

SQL 可能存在漏洞，可以通过在执行前将用户输入连接起来动态创建的查询进行注入攻击。SQL注入攻击主要针对Web、移动应用以及任何SQL数据库应用程序，通常位列 [OWASP 十大](https://owasp.org/www-community/attacks/SQL_Injection)漏洞榜单。攻击者曾在多起备受瞩目的安全漏洞事件中使用过这种技术。

在这个简单的例子中，用户在订单号框中输入的未转义字符会被插入到 SQL 字符串中，并解释为以下查询：

```sql
SELECT * FROM users WHERE email = 'example@example.com' AND order_number = '251542'' LIMIT 1
```

这样的代码会在 Web 控制台中生成数据库语法错误，这表明该应用程序可能存在 SQL 注入漏洞。
将订单号替换为 `'OR 1=1–` 意味着可以实现身份验证，因为数据库会将相应语句评估为 `True`（这是因为 1 始终等于 1）。

同样，此查询会返回表格中的所有行：

```sql
SELECT * FROM purchases WHERE email='admin@app.com' OR 1=1;
```

### Content provider

Content provider 提供结构化存储机制，可以将内容限制为仅供某个应用访问，也可以将内容导出以与其他应用共享。权限设置应基于最小权限原则；导出的 `ContentProvider` 可以具有单个指定的读取和写入权限。

值得注意的是，并非所有 SQL 注入都会导致漏洞利用。一些 Content provider 已经允许读者完全访问 SQLite 数据库；能够执行任意查询几乎没有任何优势。可能构成安全问题的模式包括：

- **多个 content provider 共享一个 SQLite 数据库文件。**
    - 在这种情况下，每个表格都可能适用于一个唯一的 content provider。如果在一个 content provider 中成功实现 SQL 注入，则会授予对所有其他表格的访问权限。
- **content provider 对同一数据库中的内容拥有多项权限。**
    - 在授予不同权限级别的访问权限的单个 content provider 中实现 SQL 注入可能会导致攻击者能够从本地绕过安全或隐私设置。

## 影响

SQL 注入可能会泄露敏感的用户或应用数据、破解身份验证和授权限制，并导致数据库容易遭到破坏或删除。泄露事件可能对个人数据泄露的用户造成危险且持久的影响。应用程序和服务提供商则面临失去知识产权或用户信任的风险。
## 缓解措施

### 可替换的参数

在选择子句中，使用 `?` 作为可替换的参数和独立的选择参数阵列时，可将用户输入直接绑定到查询，而不是将其解释为 SQL 语句的一部分。

[Kotlin](https://developer.android.com/privacy-and-security/risks/sql-injection?hl=zh-cn#kotlin)

```kotlin
// Constructs a selection clause with a replaceable parameter.
val selectionClause = "var = ?"

// Sets up an array of arguments.
val selectionArgs: Array<String> = arrayOf("")

// Adds values to the selection arguments array.
selectionArgs[0] = userInput
```
[Java](https://developer.android.com/privacy-and-security/risks/sql-injection?hl=zh-cn#java)
```java
// Constructs a selection clause with a replaceable parameter.
String selectionClause =  "var = ?";

// Sets up an array of arguments.
String[] selectionArgs = {""};

// Adds values to the selection arguments array.
selectionArgs[0] = userInput;
```

用户输入直接绑定到查询，而不会被视为 SQL，从而防止代码注入。

下面是一个更详尽的示例，展示了一款购物应用的查询如何通过可替换的参数来检索购买交易的详细信息：

[Kotlin](https://developer.android.com/privacy-and-security/risks/sql-injection?hl=zh-cn#kotlin)
```kotlin
fun validateOrderDetails(email: String, orderNumber: String): Boolean {
    val cursor = db.rawQuery(
        "select * from purchases where EMAIL = ? and ORDER_NUMBER = ?",
        arrayOf(email, orderNumber)
    )

    val bool = cursor?.moveToFirst() ?: false
    cursor?.close()

    return bool
}
```

[Java](https://developer.android.com/privacy-and-security/risks/sql-injection?hl=zh-cn#java)

```java
public boolean validateOrderDetails(String email, String orderNumber) {
    boolean bool = false;
    Cursor cursor = db.rawQuery(
      "select * from purchases where EMAIL = ? and ORDER_NUMBER = ?",
      new String[]{email, orderNumber});
    if (cursor != null) {
        if (cursor.moveToFirst()) {
            bool = true;
        }
        cursor.close();
    }
    return bool;
}
```

### 使用 PreparedStatement 对象

[`PreparedStatement`](https://developer.android.com/reference/java/sql/PreparedStatement?hl=zh-cn) 接口将 SQL 语句预编译为一个对象，然后系统可以高效地多次执行该对象。PreparedStatement 使用 `?` 作为参数的占位符，这会导致下列已编译的注入尝试无效：

```sql
WHERE id=295094 OR 1=1;
```

在这种情况下，语句`295094 OR 1=1` 会被解读为 ID 的值，因此可能不会返回任何结果。而原始查询则会将`OR 1=1` 语句解释为 `WHERE` 语句的另一部分。以下示例展示了一个参数化查询：

[Kotlin](https://developer.android.com/privacy-and-security/risks/sql-injection?hl=zh-cn#kotlin)

```kotlin
val pstmt: PreparedStatement = con.prepareStatement(
        "UPDATE EMPLOYEES SET ROLE = ? WHERE ID = ?").apply {
    setString(1, "Barista")
    setInt(2, 295094)
}
```
[Java](https://developer.android.com/privacy-and-security/risks/sql-injection?hl=zh-cn#java)

```java
PreparedStatement pstmt = con.prepareStatement(
                                "UPDATE EMPLOYEES SET ROLE = ? WHERE ID = ?");
pstmt.setString(1, "Barista")
pstmt.setInt(2, 295094)
```

### 使用查询方法

在这个详细示例中，`query()` 方法的 `selection` 和 `selectionArgs` 组合在一起构成了 `WHERE` 子句。由于参数是单独提供的，因此它们会在组合之前进行转义，从而防止 SQL 注入。

[Kotlin](https://developer.android.com/privacy-and-security/risks/sql-injection?hl=zh-cn#kotlin)
```kotlin
val db: SQLiteDatabase = dbHelper.getReadableDatabase()
// Defines a projection that specifies which columns from the database
// should be selected.
val projection = arrayOf(
    BaseColumns._ID,
    FeedEntry.COLUMN_NAME_TITLE,
    FeedEntry.COLUMN_NAME_SUBTITLE
)

// Filters results WHERE "title" = 'My Title'.
val selection: String = FeedEntry.COLUMN_NAME_TITLE.toString() + " = ?"
val selectionArgs = arrayOf("My Title")

// Specifies how to sort the results in the returned Cursor object.
val sortOrder: String = FeedEntry.COLUMN_NAME_SUBTITLE.toString() + " DESC"

val cursor = db.query(
    FeedEntry.TABLE_NAME,  // The table to query
    projection,            // The array of columns to return
                           //   (pass null to get all)
    selection,             // The columns for the WHERE clause
    selectionArgs,         // The values for the WHERE clause
    null,                  // Don't group the rows
    null,                  // Don't filter by row groups
    sortOrder              // The sort order
).use {
    // Perform operations on the query result here.
    it.moveToFirst()
}
```
[Java](https://developer.android.com/privacy-and-security/risks/sql-injection?hl=zh-cn#java)

```java
SQLiteDatabase db = dbHelper.getReadableDatabase();
// Defines a projection that specifies which columns from the database
// should be selected.
String[] projection = {
    BaseColumns._ID,
    FeedEntry.COLUMN_NAME_TITLE,
    FeedEntry.COLUMN_NAME_SUBTITLE
};

// Filters results WHERE "title" = 'My Title'.
String selection = FeedEntry.COLUMN_NAME_TITLE + " = ?";
String[] selectionArgs = { "My Title" };

// Specifies how to sort the results in the returned Cursor object.
String sortOrder =
    FeedEntry.COLUMN_NAME_SUBTITLE + " DESC";

Cursor cursor = db.query(
    FeedEntry.TABLE_NAME,   // The table to query
    projection,             // The array of columns to return (pass null to get all)
    selection,              // The columns for the WHERE clause
    selectionArgs,          // The values for the WHERE clause
    null,                   // don't group the rows
    null,                   // don't filter by row groups
    sortOrder               // The sort order
    );
```

### 使用配置正确的 SQLiteQueryBuilder

开发者可以使用 [`SQLiteQueryBuilder`](https://developer.android.com/reference/android/database/sqlite/SQLiteQueryBuilder?hl=zh-cn) 类进一步保护应用，该类有助于构建要发送到 `SQLiteDatabase` 对象的查询。推荐的配置包括：

- [`setStrict()`](https://developer.android.com/reference/android/database/sqlite/SQLiteQueryBuilder?hl=zh-cn#setStrict\(boolean\)) 查询验证模式。
- [`setStrictColumns()`](https://developer.android.com/reference/android/database/sqlite/SQLiteQueryBuilder?hl=zh-cn#setStrictColumns\(boolean\))，验证列是否在 setProjectionMap 的允许列表中。
- [`setStrictGrammar()`](https://developer.android.com/reference/android/database/sqlite/SQLiteQueryBuilder?hl=zh-cn#setStrictGrammar\(boolean\))，用于限制子查询。

### 使用 Room 库

`android.database.sqlite` 软件包提供了在 Android 上使用数据库所需的 API。不过，这种方法需要编写底层代码，并且缺乏对原始 SQL 查询的编译时验证。随着数据图表的变化，受影响的 SQL 查询需要进行手动更新，此过程既耗时又容易出错。

高阶解决方案是使用 [Room 持久性库](https://developer.android.com/training/basics/data-storage/room?hl=zh-cn)作为 SQLite 数据库的抽象层。Room 的功能包括：

- 数据库类，作为连接到应用程序持久化数据的主要访问点。
- 数据库表的数据实体。
- 数据访问对象 (DAO) 提供应用程序可以用来查询、更新、插入和删除数据的方法。

Room 的优势包括：

- 针对 SQL 查询的编译时验证。
- 减少容易出错的样板代码。
- 简化数据库迁移。

### 最佳实践

SQL 注入是一种威力强大的攻击，很难完全抵御，尤其是在大型复杂应用程序中。应采取额外的安全措施，以降低数据接口潜在缺陷的严重程度，包括：

- 使用可靠、单向的加盐哈希对密码进行加密：
    - 使用适用于商业应用的 256 位 AES。
    - 使用 224 位或 256 位公钥大小进行椭圆曲线加密。
- 限制权限。
- 精确设计数据格式，并验证数据是否符合预期的格式。
- 尽可能避免存储个人数据或敏感用户数据（例如，通过哈希而不是传输或存储数据来实现应用程序逻辑）。
- 尽量减少访问敏感数据的 API 和第三方应用程序。

## 资源

- [使用 SQLite 保存数据](https://developer.android.com/training/data-storage/sqlite?hl=zh-cn)
- [content provider 基础知识](https://developer.android.com/guide/topics/providers/content-provider-basics?hl=zh-cn)
- [使用 Room 将数据保存到本地数据库](https://developer.android.com/training/data-storage/room?hl=zh-cn)
- [安全提示](https://developer.android.com/training/articles/security-tips?hl=zh-cn)
- [Kotlin SQL 注入防范指南](https://www.stackhawk.com/blog/kotlin-sql-injection-guide-examples-and-prevention/)

