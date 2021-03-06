## 开发准备

CAS的Python SDK包含了用于访问和操作CAS的所有API

### 相关资源

[GitHub地址](https://github.com/tencentyun/cas_python_sdk) ，欢迎贡献代码以及反馈问题


### 环境依赖
Python 2.7


### 安装SDK
安装 SDK 的方式有两种：pip 安装和手动安装

**方式一：使用pip安装**

	pip install cassdk

**方式二：手动安装**

从[Github地址](https://github.com/tencentyun/cas_python_sdk)下载源码，通过setup手动安装:

    python setup.py install
### SDK配置

要使用归档存储的API服务，首先需要获取到：appid、secret_id, secret_key，[获取地址](https://console.cloud.tencent.com/capi)

```Python

	# 要使用CAS的API，首先需要初始化一个CAS的client对象，其中包含客户端访问CAS的必要信息，以及提供了http接口的低层次封装
    client = CASClient(host, appid, secret_id, secret_key) # host：host stands for the domain name, not the host IP or host name. For example: cas.ap-chengdu.myqcloud.com

    response = client.list_vaults()                       # 返回HttpResponse
    ...

    # 要使用高层次接口，则需要使用client进一步初始化CAS的API对象
    cas_api = CasAPI(client)
    cas_response = cas_api.list_vaults()                  # 返回CASResponse类型
    ...
```

### 卸载SDK

使用pip uninstall 删除package

## 低级接口描述

CASClient类中封装Http API的所有接口，包括文件库操作、档案操作和任务操作。
由于CASClient是对基本Http API的封装，因此所有接口的返回类型均为HttpResponse类型，可以方便地获取响应的Header,Status以及Body的内容。

**低级接口的所有方法签名同高级接口一致，返回类型同 API 接口一致，具体返回值可参考 API 文档**，[API 文档](/document/product/572/8742)

## 高级接口描述

CAS的高级接口主要是对低级接口的进一步封装，核心类为CasAPI，其中提供所有基本接口的同步阻塞调用以及异常抛出的特性。

同时，在CasAPI类的基础之上，根据操作对象的不同，设计了Vault、Archive、Job等类型，提供针对文件库操作、档案操作和任务操作接口的直观调用。 以下是高级接口的详细描述。

### CASAPI类

CasAPI类是对低级接口的高层次抽象，类中的方法签名与CASClient类完全一样，只是所有方法均为同步阻塞调用，同时具备异常抛出特性（异常的主要类型为：CASServerError以及CASClientError），方法的返回类型均为CASResponse，可以使用字典类操作直接取得返回体中的相应字段。（具体字段可参考Http API手册中对应方法的返回值部分）。

每个CasAPI对象都需要用正确的CASClient对象进行初始化。

### 文件库类

文件库类中封装了所有基本的文件库操作，包括创建文件库、删除文件库、上传档案到文件库、删除文件库中的指定档案等，同时也可以获得文件库对象的metadata信息，它们存储在文件库类对象的成员变量中。

#### 成员变量

| 成员变量                | 变量类型   | 变量描述                                     |
| ------------------- | ------ | ---------------------------------------- |
| creation_date       | string | 创建文件库的 UTC 日期， ISO 8601 日期格式， 例如，2017-03-20T17:03:43.221Z |
| last_inventory_date | string | 完成上次文件库清单盘点的 UTC 日期，ISO 8601 日期格式， 例如，2017-03-20T17:03:43.221Z |
| number_of_archives  | int    | 上次文件库清单盘点时，文件库中的档案数。尚未运行，返回0             |
| size                | int    | 截止到上次编制清单日期，文件库中的档案总大小，单位：B。尚未运行，返回0     |
| qcs                 | string | 文件库的资源名称                                 |
| name                | string | 创建时指定的文件库名称                              |

#### 构造方法

方法签名：Vault(cas_api, vault_props)
参数说明：

| 参数说明        | 类型          | 说明                                     | 必选   |
| ----------- | ----------- | -------------------------------------- | ---- |
| cas_api     | CasAPI      | 已正确初始化的CasAPI对象                        | 是    |
| vault_props | CASResponse | 已有Vault的属性信息，调用CasAPI类中的describe_vault | 是    |

#### 类方法

所有的类方法均可以使用 **Vault.方法签名** 来调用，每个方法均需要传入初始化好的CasAPI对象cas_api。

##### 创建文件库

方法签名：`create(cas_api, name)`

功能说明：创建一个名为name的文件库

参数说明：

| 参数名     | 类型     | 说明              | 必选   |
| ------- | ------ | --------------- | ---- |
| cas_api | CasAPI | 已正确初始化的CasAPI对象 | 是    |
| name    | string | 要创建的文件库名称       | 是    |

返回值：如果创建成功，则返回新创建的Vault对象，否则抛出异常（具体参见异常说明）。


##### 查询文件库
方法签名：`get_vault_by_name(cas_api, vault_name)`
功能说明：根据文件库的名称获取Vault对象
参数说明：

| 参数名        | 类型     | 说明        | 必选   |
| ---------- | ------ | --------- | ---- |
| cas_api    | CasAPI | 同上        | 是    |
| vault_name | string | 要获取的文件库名称 | 是    |

返回值：如果获取成功，则返回指定的Vault对象，否则抛出异常（具体参见异常说明）。

##### 删除文件库

方法签名：delete_vault_by_name(cas_api, vault_name)
功能说明：删除指定名称的文件库
参数说明：

| 参数名        | 类型     | 说明        | 必选   |
| ---------- | ------ | --------- | ---- |
| cas_api    | CasAPI | 同上        | 是    |
| vault_name | string | 要删除的文件库名称 | 是    |

返回值：如果删除成功，则返回CASResponse类型响应，status值为2XX，无返回内容。如果删除失败，则抛出异常（具体参见异常说明）。

##### 列出文件库

方法签名：list_all_vaults(cas_api)
功能说明：列出当前用户下的所有文件库
参数说明：

| 参数名     | 类型     | 说明   | 必选   |
| ------- | ------ | ---- | ---- |
| cas_api | CasAPI | 同上   | 是    |

返回值： 如果调用成功，则返回文件库列表,类型为list，元素为Vault对象。否则，抛出异常（具体参见异常说明）。


#### 成员方法


##### 取回档案

方法签名：get_archive(archive_id)
方法说明：获取指定id的Archive对象
参数说明：

| 参数名        | 类型     | 说明            | 必选   |
| ---------- | ------ | ------------- | ---- |
| archive_id | string | 要获取Archive的id | 是    |

返回值：指定id的Archive类型对象。
**注意**：该方法默认archive_id为有效，如果实际的archive_id无效，则基于返回的Archive对象进行的所有后续操作均将失败。

##### 取回部分档案

方法签名：retrieve_archive(archive_id, desc, byte_range, tier)
方法说明：检索Archive
参数说明：

| 参数名        | 类型     | 说明                                       | 必选   |
| ---------- | ------ | ---------------------------------------- | ---- |
| archive_id | string | 要检索的Archive对象的id                         | 是    |
| desc       | string | 检索任务的描述                                  | 否    |
| byte_range | string | Archive检索操作要检索的字节范围。其格式为“StartByteValue-EndByteValue”。如果未指定，则检索整个档案。 | 否    |
| tier       | string | Archive检索的检索类型。枚举值： Expedited ，Standard ，Bulk。默认值：Standard | 否    |

返回值：如果调用成功，则返回一个archive-retrieval类型的Job对象。否则，抛出异常（具体参见异常说明）

##### 获取档案清单

方法签名：initiate_retrieve_inventory(desc)
方法说明：检索Archive列表
参数说明：

| 参数名  | 类型     | 说明      | 必选   |
| ---- | ------ | ------- | ---- |
| desc | string | 检索任务的描述 | 否    |

返回值：如果调用成功，则返回一个inventory-retrieval类型的Job对象。否则，抛出异常（具体参见异常说明）

##### 上传档案

方法签名：upload_archive(file_path, desc)
方法说明：从本地文件系统中上传Archive
参数说明：

| 参数名  | 类型     | 说明                          | 必选   |
| ---- | ------ | --------------------------- | ---- |
| desc | string | Archive的描述，会在读取Archive的时候返回 | 否    |

##### 删除档案

方法签名：delete_archive(archive_id)
方法说明：删除指定的Archive
参数说明：

| 参数名        | 类型     | 说明             | 必选   |
| ---------- | ------ | -------------- | ---- |
| archive_id | string | 要删除的Archive的id | 是    |

返回值：如果删除成功，则返回CASResponse类型响应，status为2XX。否则，抛出异常（具体参见异常说明）

##### 获取分块上传

方法签名：list_all_multipart_uploads()
方法说明：列出当前Vault下面所有已经上传的数据段
参数说明：无

返回值：如果调用成功，则返回所有已经上传分段的列表

##### 获取分块列表

方法签名：get_multipart_uploader(upload_id)
方法说明：获取一个当前正在进行的分段上传对象
参数说明：

| 参数名       | 类型     | 说明                    | 必选   |
| --------- | ------ | --------------------- | ---- |
| upload_id | string | 当前正在进行的分段上传的Upload Id | 是    |

返回值：如果调用成功，则返回一个MultipartUpload对象。否则，抛出异常（具体参见异常说明）

##### 删除文件库

方法签名：delete()
方法说明：删除当前vault对象所标识的文件库
参数说明：无

返回值：如果调用成功，则返回CASResponse，status为2XX。否则，抛出异常（具体参见异常说明）


### 档案类

档案类封装了档案操作的基本调用接口，包括：档案检索和删除档案。

#### 成员变量

| 成员变量       | 变量类型   | 变量描述              |
| ---------- | ------ | ----------------- |
| vault      | Vault  | Archive所属的Vault对象 |
| archive_id | string | 档案的id             |

#### 构造方法

方法签名：Archive(vault, archive_id)
参数说明：

| 参数名        | 类型     | 说明                | 必选   |
| ---------- | ------ | ----------------- | ---- |
| vault      | Vault  | Archive所属的Vault对象 | 是    |
| archive_id | string | 档案的id             | 是    |

#### 成员方法

##### 获取文件库

方法签名：get_vault()
方法说明：获取当前Archive所属的Vault对象
参数说明：无

返回值：如果调用成功，则返回当前Archive所属的Vault对象。

##### 启动档案取回

方法签名：initiate_archive_retrieval(desc, byte_range, tier)
方法说明：启动一个档案检索任务
参数说明：

| 参数名        | 类型     | 说明                                       | 必选   |
| ---------- | ------ | ---------------------------------------- | ---- |
| desc       | string | Archive检索任务的描述                           | 否    |
| byte_range | string | 档案检索要检索的字节范围                             | 否    |
| tier       | string | Archive检索的检索类型。枚举值： Expedited ，Standard ，Bulk。默认值：Standard | 否    |

返回值：如果调用成功，则返回检索Job的对象。否则，抛出异常（具体参见异常说明）


### 任务类

#### 成员变量

| 成员变量            | 变量类型   | 变量描述                                     |
| --------------- | ------ | ---------------------------------------- |
| id              | string | 任务的id                                    |
| archive_id      | string | 在档案取回操作中请求的档案的 ID）                       |
| archive_size    | int    | 档案取回任务请求的档案的大小                           |
| archive_etag    | string | 档案取回操作的整个档案的SHA256树形哈希                   |
| description     | string | 任务的描述                                    |
| job_range       | string | 档案取回任务所取回的字节范围，格式为“StartByteValue-EndByteValue”。如果没有在档案取回中指定范围，则取回整个档案 |
| creation_date   | string | 任务启动时的通用协调时间 (UTC) 日期。ISO 8601 日期格式的字符串表示，例如，2013-03-20T17:03:43.221Z。 |
| completed       | bool   | 如果任务完成，则为True.否则为False                   |
| completion_date | string | 任务完成时的通用协调时间 (UTC) 日期。ISO 8601 日期格式的字符串表示，例如，2013-03-20T17:03:43.221Z。 |
| status_code     | string | 任务状态代码。枚举值： Succeeded、Failed 或 InProgress |
| status_message  | string | 任务状态消息                                   |
| tier            | string | Archive检索任务的类型。枚举值： Expedited ，Standard ，Bulk |
| inventory_size  | int    | 与检索Archive列表任务请求相关联的列表的大小，单位字节           |

#### 构造方法

方法签名：Job(vault,cas_response)
参数说明：

| 参数名          | 类型          | 说明                               | 必选   |
| ------------ | ----------- | -------------------------------- | ---- |
| vault        | Vault       |                                  | 是    |
| cas_response | CASResponse | 调用describe_job返回的CASResponse类型对象 | 是    |


#### 类方法

##### 修改文件分块的大小

方法签名：calc_part_size(size_total)
方法说明：计算size_total大小的内容进行分段以后，每个分段的大小
参数说明：

| 参数名        | 类型     | 说明        | 必选   |
| ---------- | ------ | --------- | ---- |
| size_total | string | 待分段内容的总大小 | 是    |

返回值：如果调用成功则返回每个分段的大小。否则，抛出异常（具体参见异常信息）。


#### 成员方法

##### 从缓存池获取部分文件

方法签名：download_by_range(byte_range, file_path, file_obj, chunk_size, block)
方法说明：下载指定字节范围的内容到文件
参数说明：

| 参数名        | 类型     | 说明                                       | 必选                           |
| ---------- | ------ | ---------------------------------------- | ---------------------------- |
| byte_range | string | 取回的字节范围                                  | 是                            |
| file_path  | string | 文件下载路径                                   | file_path和file_obj至少一个不为None |
| file_obj   | file   | 要下载到的文件对象                                | file_path和file_obj至少一个不为None |
| chunk_size | int    | 每次读写的块大小,默认1MB                           | 否                            |
| block      | bool   | 方法的调用模式，True为同步阻塞模式，False为异步调用模式（如果Job当前还未完成，则会抛出DownloadArchiveError异常） | 否                            |

返回值：无返回值。 调用失败会抛出异常（具体参见异常信息）

##### 从缓存池获取全部文件

方法签名：download_to_file(file_path, chunk_size, block)
方法说明：下载Job的输出到指定的文件路径
参数说明：

| 参数名        | 类型     | 说明                                       | 必选   |
| ---------- | ------ | ---------------------------------------- | ---- |
| file_path  | string | 文件下载的路径                                  | 是    |
| chunk_size | int    | 每次读写的块大小，默认为1MB                          | 否    |
| block      | bool   | 方法的调用模式，True为同步阻塞模式，False为异步调用模式（如果Job当前还未完成，则会抛出DownloadArchiveError异常），默认为同步阻塞调用。 | 否    |

返回值：无返回值。 调用失败会抛出异常（具体参见异常信息）

### 返回值类型

CASResponse是HTTPResponse的Dict类型，可以直接利用dict的运算符取得响应的字段，CasAPI的所有接口都返回CASResponse。 CASResponse也是初始化Vault、Job与MultipartUpload对象的参数类型（调用describe方法返回的CASResponse的响应）。

**Example**

```
cas_response = cas_api.create_vault("vault-name")
vault_props = cas_api.describe("vault-name")
vault = Vault(cas_api, vault_props)
```
