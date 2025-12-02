# SOP生成流程图

展示SOP（标准操作流程）的生成流程。

## 代码入口

| 类/函数 | 文件路径 | 说明 |
|---------|----------|------|
| `SOPManageService` | `src/backend/bisheng/api/services/linsight/sop_manage.py` | SOP管理服务 |
| `LinsightSOPDao` | `src/backend/bisheng/database/models/linsight_sop.py` | SOP数据访问 |
| `LinsightSOPRecord` | `src/backend/bisheng/database/models/linsight_sop.py` | SOP记录模型 |
| `LinsightWorkbenchImpl` | `src/backend/bisheng/api/services/linsight/workbench_impl.py` | 工作台实现 |
| `linsight_execute_utils` | `src/backend/bisheng/linsight/utils.py` | 执行工具函数 |

```mermaid
flowchart TD
    A[接收用户问题] --> B[分析问题类型]
    B --> C{有参考文件?}
    C -->|是| D[解析文件内容]
    C -->|否| E[直接分析问题]
    D --> F[提取关键信息]
    F --> G[构建SOP生成Prompt]
    E --> G
    G --> H[调用LLM生成SOP]
    H --> I[解析SOP结构]
    I --> J{SOP格式正确?}
    J -->|否| K[重新生成]
    K --> H
    J -->|是| L[返回SOP给用户]
    L --> M{用户满意?}
    M -->|否| N[用户修改SOP]
    N --> O[保存修改后SOP]
    M -->|是| O
    O --> P[SOP确认完成]
```

## SOP说明

### SOP结构

```json
{
  "title": "SOP标题",
  "description": "SOP描述",
  "steps": [
    {
      "step_id": 1,
      "name": "步骤名称",
      "description": "步骤描述",
      "tools": ["tool1", "tool2"],
      "expected_output": "预期输出"
    }
  ]
}
```

### 生成要点

| 要点 | 说明 |
|------|------|
| 步骤清晰 | 每步目标明确 |
| 逻辑连贯 | 步骤间有逻辑关系 |
| 可执行 | 每步可被Agent执行 |
| 可验证 | 有明确的完成标准 |

### 用户交互

- 用户可查看生成的SOP
- 用户可修改SOP内容
- 用户确认后开始执行
- 支持SOP模板复用
