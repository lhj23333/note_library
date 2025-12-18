---
share_link: https://share.note.sx/puq144ba#pTU87VIzX894kVhhfvys4j+vKKPXL0IdSXioYDHl8iU
share_updated: 2025-04-01T15:49:47+08:00
---

### 1. 基础概念：

- `JSON`（`JavaScript Object Notation`）是一种轻量级的数据交换格式，采用文本格式**存储**和**传输数据**。被广泛应用于 `Web` 开发、`API` 接口、配置文件等领域

---
### 2.  `JSON` 核心语法规则：

1. **基础结构**：
   - **对象(`Object`)**：用 `{}` 包裹的键值对集合
   - **数组(`Array`)**：用 `[]` 包裹的值的有序列表

---
2. **数据类型支持**：

| 类型     | 示例                   | 规则说明                        |
| ------ | -------------------- | --------------------------- |
| 字符串    | `"name" : "Alice"`   | **必须双引号**，支持转义字符如 `\"`、`\n` |
| 数字     | `"age" : 30`         | **整型、浮点数**                  |
| 布尔值    | `"isStudent" : true` | 仅 `true` 或 `false`          |
| `null` | `"data" : null`      | 表示空值                        |
| 对象     | `"address" : {....}` | 嵌套结构，键名必须唯一                 |
| 数组     | `"scores" ：[90, 85]` | 可包含不同类型元素                   |

- 示例：

```json
{
  "name": "张三",						// 字符串
  "age": 25,							// 整型
  "isStudent": false,					// bool
  "hobbies": ["阅读", "编程", "摄影"],	// 数组
  "address": {							// 对象
    "city": "北京",
    "postalCode": "100000"
  },
  "scores": null						// null
}
```

---
3. **语法规则**：

- **引号规则**：
	- 所有键名 `key` 必须用双引号 `""` 包裹。
	- 字符串值必须使用双引号，单引号非法。

```json
{ "name" : "Alice" }
```

- **逗号规则**：
	- 最后一个元素末尾不能有逗号

- **特殊字符转义**：

```json
"path": "C:\\Users\\Alice",  // 转义反斜杠
"quote": "He said: \"Hello\"" // 转义双引号
```

---
### 3. 应用场景：

1. **配置文件**：

- `JSON` 广泛用于软件配置（例如 `vscode` 的 `.vscode` 目录下的 `xxxx.json` 文件）：

```json
{
  "editor.fontSize": 14,
  "files.autoSave": "afterDelay",						
  "terminal.integrated.shell.linux": "/bin/bash"
}
```


---

2. **API 请求与响应**：

- `API` 请求：
	-  `API` 请求通常使用 `HTTP` 协议，通过请求体 (`Body`) 传递 `JSON` 数据，常见的场景：创建、更新资源或提交查询等请求 `POST` 应用场景
---
- **`POST` 请求体示例**：
	- 创建 `API` 请求：
		- `HTTP` 方法：`POST`
		- 请求体：

```json
POST /api/orders

// 请求用户信息
{
  "userId": 12345,						// 用户 ID
  "items": [							// 请求目标
    {
      "productId": "P-1001",
      "quantity": 2,
      "price": 49.99
    },
    {
      "productId": "P-1002",
      "quantity": 1,
      "price": 199.99
    }
  ],
  "shippingAddress": {					// 用户请求发送地址信息
    "street": "123 Main St",
    "city": "New York",
    "zipCode": "10001"
  },
  "paymentMethod": "credit_card"
}
```

---
- `API` 响应:
	- 服务器处理请求后返回 `JSON` 格式的响应，通常包含状态码、数据以及元信息
---
- **请求响应示例**：
	- 请求创建成功响应
		- `HTTP` 状态码：`201 Created`
		- 响应体：

```json
{
  "status": "success",							
  "data": {
    "orderId": "ORD-20231001-123",
    "totalAmount": 299.97,
    "estimatedDelivery": "2023-10-05T18:00:00Z",
    "paymentStatus": "pending"
  },
  "links": {
    "paymentUrl": "/api/payments/ORD-20231001-123"
  }
}
```

---
- 错误处理:
	- `API` 错误是返回标准化 `JSON` 错误信息
		- 错误码：`400`

```json
{
  "status": "error",
  "code": 400,
  "message": "Invalid payment method",
  "details": {
    "field": "paymentMethod",
    "allowedValues": ["credit_card", "paypal"]
  }
}
```


---

3. **SQL 数据库**：

- 现代关系型数据库（如 `MySQL`）支持 `JSON` 数据类型，运行存储和查询半结构化数据
- `SQL` 数据库主要使用的为 `SQL` 语句，对数据库以及数据存储有兴趣的同学可以去了解一下
	- `JSON` 数据体在其中的表现为可以作为一个数据类型来进行存储

```sql
CREATE TABLE orders (
  order_id VARCHAR(50) PRIMARY KEY,
  user_id INT NOT NULL,
  items JSON NOT NULL,          				-- 存储商品列表（JSON 数组）
  shipping_address JSON NOT NULL, 				-- 配送地址（JSON 对象）
  payment_method VARCHAR(20),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

> [!Tip] 
> 如果只是想简单上手使用数据库的同学（就是你项目本身并没有太大、太复杂的数据查询以及处理需求），对于 `SQL` 语句，最好上手的方式就是去浏览阅读别人的 `SQL` 代码，懂得 `SQL` 语句大致的执行逻辑以及基础语法，剩下的交给 `GPT` 来编写就好。
