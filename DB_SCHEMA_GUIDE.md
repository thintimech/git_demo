# 数据库结构整理（来自 `createtable.sql`）

> 本文按“模块 → 表 → 字段含义”整理，便于你做精简和重构决策。
> 当前统计：**17 张表**。

---

## 一、总览（按业务模块）

### 1) 题库与学习模块
- `questions`：题目主表（题干、答案、选项、解析相关字段）
- `images`：图片资源表（URL、本地路径、尺寸等）
- `question_images`：题目与图片关系表（含图片角色）
- `student_question_stats`：学员做题统计（次数、对错）
- `student_wrong_questions`：学员错题本
- `student_favorite_questions`：学员收藏

### 2) 用户与认证模块
- `student`：学员信息
- `coach`：教练信息
- `admin`：管理员信息
- `auth_account`：统一认证账号（账号 + 角色 + 密码哈希）

### 3) 教学与预约模块
- `videos`：视频教学资源
- `coach_availability_slot`：教练可预约时段
- `trial_booking`：体验课预约记录
- `coach_notification`：教练预约提醒

### 4) 历史与沟通模块
- `exam_history`：考试历史
- `training_log`：训练学时记录
- `chat_log`：师生聊天记录

---

## 二、表结构详细说明

## 1. `questions`（题目主表）

**用途**：存储题目文本、答案、选项、解析与扩展属性。

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| id | VARCHAR(64) | PK | 题目唯一ID |
| question_text | TEXT | NOT NULL | 题干 |
| answer | VARCHAR(32) | NULL | 正确答案（如 A/B/C/D） |
| difficulty | TINYINT | NULL | 难度 |
| subject | TINYINT | NULL | 科目 |
| type | TINYINT | NULL | 题型编码 |
| answer_skill | TEXT | NULL | 解题要点（短） |
| answer_skill_explain | TEXT | NULL | 解题详细讲解（长） |
| chapter_id | VARCHAR(16) | NULL | 章节ID |
| difficulty_level | TINYINT | NULL | 难度等级 |
| easy_error_flag | TINYINT(1) | NULL | 易错标识 |
| error_prone_flag | TINYINT(1) | NULL | 高错标识 |
| error_rate | VARCHAR(16) | NULL | 错误率文本 |
| flag | INT | NULL | 预留标记 |
| new_rule_flag | TINYINT(1) | NULL | 新规标识 |
| remark | MEDIUMTEXT | NULL | 备注/补充说明（常见富文本） |
| score | INT | NULL | 分值 |
| secret_flag | TINYINT(1) | NULL | 保密标识 |
| selected_flag | TINYINT(1) | NULL | 精选标识 |
| style | TINYINT | NULL | 样式/分类标识 |
| type_desc | VARCHAR(255) | NULL | 题型描述 |
| version_no | INT | NULL | 版本号 |
| items_desc_array | JSON | NULL | **选项内容数组**（如 `["正确","错误"]`） |
| items_title_array | JSON | NULL | **选项标题数组**（如 `["A","B"]`） |

**索引**：
- `idx_questions_difficulty(difficulty)`
- `idx_questions_subject(subject)`
- `idx_questions_type(type)`

---

## 2. `images`（图片资源表）

**用途**：存储图片基础信息，可是外链或本地资源。

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT | 图片ID |
| url | TEXT | NOT NULL | 图片URL或原始路径字符串 |
| local_path | VARCHAR(512) | NULL | 本地文件路径 |
| image_type | VARCHAR(32) | NOT NULL | 图片类型（如 png/jpg） |
| width | INT | NULL | 宽度 |
| height | INT | NULL | 高度 |
| file_size | BIGINT | NULL | 文件大小（bytes） |
| download_status | VARCHAR(32) | DEFAULT 'success' | 下载/同步状态 |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 创建时间 |

**索引**：
- `idx_images_type(image_type)`
- `idx_images_status(download_status)`

---

## 3. `question_images`（题目图片关系表）

**用途**：将题目与图片多对多关联，并区分图片角色（题目图/解析图等）。

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT | 关系ID |
| question_id | VARCHAR(64) | FK, NOT NULL | 题目ID |
| image_id | BIGINT | FK, NOT NULL | 图片ID |
| image_role | VARCHAR(32) | NOT NULL | 图片角色（如 stem / cover） |

**约束与索引**：
- 唯一：`uk_qi_question_image_role(question_id, image_id, image_role)`
- 外键：
  - `fk_qi_question` → `questions(id)` ON DELETE CASCADE
  - `fk_qi_image` → `images(id)` ON DELETE CASCADE

---

