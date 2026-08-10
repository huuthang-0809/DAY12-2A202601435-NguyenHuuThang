# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Hữu Thắng |
| Mã học viên | 2A202601435 |
| Repo | https://github.com/huuthang-0809/DAY12-2A202601435-NguyenHuuThang |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-production-6d10.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

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

# 1. Liveness — mong đợi 200 {"status":"ok"}
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> curl.exe -i "$URL/health"       
HTTP/1.1 200 OK                           
Content-Type: application/json            
Date: Mon, 10 Aug 2026 07:05:39 GMT
Server: railway-hikari                                 
x-railway-request-id: oymQCsUOQ2iHA5M-9o6EoQ
Content-Length: 57
x-hikari-trace: sin1.tr00                    
x-railway-edge: sin1
Connection: keep-alive

{"status":"ok","service":"day12-agent","version":"1.0.0"}

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> curl.exe -i "$URL/ready"        
HTTP/1.1 200 OK                           
Content-Type: application/json            
Date: Mon, 10 Aug 2026 07:09:45 GMT
Server: railway-hikari                                 
x-railway-request-id: xJzYbUQzTjusXeoTwUFZXw
Content-Length: 31                                       
x-hikari-trace: sin1.98a6                    
x-railway-edge: sin1            
Connection: keep-alive          
              
{"status":"ready","redis":true}  

# 3. Không có API key — mong đợi 401
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> $body = @{ question = "Hello" } | ConvertTo-Json -Compress
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> $bodyBytes = [System.Text.Encoding]::UTF8.GetBytes($body)
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> 
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> try {
>>   Invoke-RestMethod `
>>     -Method POST `
>>     -Uri "$URL/ask" `
>>     -ContentType "application/json; charset=utf-8" `
>>     -Body $bodyBytes
>> } catch {
>>   $_.Exception.Response.StatusCode.value__
>> }
401

# 4. Có API key — mong đợi 200 kèm câu trả lời
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> $headers = @{
>>   "X-API-Key" = $env:AGENT_API_KEY
>>   "X-User-Id" = "sv-test"
>> }
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> 
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> $body = @{ question = "Deploy la gi?" } | ConvertTo-Json -Compress
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> $bodyBytes = [System.Text.Encoding]::UTF8.GetBytes($body)
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> 
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> Invoke-RestMethod `
>>   -Method POST `
>>   -Uri "$URL/ask" `
>>   -Headers $headers `
>>   -ContentType "application/json; charset=utf-8" `
>>   -Body $bodyBytes


answer         : Ngáº¯n gá»n: Deploy la gi phá»¥ thuá»c vÃo ba yáº¿u tá» â cáº¥u hÃ¬nh qua biáº¿n 
                 mÃ´i trÆ°á»á» orchestrator biáº¿t tráº¡ng thÃ¡i, vÃ giá» háº¡n tÃ
                 i nguyÃªn. (MÃ¬nh Äang nhá»Æ°á»£t trao Äá»i trÆ°á» ÄÃ³.)
user_id        : sv-test
history_length : 20
cost_usd       : 9.555E-05
tokens         : @{in=449; out=47}

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> $headers = @{
>>   "X-API-Key" = $env:AGENT_API_KEY                                                                      
>>   "X-User-Id" = "sv-test"                                                                               
>> }                                                                                                       
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang>                                          
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> $body = @{ question = "test" } | ConvertTo-Json -Compress
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> $bodyBytes = [System.Text.Encoding]::UTF8.GetBytes($body)
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> 
(.venv) (base) PS D:\AI_Vin\LAB\DAY12-2A202601435-NguyenHuuThang> 1..15 | ForEach-Object {
>>   try {
>>     Invoke-WebRequest `
>>       -Method POST `
>>       -Uri "$URL/ask" `
>>       -Headers $headers `
>>       -ContentType "application/json; charset=utf-8" `
>>       -Body $bodyBytes | Out-Null
>>     Write-Host 200 -NoNewline
>>     Write-Host " " -NoNewline
>>   } catch {
>>     Write-Host $_.Exception.Response.StatusCode.value__ -NoNewline
>>     Write-Host " " -NoNewline
>>   }
>> }
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429 
## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

