# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Xuân Trường |
| Mã học viên | 2A202602761 |
| Repo | https://github.com/truongapep/K4-L3B-DAY12-NguyenXuanTruong-2A202602761-CloudServicesAndDeployment.git |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k4-l3b-day12-nguyenxuantruong-2a202602761-clouds-production.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-09-29 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và nguồn giá trị, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis add-on của Railway |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |


## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i <URL>/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
(.venv) PS D:\BaiLab\lab12 Cloud\K4-L3B-Day12-NguyenXuanTruong-2A202602761-Cloud-Service-And-Deployment> curl.exe -i https://k4-l3b-day12-nguyenxuantruong-2a202602761-clouds-production.up.railway.app/health
HTTP/1.1 200 OK
Content-Type: application/json
Date: Tue, 29 Sep 2026 04:40:11 GMT
Server: railway-hikari
x-railway-request-id: WqOdZO21SUCiP7NQn6XIxQ
Content-Length: 57
x-hikari-trace: hkg1.hn7d
x-railway-edge: hkg1
Connection: keep-alive

{"status":"ok","service":"day12-agent","version":"1.0.0"}
(.venv) PS D:\BaiLab\lab12 Cloud\K4-L3B-Day12-NguyenXuanTruong-2A202602761-Cloud-Service-And-Deployment> curl.exe -i https://k4-l3b-day12-nguyenxuantruong-2a202602761-clouds-production.up.railway.app/ready 
HTTP/1.1 200 OK
Content-Type: application/json
Date: Tue, 29 Sep 2026 04:40:31 GMT
Server: railway-hikari
x-railway-request-id: VxlCLSwPSB6RQLWnxtoGcA
Content-Length: 31
x-hikari-trace: sin1.hs0s
x-railway-edge: sin1
Connection: keep-alive

{"status":"ready","redis":true}

(.venv) PS D:\BaiLab\lab12 Cloud\K4-L3B-Day12-NguyenXuanTruong-2A202602761-Cloud-Service-And-Deployment> try {
>>     Invoke-RestMethod -Uri "$URL/ask" -Method Post -ContentType "application/json" -Body '{"question":"Hello"}'
>> } catch {
>>     $_.Exception.Response.StatusCode.Value__
>> }
401

(.venv) PS D:\BaiLab\lab12 Cloud\K4-L3B-Day12-NguyenXuanTruong-2A202602761-Cloud-Service-And-Deployment> $headers = @{                      
>>     "X-API-Key" = $AGENT_API_KEY                                        
>>     "X-User-Id" = "sv-test"
>> }
>> $body = '{"question":"Deploy la gi?"}'
>> 
>> Invoke-RestMethod -Uri "$URL/ask" -Method Post -ContentType "application/json" -Headers $headers -Body $body


answer         : Ngáº¯n gá»n: Deploy la gi phá»¥ thuá»c vÃo ba yáº¿u tá» â cáº¥u hÃ¬nh qua biáº¿n mÃ´i trÆ°á»á» 
                 orchestrator biáº¿t tráº¡ng thÃ¡i, vÃ giá» háº¡n tÃi nguyÃªn.
user_id        : sv-test
history_length : 0
cost_usd       : 2.265E-05
tokens         : @{in=3; out=37}



(.venv) PS D:\BaiLab\lab12 Cloud\K4-L3B-Day12-NguyenXuanTruong-2A202602761-Cloud-Service-And-Deployment> 

(.venv) PS D:\BaiLab\lab12 Cloud\K4-L3B-Day12-NguyenXuanTruong-2A202602761-Cloud-Service-And-Deployment> $headers = @{
>>     "X-API-Key" = $AGENT_API_KEY                                                                                                         
>>     "X-User-Id" = "sv-test"                                                                                                              
>> }                                                                                                                                        
>> $body = '{"question":"test"}'                                                                                                            
>> 
>> 1..15 | ForEach-Object {
>>     try {
>>         $res = Invoke-WebRequest -Uri "$URL/ask" -Method Post -ContentType "application/json" -Headers $headers -Body $body -UseBasicParsing
>>         Write-Host "$($res.StatusCode) " -NoNewline
>>     } catch {
>>         if ($_.Exception.Response) {
>>             Write-Host "$([int]$_.Exception.Response.StatusCode) " -NoNewline
>>         } else {
>>             Write-Host "Err " -NoNewline
>>         }
>>     }
>> }; Write-Host ""
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429 
(.venv) PS D:\BaiLab\lab12 Cloud\K4-L3B-Day12-NguyenXuanTruong-2A202602761-Cloud-Service-And-Deployment> 
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

