# TrendRadar 修改测试文档

## 修改内容总结

### 1. 每天准点推送一次且无重复内容
**位置**: `config/config.yaml`

**修改内容**:
```yaml
push_window:
  enabled: true  # 启用推送时间窗口控制
  time_range:
    start: "09:00"  # 推送时间窗口开始（北京时间）
    end: "11:00"    # 推送时间窗口结束（北京时间）- 已调整为2小时窗口确保准点推送
  once_per_day: true  # 每天在时间窗口内只推送一次
  push_record_retention_days: 7  # 推送记录保留天数
```

**说明**:
- ✅ 已启用推送时间窗口控制 (`enabled: true`)
- ✅ 设置推送窗口为 09:00-11:00（2小时窗口确保准点推送）
- ✅ 启用每天只推送一次 (`once_per_day: true`)
- ✅ 项目已内置推送记录管理，自动防止重复推送

### 2. 每个分类最多显示5条消息
**位置**: `main.py` 的 `count_word_frequency` 函数

**修改内容**:
```python
stats = []
max_items_per_category = 5  # 每个分类最多显示5条消息

for group_key, data in word_stats.items():
    all_titles = []
    for source_id, title_list in data["titles"].items():
        all_titles.extend(title_list)

    # 按权重排序
    sorted_titles = sorted(...)

    # 限制每个分类最多5条消息
    limited_titles = sorted_titles[:max_items_per_category]
    limited_count = len(limited_titles)
    original_count = len(sorted_titles)
    
    # 如果被截断，输出日志
    if original_count > max_items_per_category:
        print(f"分类 [{group_key}] 原有 {original_count} 条消息，限制为 {max_items_per_category} 条")
```

**说明**:
- ✅ 在消息统计时限制每个分类最多5条
- ✅ 保留原始数量信息便于查看
- ✅ 当分类消息被截断时会输出日志提示

## 测试步骤

### 1. 验证配置文件
```bash
cat config/config.yaml | grep -A 10 "push_window"
```
预期输出应包含:
- `enabled: true`
- `start: "09:00"`
- `end: "11:00"`
- `once_per_day: true`

### 2. 运行程序测试
```bash
python main.py
```

### 3. 检查输出日志
在程序运行日志中查找:
- ✅ 推送时间窗口信息：`推送窗口控制：当前时间 XX:XX 在/不在推送时间窗口...`
- ✅ 分类限制信息：`分类 [XXX] 原有 N 条消息，限制为 5 条`
- ✅ 推送记录信息：`今天首次推送` 或 `今天已推送过`

### 4. 验证推送内容
检查推送的消息（企业微信/飞书/钉钉等）：
- ✅ 每个分类最多显示5条消息
- ✅ 在同一天的时间窗口内，第二次运行时不会重复推送

### 5. 验证HTML输出
查看生成的HTML文件：
```bash
# 查看最新生成的HTML文件
ls -lt output/$(date +%Y年%m月%d日)/html/
```
- ✅ HTML文件中每个分类也应该只显示5条消息

## 注意事项

1. **推送时间窗口**
   - 如果在 GitHub Actions 上运行，由于执行时间不稳定，建议将时间窗口设置为至少2小时
   - 如果想要更精准的定时推送，建议使用 Docker 部署在个人服务器上
   - 可以根据实际需求调整 `start` 和 `end` 时间

2. **分类消息限制**
   - 当前限制为每个分类5条消息
   - 如需修改数量，在 `main.py` 中修改 `max_items_per_category = 5` 这行代码
   - 限制是按权重排序后取前5条，确保显示最重要的消息

3. **推送记录**
   - 推送记录保存在 `output/push_records/` 目录
   - 记录保留7天（可在配置文件中修改）
   - 如需手动清除推送记录以重新推送，删除对应日期的记录文件即可

## 回滚方法

如果需要回滚修改：

1. **恢复配置文件**
```yaml
push_window:
  enabled: false  # 关闭推送时间窗口控制
```

2. **恢复分类数量限制**
在 `main.py` 的第1311行：
```python
# 注释掉或删除这些限制代码
# max_items_per_category = 5
# limited_titles = sorted_titles[:max_items_per_category]
# 改为使用全部数据：
limited_titles = sorted_titles
```

## 测试结果记录

| 测试项 | 状态 | 说明 |
|--------|------|------|
| 配置文件修改 | ✅ 完成 | push_window 已正确配置 |
| 代码修改 | ✅ 完成 | 分类限制逻辑已实现 |
| 语法检查 | ✅ 通过 | 无 lint 错误 |
| 功能测试 | ⏳ 待测试 | 需要运行程序验证 |

## 修改文件清单

1. `/Users/marshal/Documents/git/TrendRadar/config/config.yaml` - 修改推送窗口配置
2. `/Users/marshal/Documents/git/TrendRadar/main.py` - 添加分类消息数量限制逻辑

