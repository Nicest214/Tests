# README

---

# 项目一：校园课程数据采集与查询系统

```markdown
# 校园课程数据采集与查询系统

用 Python 采集公开课程信息，清洗后入库 MySQL，并通过 Flask 提供查询接口。

## 技术栈

- Python 3.x（requests、BeautifulSoup、pandas）
- MySQL 8.x
- Flask

## 功能

- 采集：按院系分页抓取公开课程信息（约 1.2 万条）
- 清洗：去重、字段规整、空值处理
- 入库：3 张表（院系 / 课程 / 教师）
- 查询：4 个 REST 接口（列表、条件筛选、详情、统计）

## 怎么跑起来

```bash
pip install -r requirements.txt

# 1) 建库建表
mysql -u root -p < schema.sql

# 2) 采集（可指定页数与院系）
python crawl.py --pages 20

# 3) 启动接口
python app.py
# 默认 http://127.0.0.1:5000
```

## 接口示例

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `/api/courses?page=1&size=20` | 课程列表（分页） |
| GET | `/api/courses?dept=计算机&type=必修` | 条件筛选 |
| GET | `/api/courses/<id>` | 课程详情 |
| GET | `/api/stats/by-dept` | 各院系课程数统计 |

## 核心实现

- **采集容错**：请求带 UA 与限速（每页间隔 1s），失败重试 3 次，失败率从 8% 降到 0.5%
- **性能**：为 `dept`、`type`、`course_code` 建联合索引，列表查询从 1.8s 降到 0.2s
- **去重**：以「课程代码 + 开课学期」为唯一键，用 `INSERT ... ON DUPLICATE KEY UPDATE` 幂等写入


## 踩过的坑

- 翻页到后面返回空数组：对方限制了最大页码，改成按学期分批抓
- 中文写入 MySQL 乱码：建库时指定 `utf8mb4`，连接串加 `?charset=utf8mb4`
```

---

# 项目二：图书借阅管理系统

```markdown
# 图书借阅管理系统

课程设计项目（3 人小组），负责数据库设计与后端业务逻辑：借书、还书、续借、逾期自动标记。

## 技术栈

- MySQL 8.x（6 张表、触发器、存储过程）
- Python 3.x（业务逻辑与命令行界面）

## 数据库设计

6 张表：`reader`（读者）、`book`（图书）、`borrow`（借阅）、`overdue`（逾期）、`admin`（管理员）、`op_log`（操作日志）。

核心关系：

- 一个读者可有多条借阅记录（1:N）
- 一本书在借出状态下不可重复借出（通过 `borrow.status` 控制）
- 逾期由触发器在还书时自动判定并写入 `overdue`

## 功能

- 借书 / 还书 / 续借（最多续借 1 次）
- 逾期自动标记与罚金计算（每天 0.2 元，上限 20 元）
- 图书检索（按书名、作者、ISBN 模糊匹配）
- 操作日志记录

## 怎么跑起来

```bash
pip install -r requirements.txt
mysql -u root -p < schema.sql
python main.py
```

## 核心 SQL 示例

```sql
-- 逾期自动标记触发器
CREATE TRIGGER trg_borrow_overdue
AFTER UPDATE ON borrow
FOR EACH ROW
BEGIN
    IF NEW.return_date IS NOT NULL AND NEW.return_date > NEW.due_date THEN
        INSERT INTO overdue (borrow_id, days, fine)
        VALUES (NEW.id, DATEDIFF(NEW.return_date, NEW.due_date),
                LEAST(DATEDIFF(NEW.return_date, NEW.due_date) * 0.2, 20));
    END IF;
END;
```

---

## 写 README 的三条规矩

1. **第一段必须是一句话说明「这是什么」** —— 别写「本项目旨在……」，直接说人话。
2. **「怎么跑起来」必须能跑通** —— 面试官真会试。跑不通比不写更糟。
3. **截图比文字有说服力** —— 命令行输出、接口返回、页面效果，截一张贴进 `images/` 目录，README 里用 `![](images/demo.png)` 引用。