## 4. `student`（学员表）

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| student_id | VARCHAR(20) | PK | 学员ID/学号 |
| student_name | VARCHAR(50) | NOT NULL | 姓名 |
| student_phone | VARCHAR(20) | NOT NULL | 手机号 |
| student_gender | CHAR(1) | NOT NULL | 性别（M/F） |
| enroll_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 报名时间 |
| student_status | TINYINT | DEFAULT 0 | 状态（0在读/1毕业/2退学） |
| id_card_hash | VARCHAR(100) | NOT NULL | 身份证哈希 |
| exam_count | INT | DEFAULT 0 | 考试次数 |
| exam_change_count | INT | DEFAULT 0 | 考场变更次数 |
| subject2_bookable_count | INT | DEFAULT 0 | 科二可约次数 |
| subject3_bookable_count | INT | DEFAULT 0 | 科三可约次数 |

---

## 5. `coach`（教练表）

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| coach_id | VARCHAR(20) | PK | 教练ID |
| coach_name | VARCHAR(50) | NOT NULL | 姓名 |
| coach_phone | VARCHAR(20) | NOT NULL | 手机号 |
| coach_license | VARCHAR(50) | NOT NULL | 教练证号 |
| coach_status | TINYINT | DEFAULT 0 | 状态（0在职/1离职） |
| teaching_years | INT | DEFAULT 0 | 教龄 |

---

## 6. `admin`（管理员表）

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| admin_id | VARCHAR(20) | PK | 管理员ID |
| admin_name | VARCHAR(50) | NOT NULL | 管理员姓名 |
| admin_phone | VARCHAR(20) | NULL | 手机号 |
| admin_status | TINYINT | DEFAULT 0 | 状态（0在职/1停用） |

并初始化管理员：`A001`。

---

## 7. `auth_account`（统一认证表）

**用途**：同一 `account_id` 在不同角色下可有不同认证记录。

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| account_id | VARCHAR(20) | NOT NULL | 账号ID |
| role | VARCHAR(16) | NOT NULL | 角色（STUDENT/COACH/ADMIN） |
| password_hash | VARCHAR(100) | NOT NULL | 密码哈希 |
| enabled | TINYINT(1) | DEFAULT 1 | 启用状态 |

**唯一约束**：`uk_auth_account_role(account_id, role)`。

---

## 8. `exam_history`（考试历史）

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| exam_id | VARCHAR(20) | PK | 考试记录ID |
| student_id | VARCHAR(20) | FK, NOT NULL | 学员 |
| coach_id | VARCHAR(20) | FK, NOT NULL | 教练 |
| exam_subject | ENUM | NOT NULL | 考试科目 |
| exam_date | DATETIME | NOT NULL | 考试日期 |
| exam_result | ENUM | NOT NULL | 结果 |
| score | DECIMAL(5,2) | NOT NULL | 分数 |
| exam_location | VARCHAR(50) | NOT NULL | 考试场地 |
| site_change_count | INT | DEFAULT 0 | 场地变更次数 |
| remark | VARCHAR(255) | NULL | 备注 |

---

## 9. `training_log`（培训学时记录）

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| training_id | VARCHAR(20) | PK | 培训记录ID |
| student_id | VARCHAR(20) | FK, NOT NULL | 学员 |
| coach_id | VARCHAR(20) | FK, NOT NULL | 教练 |
| subject | ENUM | NOT NULL | 培训科目 |
| duration | DECIMAL(5,2) | NOT NULL | 学时（小时） |
| start_time | DATETIME | NOT NULL | 开始时间 |
| end_time | DATETIME | NOT NULL | 结束时间 |
| status | ENUM | DEFAULT 'PENDING' | 审核状态 |
| gps_track | JSON | NULL | GPS轨迹 |

---

## 10. `chat_log`（聊天记录）

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| chat_id | VARCHAR(20) | PK | 消息ID |
| student_id | VARCHAR(20) | FK, NOT NULL | 学员ID |
| coach_id | VARCHAR(20) | FK, NULL | 教练ID（可空） |
| content | TEXT | NOT NULL | 消息内容 |
| create_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 发送时间 |
| chat_session_id | VARCHAR(20) | NOT NULL | 会话ID |
| is_read | TINYINT(1) | DEFAULT 0 | 是否已读 |

---

## 11. `student_question_stats`（学员题目统计）

**用途**：每个“学员-题目”一行，累计做题统计。

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| student_id | VARCHAR(20) | FK, NOT NULL | 学员ID |
| question_id | VARCHAR(64) | FK, NOT NULL | 题目ID |
| attempt_count | INT | DEFAULT 0 | 作答次数 |
| correct_count | INT | DEFAULT 0 | 答对次数 |
| wrong_count | INT | DEFAULT 0 | 答错次数 |
| last_result | TINYINT(1) | NULL | 最近结果 |
| last_answer_time | DATETIME | NULL | 最近作答时间 |

