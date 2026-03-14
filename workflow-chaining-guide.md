# BMAD Workflow Chaining Guide

## Vấn đề của bạn

Bạn muốn:
1. Chạy lần lượt: `/create-story` → `/dev-story` → `/review-code` (code-review)
2. Chạy trong **yolo mode** (bỏ qua confirmations)
3. Trong **1 session mới** (fresh context)
4. Để lại các action cần manual hoặc secret vào file riêng

---

## Giải pháp hiện tại của BMAD

### 1. YOLO Mode

BMAD đã có **yolo mode** tích hợp. Khi chạy workflow:

- Sau mỗi `template-output` tag, hệ thống sẽ hỏi: `[a] Advanced Elicitation, [c] Continue, [p] Party-Mode, [y] YOLO the rest`
- Chọn **`y`** để skip tất cả confirmations còn lại

**Tuy nhiên:**
- YOLO chỉ hoạt động **trong 1 workflow** (không chain được)
- Mỗi workflow (create-story, dev-story, code-review) là **subprocess riêng**
- Không có cơ chế chạy liên tiếp tự động

### 2. Workflow Chaining trong BMAD

BMAD có `invoke-workflow` tag nhưng chỉ dùng **trong nội bộ 1 workflow** (gọi workflow con), không phải để chain nhiều workflow chính.

---

## Giải pháp đề xuất

### Cách 1: Sử dụng BMAD Party Mode

Party Mode (`/bmad-party-mode`) có thể orchestrate multiple agents:

```markdown
Run: /bmad-party-mode
Explain: "I want to run create-story, dev-story, and code-review in sequence with yolo mode"
```

**Ưu:** Native BMAD feature
**Nhược:** Cần test thêm vì party mode hơi khác use case

### Cách 2: Tạo Wrapper Workflow (Custom)

Tạo 1 workflow wrapper gọi 3 workflow lần lượt:

```yaml
# _bmad/core/workflows/chain-workflows/workflow.yaml
name: chain-workflows
description: 'Run multiple workflows in sequence with yolo mode'

chain:
  - workflow: "bmm-create-story"
    yolo: true
  - workflow: "bmm-dev-story"
    yolo: true
  - workflow: "bmm-code-review"
    yolo: true
```

### Cách 3: Script bên ngoài (Hiện tại khả thi nhất)

Tạo script chạy nhiều Claude session:

```bash
#!/bin/bash
# chain-bmad.sh

# Session 1: Create Story (yolo)
claude -p "Run /bmad-bmm-create-story in yolo mode" --continue

# Session 2: Dev Story (yolo)
claude -p "Run /bmad-bmm-dev-story in yolo mode" --continue

# Session 3: Code Review (yolo)
claude -p "Run /bmad-bmm-code-review in yolo mode" --continue
```

---

## Giải pháp TỐT NHẤT cho bạn

### Step-by-step guide:

#### Bước 1: Tạo file hướng dẫn cho manual actions

Tạo `MANUAL_ACTIONS.md` trong project:

```markdown
# Manual Actions Required

## Secrets/Keys cần set:
- [ ] DATABASE_URL
- [ ] API_KEY
- [ ] AWS_CREDENTIALS

## Manual Steps:
- [ ] Deploy staging environment
- [ ] Run migration scripts
- [ ] Verify OAuth setup
```

#### Bước 2: Chạy từng workflow với yolo

**Workflow 1: Create Story**
```
/bmad-bmm-create-story
→ Sau mỗi template-output: nhập [y] để YOLO
```

**Workflow 2: Dev Story**
```
/bmad-bmm-dev-story
→ Tự động discover story file từ Bước 1
→ Với mỗi template-output: nhập [y]
```

**Workflow 3: Code Review**
```
/bmad-bmm-code-review
→ Review code từ Bước 2
```

---

## Tìm hiểu thêm: Có thể tạo "Chain Workflow" mới?

Có thể tạo 1 workflow mới trong BMAD:

```yaml
# _bmad/bmm/workflows/chain-story-cycle/workflow.yaml
name: chain-story-cycle
description: 'Run create-story → dev-story → code-review in one go'

variables:
  story_key: ""  # e.g., "1.1", "2.3"

steps:
  - step: 1
    title: "Create Story"
    invoke-workflow: "../create-story/workflow.yaml"
    yolo: true

  - step: 2
    title: "Dev Story"
    invoke-workflow: "../dev-story/workflow.yaml"
    yolo: true

  - step: 3
    title: "Code Review"
    invoke-workflow: "../code-review/workflow.yaml"
    yolo: true
```

**Lưu ý:** Cần kiểm tra xem `invoke-workflow` có hỗ trợ passing context giữa các workflow không.

---

## Recommendation

1. **Ngay bây giờ**: Chạy thủ công từng workflow với yolo mode
2. **Tương lai**: Có thể tạo 1 "chain-workflow" wrapper nếu cần thường xuyên

Bạn muốn tôi tạo draft cho wrapper workflow này không?
