# 项目运行顺序指南

## 🚀 完整执行流程

### Step 0: 环境准备（首次运行）

```bash
# 1. 安装依赖
pip install -r requirements.txt

# 2. 配置环境变量（如果还没有）
cp env.example .env
# 编辑 .env 文件，确保 OPENAI_API_KEY 已设置
```

---

### Step 1: 数据预处理（如果数据未处理）

```bash
python src/data_collection/preprocess.py
```

**输出**: `data/processed/unified_daily.parquet`

**检查**:
```bash
ls -lh data/processed/unified_daily.parquet
```

---

### Step 2: 特征工程（如果特征未生成）

```bash
python src/feature_store/feature_engineering.py
```

**输出**: `data/features/daily_features.parquet`

**检查**:
```bash
ls -lh data/features/daily_features.parquet
```

---

### Step 3: 初始化数据库（首次运行）

```bash
python scripts/init_database.py
```

**输出**: `data/fitness.db` (SQLite数据库)

**检查**:
```bash
ls -lh data/fitness.db
```

---

### Step 4: 训练模型（如果模型未训练）

```bash
python src/recommendation/train.py
```

**输出**: `models/bandit_model.pkl`

**检查**:
```bash
ls -lh models/bandit_model.pkl
```

---

### Step 5: 验证配置（可选但推荐）

```bash
python scripts/setup_project.py
```

这会检查：
- ✅ 环境变量配置
- ✅ 数据库状态
- ✅ 数据文件存在性
- ✅ 模型文件存在性

---

### Step 6: 启动API服务器

```bash
python src/serving/api_server.py
```

**服务地址**: `http://localhost:8000`

**保持运行**，在另一个终端进行测试。

---

### Step 7: 测试API（在另一个终端）

#### 7.1 健康检查

```bash
curl http://localhost:8000/health
```

**预期输出**:
```json
{
  "status": "healthy",
  "recommender": "initialized",
  "learning_loop": "initialized",
  "test_recommendation": "working"
}
```

#### 7.2 获取推荐

```bash
curl -X POST "http://localhost:8000/recommend" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "user_001",
    "state": {
      "readiness_score": 75,
      "sleep_score": 80,
      "hrv": 50,
      "resting_hr": 60,
      "fatigue": 5,
      "activity_score": 70
    }
  }'
```

**预期输出**:
```json
{
  "action_id": 5,
  "workout_type": "STRENGTH",
  "intensity": "MEDIUM",
  "duration_minutes": 30,
  ...
}
```

#### 7.3 提交反馈

```bash
curl -X POST "http://localhost:8000/feedback" \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "user_001",
    "action_id": 5,
    "feedback": {
      "completed": true,
      "rpe": 7,
      "mood": 4,
      "satisfaction": 8
    }
  }'
```

---

## 📋 快速检查清单

### 首次运行前检查

- [ ] Python >= 3.8
- [ ] 依赖已安装 (`pip install -r requirements.txt`)
- [ ] `.env` 文件存在且配置了 `OPENAI_API_KEY`
- [ ] 在项目根目录 (`/Users/zhengdong/Documents/GitHub/RL`)

### 数据文件检查

```bash
# 检查数据文件是否存在
ls -lh data/processed/unified_daily.parquet
ls -lh data/features/daily_features.parquet

# 如果不存在，运行预处理和特征工程
python src/data_collection/preprocess.py
python src/feature_store/feature_engineering.py
```

### 模型检查

```bash
# 检查模型是否存在
ls -lh models/bandit_model.pkl

# 如果不存在，训练模型
python src/recommendation/train.py
```

---

## 🎯 最小运行流程（如果所有文件已存在）

如果数据、特征、模型都已准备好，只需：

```bash
# 1. 启动API服务器
python src/serving/api_server.py

# 2. 在另一个终端测试
curl http://localhost:8000/health
```

---

## 🔄 日常使用流程

### 每天使用

```bash
# 1. 启动API服务器
python src/serving/api_server.py

# 2. 使用API获取推荐和提交反馈
# （通过curl或前端应用）
```

### 有新数据时

```bash
# 1. 重新预处理
python src/data_collection/preprocess.py

# 2. 重新生成特征
python src/feature_store/feature_engineering.py

# 3. 重新训练模型
python src/recommendation/train.py

# 4. 重启API服务器
python src/serving/api_server.py
```

---

## ⚠️ 常见问题

### 问题1: 数据文件不存在

```bash
# 运行预处理
python src/data_collection/preprocess.py
```

### 问题2: 模型文件不存在

```bash
# 训练模型
python src/recommendation/train.py
```

### 问题3: API启动失败

```bash
# 检查端口是否被占用
lsof -i :8000

# 检查依赖是否安装
pip install fastapi uvicorn
```

### 问题4: OpenAI API错误

```bash
# 检查.env文件
cat .env | grep OPENAI_API_KEY

# 确保API key正确设置
```

---

## 📊 执行时间估算

- Step 0 (环境): ~2-5分钟
- Step 1 (预处理): ~2-5分钟
- Step 2 (特征): ~1-2分钟
- Step 3 (数据库): ~1秒
- Step 4 (训练): ~1-2分钟
- Step 5 (验证): ~5秒
- Step 6 (API): 持续运行

**首次完整运行**: ~10-15分钟
**日常运行**: 只需启动API服务器

---

## ✅ 验证成功标志

### 预处理成功
- ✅ `data/processed/unified_daily.parquet` 存在
- ✅ 文件大小 > 0

### 特征工程成功
- ✅ `data/features/daily_features.parquet` 存在
- ✅ 包含45个特征列

### 训练成功
- ✅ `models/bandit_model.pkl` 存在
- ✅ 控制台显示训练统计

### API成功
- ✅ 服务器启动无错误
- ✅ `/health` 返回 `{"status": "healthy"}`
- ✅ `/recommend` 返回推荐结果

---

## 🎯 推荐执行顺序

### 第一次运行（完整流程）

```bash
# 1. 环境准备
pip install -r requirements.txt
cp env.example .env  # 编辑添加API key

# 2. 数据预处理
python src/data_collection/preprocess.py

# 3. 特征工程
python src/feature_store/feature_engineering.py

# 4. 初始化数据库
python scripts/init_database.py

# 5. 训练模型
python src/recommendation/train.py

# 6. 验证配置
python scripts/setup_project.py

# 7. 启动API
python src/serving/api_server.py
```

### 日常使用（快速启动）

```bash
# 直接启动API服务器
python src/serving/api_server.py
```

