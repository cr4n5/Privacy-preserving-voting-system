# 基于同态加密的隐私保护投票系统

## 简介

本系统建立了一个基于Paillier同态加密技术的隐私保护投票平台，实现了对投票过程的隐私保护。系统分为数据需求方和三方计算两部分，数据需求方负责投票信息的设置、获取和投票结果的获取，三方计算负责接收加密投票信息并计算投票结果。系统使用Paillier同态加密技术对投票信息进行加密，保证了投票过程的隐私性，有效的防止了贿选和抵抗了强迫投票人的攻击。

## 框架图

![框架图](img/基于同态加密的隐私保护投票系统/image.png)

## 密码学安全逻辑

![密码学安全逻辑](img/基于同态加密的隐私保护投票系统/image1.png)

## Paillier同态加密

**生成密钥对**：

1. 选择两个大素数 p 和 q。
2. 计算 n = p * q 和 λ = lcm(p-1, q-1)。
3. 选择一个随机数 g，使得 g 的阶为 n。
4. 计算 μ = (L(g^λ mod n^2))^(-1) mod n，其中 L(u) = (u-1)/n。
5. 公钥为 (n, g)，私钥为 (λ, μ)。

**加密**：

1. 选择一个随机数 r。
2. 计算密文 c = g^m * r^n mod n^2，其中 m 是明文。

**解密**：

1. 计算 m = L(c^λ mod n^2) * μ mod n。

**同态性质**：`E(m1) * E(m2) = E(m1 + m2) mod n^2 加法同态性质`

1. 给定两个密文 c1 和 c2，对应的明文为 m1 和 m2。
2. 计算 c = c1 * c2 mod n^2，对应的明文为 m1 + m2。

## 后端接口文档

### 数据需求方 `端口号：5000`

1. **获取公钥**：

- URL: /get_pubkey
- method: GET
- response:

```json
{
    "status": "success",
    "message": "Public key fetched",
    "key": {
        "班干部":[123,123],
        "学习委员":[123,123]
    }
}
```

2. **删除投票会话（删除密钥对,删除投票对象信息,删除投票结果）**：

- URL: /restart
- method: POST
- request:

```json
{
    "vote_obj": "班干部",
}
```

- response:

```json
{
    "status": "success",
    "message": "Restart success"
}
```

Error:

```json
{
    "status": "error",
    "message": "No vote info"
}
```

3. **设置投票信息**：

- URL: /set_vote_info
- method: POST
- request:

```json
{
    "vote_obj": "班干部",
    "list": ["teacher1", "teacher2", "teacher3"]
}
```

- response:

```json
{
    "status": "success",
    "message": "Vote info saved"
}
```

4. **获取投票信息**：

- URL: /get_vote_info
- method: GET
- response:

```json
{
    "status": "success",
    "message": "Vote info fetched",
    "result":[
        {
            "vote_obj": "班干部",
            "list": ["teacher1", "teacher2", "teacher3"]
        },
        {
            "vote_obj": "学习委员",
            "list": ["student1", "student2", "student3"]
        }
    ]
}
```

Error:

```json
{
    "status": "error",
    "message": "No vote info"
} 
```

5. **获取投票结果**：

- URL: /get_vote_result
- method: GET
- response:

分数为总分
正常包含投票结果

```json
{
    "status": "success",
    "message": "Vote result fetched",
    "vote_obj":{
        "班干部":{
            "result": {
                "teacher1": 123,
                "teacher2": 123,
                "teacher3": 123
            },
            "vote_people_number":123
        },
        "学习委员":{
            "result": {
                "student1": 123,
                "student2": 123,
                "student3": 123
            },
            "vote_people_number":123
        },
    }
}
```

有投票会话未有人投票

```json
{
    "status": "success",
    "message": "Vote result fetched",
    "vote_obj":{
        "班干部":{
            "result": "No vote result", 
            "vote_people_number":0
        },
        "学习委员":{
            "result": {
                "student1": 123,
                "student2": 123,
                "student3": 123
            },
            "vote_people_number":123
        },
    }
}
```

6. **接收加密投票**：`三方计算接口`

- URL: /receive_vote
- method: POST
- request:

```json
{
    "vote_obj": "班干部",
    "vote_sum":{
        "teacher1":123,
        "teacher2":123,
        "teacher3":123
    },
    "user_sum":["userA","userB"]
}
```

- response:

```json
{
    "status": "success",
    "message": "Vote received"
}
```

### 三方计算 `端口号：5001`

1. **删除投票会话（删除加密投票信息）**：

- URL: /restart
- method: POST
- request:

```json
{
    "vote_obj": "班干部",
}
```

- response:

```json
{
    "status": "success",
    "message": "Restart success"
}
```

Error:

```json
{
    "status": "error",
    "message": "No vote info"
}
```

2. **提交加密投票**：

- URL: /submit_vote
- method: POST
- request:

```json
{
    "username": "userA",
    "vote_obj": "班干部",
    "vote": {
        "teacher1":123,
        "teacher2":123,
        "teacher3":123
    },
    "pubkey": [123, 123]
}
```

- response:

```json
{
    "status": "success",
    "message": "Vote submitted"
}
```
