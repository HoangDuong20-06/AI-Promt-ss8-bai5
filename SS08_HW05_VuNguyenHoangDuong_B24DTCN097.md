BÀI 5: Thiết kế Cổng Kiểm toán Đối soát Giao dịch
1. Mô tả ý đồ thiết kế

Hệ thống Reconciliation được xây dựng nhằm so sánh dữ liệu giao dịch giữa Core Banking và hệ thống đối tác thanh toán (Visa/Mastercard) để phát hiện các giao dịch sai lệch.

Quy trình được chia thành ba bước:

Bước 1: Phân tích hai phương án triển khai đối soát để lựa chọn giải pháp phù hợp với quy mô dữ liệu lớn.
Bước 2: Phân tích rủi ro về lệch múi giờ giữa hai hệ thống và chuẩn hóa thời gian về UTC trước khi đối chiếu.
Bước 3: Sinh mã nguồn Java Spring Boot cho dịch vụ đối soát với đầy đủ kiểm tra dữ liệu, xử lý ngoại lệ và ghi log.
BƯỚC 1
Prompt 1
Bạn hãy đóng vai trò là một System Architect nhiều kinh nghiệm.

Tôi đang xây dựng hệ thống Reconciliation cho ngân hàng SecureBank.

Mỗi ngày hệ thống nhận:

- File giao dịch từ Core Banking
- File giao dịch từ Visa/Mastercard

Hãy đề xuất và so sánh 2 phương án đối soát:

Phương án 1:
Đọc toàn bộ dữ liệu vào Java Collection (HashMap/List) rồi đối chiếu trong bộ nhớ.

Phương án 2:
Import dữ liệu vào Database và thực hiện SQL JOIN để đối soát.

Yêu cầu:

1. Giải thích nguyên lý hoạt động của từng phương án.

2. Lập bảng so sánh gồm:

- Hiệu năng
- Mức sử dụng RAM
- Khả năng mở rộng
- Độ phức tạp triển khai
- Khả năng xử lý khi có 5 triệu giao dịch/ngày

3. Đưa ra ưu nhược điểm.

4. Kết luận nên sử dụng phương án nào trong hệ thống ngân hàng thực tế và giải thích lý do.
Log Chat AI (Bước 1)

AI trả lời:

Tiêu chí	Java Collection	SQL JOIN
Hiệu năng	Rất nhanh khi dữ liệu vừa	Tốt với dữ liệu lớn
RAM	Rất cao	Thấp
CPU	Cao	Database xử lý
Mở rộng	Khó	Dễ
5 triệu giao dịch/ngày	Không phù hợp	Phù hợp
Phương án 1

Ưu điểm

Code đơn giản
Tra cứu HashMap O(1)
Không phụ thuộc DB

Nhược điểm

Tốn RAM
Dễ OutOfMemory
Không phù hợp dữ liệu lớn
Phương án 2

Ưu điểm

Database tối ưu Join
Có Index
Phân trang
Phù hợp Big Data

Nhược điểm

Phụ thuộc Database
SQL phức tạp hơn
Kết luận

Đối với ngân hàng xử lý khoảng 5 triệu giao dịch/ngày, nên sử dụng SQL JOIN vì khả năng mở rộng tốt, tiết kiệm bộ nhớ và dễ tối ưu hiệu năng.

BƯỚC 2
Prompt 2
Giả sử hệ thống Core Banking lưu thời gian theo GMT+7 trong khi Visa lưu theo GMT+0.

Điều này làm cho khi đối soát theo ngày giao dịch xuất hiện sai lệch mặc dù thực tế hai giao dịch là một.

Hãy phân tích:

1. Nguyên nhân.

2. Điều gì sẽ xảy ra nếu không chuẩn hóa múi giờ.

3. Đề xuất giải pháp bằng Java 8 Date Time API.

4. Hướng dẫn sử dụng ZonedDateTime để chuyển toàn bộ thời gian về UTC trước khi đối soát.

5. Minh họa bằng đoạn mã Java.
Log Chat AI (Bước 2)

AI trả lời:

Nguyên nhân:

Core Banking sử dụng GMT+7 trong khi Visa sử dụng UTC.

Ví dụ:

Core

2026-07-10T00:30+07:00

Visa

2026-07-09T17:30Z

Hai thời điểm giống nhau nhưng khác múi giờ.

Nếu không chuẩn hóa:

Sai ngày giao dịch
Báo lỗi giả
Đối soát sai

Giải pháp:

Chuyển toàn bộ về UTC trước khi so sánh.

Ví dụ:

ZonedDateTime coreTime =
        ZonedDateTime.parse("2026-07-10T00:30:00+07:00");

ZonedDateTime utc =
        coreTime.withZoneSameInstant(ZoneOffset.UTC);

System.out.println(utc);

Kết quả

2026-07-09T17:30Z

Sau khi chuẩn hóa UTC, hai hệ thống sẽ có cùng thời điểm để đối chiếu.

BƯỚC 3
Prompt 3
Đóng vai trò Senior Java Developer.

Hãy suy nghĩ từng bước trước khi viết mã (Chain of Thought nội bộ), nhưng chỉ trình bày kết quả cuối cùng.

Viết dịch vụ Spring Boot:

ReconciliationService

có phương thức

reconcile(
List<Transaction> coreTxns,
List<Transaction> partnerTxns
)

Yêu cầu:

- Chuẩn hóa toàn bộ thời gian về UTC bằng ZonedDateTime.
- So khớp theo Transaction ID.
- Nếu lệch số tiền thì ném ngoại lệ.
- Nếu lệch trạng thái thì ghi lỗi.
- Nếu thiếu giao dịch thì ghi log.
- Ghi log bằng @Slf4j.
- Viết đầy đủ import.
- Code sạch theo Clean Code.
- Có comment ngắn giải thích từng bước.
Log Chat AI (Bước 3)

AI trả lời:

package com.securebank.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.time.ZoneOffset;
import java.time.ZonedDateTime;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Slf4j
@Service
public class ReconciliationService {

    public void reconcile(List<Transaction> coreTxns,
                          List<Transaction> partnerTxns) {

        Map<String, Transaction> partnerMap = new HashMap<>();

        for (Transaction txn : partnerTxns) {
            partnerMap.put(txn.getId(), txn);
        }

        for (Transaction core : coreTxns) {

            Transaction partner = partnerMap.get(core.getId());

            if (partner == null) {
                log.error("Missing transaction: {}", core.getId());
                continue;
            }

            ZonedDateTime coreUtc =
                    core.getTransactionTime()
                            .withZoneSameInstant(ZoneOffset.UTC);

            ZonedDateTime partnerUtc =
                    partner.getTransactionTime()
                            .withZoneSameInstant(ZoneOffset.UTC);

            if (!coreUtc.equals(partnerUtc)) {
                log.warn("Timezone difference: {}", core.getId());
            }

            if (core.getAmount().compareTo(partner.getAmount()) != 0) {
                throw new RuntimeException(
                        "Amount mismatch: " + core.getId());
            }

            if (!core.getStatus().equals(partner.getStatus())) {
                log.error("Status mismatch: {}", core.getId());
            }

            log.info("Transaction {} reconciled successfully",
                    core.getId());
        }
    }
}