**唯一约束**：`uk_sqs_student_question(student_id, question_id)`。

---

## 12. `student_wrong_questions`（错题本）

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| student_id | VARCHAR(20) | FK, NOT NULL | 学员ID |
| question_id | VARCHAR(64) | FK, NOT NULL | 题目ID |
| wrong_count | INT | DEFAULT 1 | 进入错题本后累计错次数 |
| first_wrong_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 首次入错题本时间 |
| last_wrong_time | DATETIME | DEFAULT CURRENT_TIMESTAMP | 最近错题时间 |

**唯一约束**：`uk_swq_student_question(student_id, question_id)`。

---

## 13. `student_favorite_questions`（收藏表）

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT | 主键 |
| student_id | VARCHAR(20) | FK, NOT NULL | 学员ID |
| question_id | VARCHAR(64) | FK, NOT NULL | 题目ID |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | 收藏时间 |

**唯一约束**：`uk_sfq_student_question(student_id, question_id)`。

---

## 14. `videos`（视频资源）

| 字段 | 类型 | 约束/默认 | 含义 |
|---|---|---|---|
| id | BIGINT | PK, AUTO_INCREMENT | 视频ID |
| title | VARCHAR(255) | NOT NULL | 标题 |
| subject_category | VARCHAR(20) | NOT NULL | 科目分类 |
| video_url | VARCHAR(512) | NOT NULL | 视频地址 |
| cover_url | VARCHAR(512) | NULL | 封面地址 |
| duration_seconds | INT | NULL | 时长秒数 |
| resolution | VARCHAR(20) | NULL | 分辨率 |
| file_size | BIGINT | NULL | 文件大小 |
| file_format | VARCHAR(10) | DEFAULT 'mp4' | 文件格式 |
| description | TEXT | NULL | 简介 |
| view_count | INT | DEFAULT 0 | 播放次数 |
| status | TINYINT | DEFAULT 1 | 上下架状态 |
| upload_time | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | 上传时间 |
| remark | VARCHAR(255) | NULL | 备注 |

---

## 15. 预约相关三表

### `coach_availability_slot`（教练可预约时段）
- `id` PK
- `coach_id` FK
- `start_time` / `end_time`
- `status`（OPEN/CLOSED）
- `booked`（是否已占用）

### `trial_booking`（体验课预约）
- `id` PK
- `student_id` / `coach_id` / `slot_id` FK
- `subject`（SUBJECT_2/SUBJECT_3）
- `booking_type`（FREE_TRIAL）
- `status`（BOOKED/CONFIRMED/CANCELLED）
- `created_at`
- 唯一约束：`slot_id`（一个时段只能有一条预约）

### `coach_notification`（教练提醒）
- `id` PK
- `coach_id` / `booking_id` FK
- `type`（TRIAL_BOOKING_CREATED / TRIAL_BOOKING_CONFIRMED）
- `content`
- `is_read`
- `created_at`

---

## 三、关系图（文字版）

- `student` 1---n `student_question_stats` n---1 `questions`
- `student` 1---n `student_wrong_questions` n---1 `questions`
- `student` 1---n `student_favorite_questions` n---1 `questions`
- `questions` 1---n `question_images` n---1 `images`
- `student` 1---n `trial_booking` n---1 `coach`
- `coach_availability_slot` 1---1 `trial_booking`
- `trial_booking` 1---n `coach_notification`
- `student`/`coach`/`admin` 与 `auth_account` 通过 `account_id + role` 对应

---

## 四、你当前最值得优先精简的点（建议）

如果你担心“繁杂”，建议优先看 `questions` 这张表：

1. **业务刚需保留**（先别动）
   - `id, question_text, answer, subject, type`
   - `items_title_array, items_desc_array`
   - `answer_skill, answer_skill_explain, remark`

2. **可评估精简**（看是否真实使用）
   - `chapter_id, difficulty_level, easy_error_flag, error_prone_flag, error_rate`
   - `flag, new_rule_flag, score, secret_flag, selected_flag, style, type_desc, version_no`

3. **精简策略**
   - 先做“字段使用盘点”（后端查询 + 前端显示 + 管理端编辑）
   - 将“从未读写”的字段列入候选删除清单
   - 分两步迁移：先停写停读，再删列

---

## 五、附：本项目当前前端对题目字段的主要使用

- 显示题干：`question_text`
- 正确答案校验：`answer`
- 选项：`items_title_array + items_desc_array`
- 解析区：优先 `answer_skill_explain`，再 `answer_skill`，再 `remark`
- 解析配图：`question_images + images`

---

如果你希望，我下一步可以继续给你做一个 **“可删字段候选清单（按风险分级）”**：
- 低风险可删（确认未使用）
- 中风险需迁移
- 高风险暂不建议动
