# Boost 站内搜索引擎

一个基于 C++11 的轻量级站内搜索引擎，为 Boost C++ 官方文档提供全文检索功能。

**项目地址**：[https://github.com/woodencola/Boost-the-site-search-engine](https://github.com/woodencola/Boost-the-site-search-engine)

## 项目背景

Boost 官网（https://www.boost.org）目前缺乏站内搜索功能，用户查阅文档时需要手动浏览目录结构，效率较低。本项目通过离线构建索引 + 在线检索的方式，为 Boost 官方文档提供便捷的全文搜索服务。

## 技术栈

| 模块 | 技术 |
|------|------|
| 开发语言 | C++11 |
| 分词库 | cppjieba |
| JSON 处理 | jsoncpp |
| HTTP 服务 | cpp-httplib |
| 前端 | HTML + CSS + JavaScript + jQuery + Ajax |

## 整体流程

```
用户输入关键词
    ↓
服务端接收请求
    ↓
cppjieba 分词
    ↓
倒排索引检索 → 按 weight 排序 → 取正排数据
    ↓
jsoncpp 组装 JSON 响应
    ↓
前端 Ajax 接收 → 动态渲染搜索结果
```

## 模块说明

### 1. Parser（数据清洗）

- 递归遍历 `data/input/` 目录下 8,141 份 HTML 文档
- 去标签提取纯净文本（跳过 `< >` 标签内容）
- 提取每份文档的 **标题**（`<title>` 标签）、**正文**（去标签后的文本）、**URL**（拼接生成）
- 按 `title\3content\3url` 格式写入 `data/raw_html/raw.txt`，每行一个文档

### 2. Index（索引构建）

**正排索引**：`vector<DocInfo>`，下标即文档 ID，存储 title / content / url。

**倒排索引**：`unordered_map<string, InvertedList>`，关键字 → 倒排拉链。

**分词与加权**：集成 cppjieba 对标题和内容分别分词，统计词频后计算权重：

```cpp
weight = 10 * title_cnt + content_cnt;
```

标题中的词权重更高，搜索结果按 weight 降序排列。

### 3. Searcher（搜索服务）

- 接收查询词 → cppjieba 分词
- 逐个关键词查倒排索引 → 合并文档 ID
- 同一文档被多个关键词命中时累加权重
- 按权重降序排序，取前若干条结果
- 生成摘要（在 content 中定位关键词前后文）
- 通过 jsoncpp 组装 JSON 返回

### 4. http_server（HTTP 服务）

基于 cpp-httplib 提供 RESTful 搜索接口：

```
GET /s?word={关键词}
```

- 请求：`/s?word=asio`
- 响应：`application/json` 格式的搜索结果列表

### 5. 前端（Web 交互）

- 搜索框 + 搜索按钮
- jQuery + Ajax 异步请求，**无需刷新页面**
- 动态渲染：标题（可点击跳转）、摘要（关键词高亮）、URL

## 快速部署

### 环境要求

- Linux（CentOS 7 及以上）
- g++ 7.3+（cpp-httplib 需要 C++11 以上支持）
- make

### 升级 gcc（CentOS 7 默认 gcc 4.8 不满足）

```bash
sudo yum install -y centos-release-scl scl-utils-build
sudo yum install -y devtoolset-7-gcc devtoolset-7-gcc-c++
scl enable devtoolset-7 bash
```

### 安装依赖

```bash
# Boost 开发库
sudo yum install -y boost-devel

# jsoncpp
sudo yum install -y jsoncpp-devel

# cppjieba（需手动拷贝字典）
git clone https://gitcode.net/mirrors/yanyiwu/cppjieba.git
cd cppjieba && cp -rf deps/limonp include/cppjieba/
```

### 编译运行

```bash
make
./http_server
```

服务启动后访问：`http://{服务器IP}:8081`

## 效果展示

访问首页，输入关键词（如 `asio`、`thread`、`smart_ptr`），搜索结果按相关性排序展示：

- **标题**：可点击跳转 Boost 官方文档对应页面
- **摘要**：包含关键词的上下文片段
- **来源**：文档的官网 URL

## 项目扩展方向

| 方向 | 说明 |
|------|------|
| 整站搜索 | 扩展至 Boost 全站文档 |
| 在线更新 | 信号触发 + 爬虫定时拉取增量数据 |
| 竞价排名 | 模拟搜索引擎商业模式 |
| 热词统计 | 字典树 + 优先级队列，智能推荐搜索词 |
| 用户系统 | 引入 MySQL 实现注册/登录/搜索历史 |

## 项目定位

本项目是**网络库主项目**的辅助实践，重点在于：

- 掌握**正排/倒排索引**原理与实现
- 熟悉 **cppjieba 分词**、**jsoncpp**、**cpp-httplib** 等三方库的使用
- 实践**前后端分离**的 Web 开发模式

---
