# Coolify Remote Command Action

GitHub Action để chạy lệnh trên tất cả Coolify servers qua SSH, chạy song song với matrix batches.

## Tính năng

- **Dynamic Matrix + Parallel**: Tự động chia servers vào 10 matrix batches chạy song song
- **Manual Trigger**: Nhập lệnh qua GitHub UI (workflow_dispatch)
- **Status Report**: Push `status/latest.md` (báo cáo đầy đủ + output) + `status/history.md` (lịch sử các run) lên branch `status`
- **Auto History Cleanup**: Tự xóa workflow runs cũ, giữ 5 run gần nhất

## Secrets cần setup

| Secret | Mô tả | Ví dụ |
|--------|-------|-------|
| `COOLIFY_API_URL` | Full API URL | `https://coolify.example.com/api/v1` |
| `COOLIFY_API_TOKEN` | API bearer token | `xxxxxxxxxxxx` |
| `SSH_PRIVATE_KEY` | Private key để SSH vào các servers | `-----BEGIN OPENSSH PRIVATE KEY-----...` |

## Setup

1. Trong repo của bạn, vào **Settings → Secrets and variables → Actions**
2. Thêm 3 secrets:
   - `COOLIFY_API_URL` — Coolify API endpoint
   - `COOLIFY_API_TOKEN` — API token (có quyền đọc servers)
   - `SSH_PRIVATE_KEY` — Private key để SSH vào tất cả managed servers
3. Push file workflow vào `.github/workflows/coolify-remote-command.yml`
4. Vào **Actions → Coolify Remote Command → Run workflow**
5. Nhập lệnh và chạy

## Ví dụ lệnh

```
hostname && uptime
docker ps --format '{{.Names}} {{.Status}}'
df -h /
free -m
systemctl status docker
```

## Performance

- 10 concurrent runners (matrix batches, tự chia servers đều)
- 10 concurrent SSH sessions per batch
- 35 servers: ~10-30 giây tùy lệnh
- `MATRIX_BATCHES` có thể chỉnh trong workflow

## Status Report

Sau mỗi run, vào branch `status` để xem:
- `latest.md` — Báo cáo chi tiết + output của từng server
- `history.md` — Lịch sử các lần chạy

## Cách hoạt động

1. **get-servers**: Gọi Coolify API để lấy danh sách IP servers
2. **execute**: Chia servers thành batches, SSH vào từng server chạy lệnh
3. **report**: Tổng hợp kết quả, push status branch

## Lưu ý

- SSH key phải có quyền truy cập tất cả managed servers (user `root` hoặc user tương đương)
- Nếu một server unreachable, batch vẫn tiếp tục với servers khác
- Timeout mỗi SSH connection: 60 giây
- Output tối đa ~1MB/server (giới hạn của GitHub Actions artifact)
