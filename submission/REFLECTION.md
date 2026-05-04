# Reflection: Anti-Pattern Risk Analysis

## Anti-Pattern Được Chọn: Small-File Problem (Vấn đề tập tin nhỏ)

Trong hệ thống LLM observability của team, chúng tôi **rất dễ gặp small-file problem** vì hai lý do chính:

### 1. Tính Chất Streaming của LLM Calls
Mỗi LLM request được ghi lập tức sau khi hoàn thành (latency ~200-500ms). Nếu team không batch writes lại, mỗi giờ từ 100+ endpoints khác nhau sẽ tạo ra **hàng ngàn request nhỏ**, dẫn đến **hàng chục nghìn files** trong Bronze layer. Điều này làm chậm query **10-100 lần** so với data được compact.

### 2. Lack of Awareness
Không phải AI engineer đều biết về Delta compaction. Nếu không có OPTIMIZE tự động hàng đêm, lake sẽ **tích tụ garbage files** và performance sẽ **degrade dần mà không ai nhận ra** cho đến khi thứ 4 hoặc thứ 5 của tuần đó.

### 3. Chi Phí Cloud Storage
Mỗi file nhỏ vẫn chiếm ~4KB metadata overhead trên S3. Với 100K files nhỏ, đó là ~400MB metadata cost hàng tháng — lãng phí vì data chỉ ~200MB thực tế.

## Giải Pháp
- OPTIMIZE + Z-ORDER theo `user_id` hoặc `model` hàng đêm (NB2 chứng minh speedup 3-10×)
- Tăng batch size từ application layer (aggregate ≥ 100 calls/batch trước ghi)
- Monitor file count hàng tuần; trigger alert nếu > 1000 files

**Kết luận:** Small-file problem **dễ bị bỏ qua** nhưng **tác động lớn** → team cần setup automated compaction từ đầu.

---

*Độ dài: ~160 từ*
