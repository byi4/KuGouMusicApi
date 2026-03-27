# 酷狗私人FM接口解密文档

## 1. Signature (签名)

### 算法
```
signature = MD5(salt + sorted_query_params + POST_body[0:256] + salt)
```

### 参数说明
| 参数 | 说明 |
|------|------|
| salt | `LnT6xpN3khm36zse0QzvmgTZ3waWdRSA` (固定值) |
| sorted_query_params | 按key字母排序的query参数，格式: `key1=value1key2=value2...` |
| POST_body[0:256] | POST请求体的前256字节 (UTF-8编码) |

### Python示例
```python
import hashlib

salt = "LnT6xpN3khm36zse0QzvmgTZ3waWdRSA"

# query参数(不含signature)
query_params = {
    "appid": "3116",
    "clienttime": "1774622579",
    "clientver": "11450",
    "dfid": "2BePAE01cmE63loG711xhKQn",
    "mid": "205503618508675370314862791722818401420",
    "uuid": "-"
}

# POST请求体
post_body = '{"remain_songcnt":5,"userid":2457452028,...}'

# 生成signature
sorted_params = ''.join([f'{k}={query_params[k]}' for k in sorted(query_params.keys())])
combo = salt + sorted_params + post_body[:256] + salt
signature = hashlib.md5(combo.encode('utf-8')).hexdigest()

print(signature)
# 输出: d0f0c44135bfd98a15fa009496b178dc
```

### 解密方式
通过分析Java源码 `com.kugou.common.network.w.java` 中的 `c()` 方法，发现签名生成逻辑：
1. 将query参数按key字母排序
2. 拼接为 `key=value` 格式
3. 与固定salt组合后计算MD5

---

## 2. Key (密钥参数)

### 算法
```
key = MD5(appid + salt + clientver + clienttime).toLowerCase()
```

### 参数说明
| 参数 | 说明 | 示例值 |
|------|------|--------|
| appid | 应用ID | `3116` |
| salt | 固定盐值 | `LnT6xpN3khm36zse0QzvmgTZ3waWdRSA` |
| clientver | 客户端版本 | `11450` |
| clienttime | 客户端时间戳 | `1774622578` |

### Python示例
```python
import hashlib

appid = "3116"
salt = "LnT6xpN3khm36zse0QzvmgTZ3waWdRSA"
clientver = "11450"
clienttime = "1774622578"

combo = appid + salt + clientver + clienttime
key = hashlib.md5(combo.encode()).hexdigest().lower()

print(key)
# 输出: 82d0720c4149736b8f75c2db5d15764a
```

### 解密方式
通过分析Java源码 `com.kugou.common.useraccount.utils.d.java` 中的 `a()` 方法：
```java
public static String a(long j, String str, int i, String str2) {
    return new ba().a(String.valueOf(j) + str + String.valueOf(i) + str2).toLowerCase();
}
```
参数对应关系：`j=appid, str=salt, i=clientver, str2=clienttime`

---

## 代码位置参考

| 参数 | Java源码位置 |
|------|-------------|
| signature | `com.kugou.common.network.w.java` |
| key | `com.kugou.common.useraccount.utils.d.java` |
| salt默认值 | `com.kugou.common.utils.br.java` (aE方法) |
