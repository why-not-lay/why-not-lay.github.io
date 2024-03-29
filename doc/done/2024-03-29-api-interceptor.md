# 接口拦截中心

> 创建时间: 2024-03-29  
> 更新时间: 2024-03-29

## 需求
* 对外实现接口请求的拦截，对内服务分发，能对数据进行加解密。

## 功能
* 接口的绑定和解绑
* 出入请求的数据加解密

## 总体设计与实现

### 数据流

![DFD](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/projects/api-interceptor/dfd.png)

### 执行流程

![Flow](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/projects/api-interceptor/flow.png)kk

### 数据库说明

![ER](https://storage-1301473886.cos.ap-guangzhou.myqcloud.com/img/projects/api-interceptor/ER.png)

|键|类型|说明|
|--|--|--|
| `api` | `VARCHAR(255)` | 接口地址; 主键 |
| `origin` | `VARCHAR(255)` | 主机源地址 |
| `create_time` | `BIGINT` | 接口创建时间; 时间戳 |
| `update_time` | `BIGINT` | 接口更新时间; 时间戳 |
| `flag` | `BIGINT` | 接口状态; |

**flag 字段说明**

|状态|位|说明|
|--|--|--|
| 正常 | 1 | 置位时代表当前接口存在 |
| 删除 | 2 | 置位时代表当前接口被删除 |
| 暂停 | 3 | 置位时代表当前接口暂停使用 |
| 错误 | 4 | 置位时代表当前接口有误 |

> 目前启用的只有正常位和暂停位

## 备注
### 加解密部分

目前加解密功能只适用于 post 请求

使用到的加解密算法：
* RSA-2048-PKCS1
* AES-256-cbc

对于客户端，RSA 公钥数据预先设定；用于 AES 加密的 token 则是通过 RSA 加解密方式进行交换获得，iv 值预先设定

## 改进

* 添加接口自检测功能
* 加解密适用于 get 请求
* 新增 UI 界面允许手动管理接口

