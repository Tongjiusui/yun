# Age of History II: Definitive Edition —— 全面解构报告
**分析日期**：2026-10-05
**目标**：`D:\steam\steamapps\common\Age of History 2 Definitive Edition`
**工作副本**：`E:\解构\_backup`（67,515 文件 / 692.03 MB，与原目录逐字节一致）
**原始目录**：全程只读，零写入

---

## 一句话结论

这个游戏的数据层**不是**「文本 + JSON」，而是 **Java 原生序列化为主**（26,407 个文件）。全部数据类位于旧命名空间 `age.of.civilizations2.jakowski.lukasz`，**无自定义 `readObject/writeObject`、`serialVersionUID` 恒为 0**——因此格式是**确定性的、可完全解析的，也是可回写的**。

---

## 完成度总览

| 阶段 | 内容 | 状态 | 报告 |
|---|---|---|---|
| 一 | 侦察与分类 | 完成 | [PHASE1](00_recon/PHASE1_报告_侦察与分类.md) |
| 二 | JAR 全量反编译与源码分析 | 完成（3,779 个 .java） | [PHASE2](20_decompiled/PHASE2_报告_反编译与源码分析.md) |
| 三 | 数据文件解析 | 完成（233/233 JSON） | [PHASE3](30_formats/PHASE3_报告_数据文件解析.md) |
| 四 | 存档文件分析 | 完成（133 文明 / 4,269 省） | [PHASE4](40_saves/PHASE4_报告_存档文件分析.md) |
| 五 | 地图几何与邻接 | 完成（4,269 省 / 12,449 边） | [PHASE5](50_map/PHASE5_报告_地图几何与邻接.md) |
| 六 | 图形与音频资源 | 完成（11,102 PNG / 103 音频） | [PHASE6](60_assets/PHASE6_报告_图形与音频资源.md) |
| 七 | 文档化与工具化 | 完成 | [格式规格](70_docs/01_文件格式规格总表.md) · [脚本手册](70_docs/02_脚本使用手册.md) · [未解问题](70_docs/03_未解问题与下一步.md) |

---

## 关键成果

### 1. 攻克 5 种物理编码

| 编码 | 魔数 | 文件数 | 解析器 |
|---|---|---:|---|
| Java 原生序列化 | `AC ED 00 05` | 26,407 | `scripts/javatree.py` |
| zlib + int16 网格 | `78 9C` | 22,268 | `scripts/scales.py` |
| libGDX JSON 方言 | `{` | 233 | `scripts/libgdxjson.py` |
| 纯文本索引 | ASCII | 7,360 | 标准读取 |
| PNG / Ogg | 标准魔数 | 11,253 | PIL / 自定义头解析 |

### 2. 五项原创发现

1. **Java 序列化字段排序规则**：JDK 会重排字段为「基本类型字母序，再对象引用字母序」，与源码声明顺序不同。**写档工具必须遵守**。
2. **`scales` 网格尺寸公式**：`w = (包围盒宽 − 1) × scale`，scale 7 命中 300/300。（减 1 这个 off-by-one 是实测得出的）
3. **邻接的正确模型**：`N(p) = lNeighboringProvinces ∪ lNeighboringSeaProvinces`，并集图 12,449 条边中仅 3 条单向。
4. **JSON 事件藏在 `eventsJ/` 且扩展名是 `.txt`**：入口 `Age_of_HISTORY.txt` 是分号列表，每个事件是 libGDX JSON。
5. **存档分片 ID 映射**：`_2Xn` → 文明 `1+3n`；`_4Xn` → 省份 `150n`（源码常量 `CIVS_PER_FILE=3`、`PROVINCES_PER_FILE=150`）。

### 3. 对既有假设的更正

| 流传说法 | 实测 |
|---|---|
| 主程序 `AoC2.jar` 约 17 MB | 实为 `game.jar` **26.19 MB** |
| provinces 是 X/Y 坐标**列表文件** | 实为 Java 序列化 `Province_GameData2` |
| civilizations 是 `tag + RGB` **文本** | 实为 Java 序列化，RGB 是 int 字段 |
| 事件已改为 JSON | **部分正确**：JSON 事件在 `eventsJ/` 且用 `.txt` 扩展名，存量剧本仍用二进制 |
| 领袖图在 `game/leadersIMG/` | **该目录为空（0 文件）**，DE 版不再分发领袖头像 |
| `iTrueOwnerOfProvince` | DE 版已改名为 `trueOwnerOfProvince` |

---

## 可视化成果

| 图 | 说明 |
|---|---|
| ![政治地图](50_map/map_political.png) | **存档真实政治地图**——3,963 省 / 116 国，一战格局 |
| ![地形图](50_map/map_land_only_terrain.png) | 陆地省份按地形着色 |
| ![邻接度](50_map/map_adjacency_degree.png) | 邻接度数热图 |
| ![大陆](50_map/map_continent.png) | 按 7 个大陆着色 |

---

## 目录结构

```
E:\解构\
├── README.md                    本文件（总索引）
├── _backup/                     游戏完整副本（692 MB，所有分析的输入）
├── 00_recon/                    阶段一产物 + PHASE1 报告
├── 20_decompiled/               阶段二产物：3,779 个反编译 .java + PHASE2 报告
├── 30_formats/                  阶段三产物：JSON schema、civilizations.csv + PHASE3 报告
├── 40_saves/                    阶段四产物：存档字段树、8 张表 + PHASE4 报告
├── 50_map/                      阶段五产物：几何、邻接、图、6 张渲染图 + PHASE5 报告
├── 60_assets/                   阶段六产物：资源清单 + PHASE6 报告
├── 70_docs/                     阶段七产物：格式规格总表、脚本手册、未解问题
├── scripts/                     17 个可复用 Python 脚本
└── 90_tools/                    cfr.jar + procyon.jar
```

---

## 快速上手

```python
import sys; sys.path.insert(0, r"E:\解构\scripts")
import javatree as jt

# 读一个省份
prov = jt.load(r"E:\解构\_backup\map\Earth\data\provinces\0")
f = jt.fields_of(prov)
print("顶点数:", len(jt.as_list(f["lPointsX"])))
print("邻接  :", jt.as_list(f["lNeighboringProvinces"]))
print("地形  :", jt.scalar(jt.fields_of(f["provinceInfo"])["sTerrainTAG"]))

# 读一个 JSON
import libgdxjson as lg
print(lg.dumps(lg.load(r"E:\解构\_backup\map\Earth\config.json")))
```

详见 [脚本使用手册](70_docs/02_脚本使用手册.md)。

---

## 数字总览

| 指标 | 值 |
|---|---:|
| 安装目录文件 / 大小 | 67,515 / 692.03 MB |
| Java 序列化文件 | 26,407（51 个不同类） |
| 反编译产出 | 3,779 个 .java |
| JSON 文件（全部解析成功） | 233 / 233 |
| 文明定义 | 3,710 |
| 省份（Earth） | 4,269（349,657 个顶点） |
| 邻接边 | 12,449（3 条单向） |
| 地图画布 | 4753 × 2601 |
| 存档实体 | 133 文明 / 4,269 省 / 6,042 条人口 |
| PNG / 音频 | 11,102 / 103 |
| 本项目产物 | 4,016 文件 / 66.0 MB（不含备份） |
