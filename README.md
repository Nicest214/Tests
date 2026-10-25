# 练手底座：采集 → MySQL → Flask 接口

> ⚠️ **这是一份练手用的底座，不是让你直接挂到简历上的「作品」。**
> 用途：让你在没有代码的情况下，**两个晚上**跑通一条完整链路，然后改到「每一行都能讲清楚」。
> 详见文末《改造清单》—— 那 5 条做完，它才算你的。

## 它能干什么

1. 抓一个公开练习站点（<https://quotes.toscrape.com>，专门给人练爬虫用的）
2. 清洗后写进 MySQL（幂等：重复跑不会插两条）
3. 用 Flask 提供 4 个接口：列表分页 / 按标签筛 / 详情 / 按作者统计

## 环境准备

```bash
# 1) 装依赖
pip install -r requirements.txt

# 2) 建库建表（把 root 密码换成你自己的）
mysql -u root -p < schema.sql

# 3) 改两处配置：crawl.py 和 app.py 顶部的 DB 字典里的 password

# 4) 跑采集
python crawl.py

# 5) 起服务
python app.py
# 浏览器打开 http://127.0.0.1:5000/api/quotes?page=1&size=5
```

## 代码里值得注意的四个点（面试会问）

| 位置 | 做了什么 | 为什么 |
|---|---|---|
| `fetch()` | 失败重试 3 次 + 指数退避 | 网络抖动时不会整批崩掉 |
| `time.sleep(DELAY)` | 每页间隔 1 秒 | 礼貌爬取，避免被封 IP |
| `ON DUPLICATE KEY UPDATE` | 以 fingerprint 唯一键幂等写入 | 重复跑不会产生重复数据 |
| `schema.sql` 的三个索引 | `idx_author` / `idx_tags` / `idx_created` | 筛选和统计不用全表扫描 |

## 改造清单（做完这 5 条，才算你的项目）

1. **换数据源**：把 `BASE` 改成你真正想采的公开列表页（**先看它的 robots.txt**），改 `fetch()` 里的选择器
2. **加字段**：比如加上「发布时间」「点赞数」，同步改表结构和接口
3. **加一个你自己的接口**：例如「按关键词模糊搜索」或「导出 CSV」
4. **做一次性能对比**：删掉索引跑一次查询记下耗时，再加回索引跑一次，把两个数字写进 README
5. **写 README**：照桌面那份 `GitHub补录-README模板.md` 填，**必须包含「怎么跑起来」和一张截图**

## 上传前自查

- [ ] 代码里没有数据库密码（改成从环境变量读，或写个 `config.example.py`）
- [ ] 没有 `.venv/`、`__pycache__/`、`*.db`、`*.log`
- [ ] `pip install -r requirements.txt` 能装、`python crawl.py` 能跑、`python app.py` 能起
- [ ] README 里的命令能照着敲通
- [ ] **你能对着每一行说出「为什么这么写」**

## 最后一句

跑通它只要两小时，**改到能讲清楚要两天**。前者让你有东西可传，后者让你面试不翻车 —— 别只做前者。
