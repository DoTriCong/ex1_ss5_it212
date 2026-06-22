# BÀI 1: Phân tích & Lựa chọn (Kỹ thuật Prompt suy luận theo các bước - Chain-of-thought)

**Đáp án lựa chọn:** **B**

---

## Phân tích chi tiết phương án B
Phương án B là tối ưu nhất vì nó áp dụng đầy đủ kỹ thuật **Chain-of-thought (CoT)** và cấu trúc **5 thành phần Prompt**:

- **Role (Vai trò):** "Hãy đóng vai trò là một Java Developer chuyên nghiệp kiêm chuyên gia tài chính" → xác định rõ AI phải vừa có kỹ năng lập trình, vừa hiểu nghiệp vụ tài chính.  
- **Goal (Mục tiêu):** Viết class Java tính thuế TNCN lũy tiến dựa trên thu nhập chịu thuế.  
- **Context (Ngữ cảnh):** Sử dụng Java 17, kiểu dữ liệu BigDecimal để tránh sai số tiền tệ.  
- **Constraint (Ràng buộc):** Áp dụng biểu thuế lũy tiến từng phần, giữ nguyên quy định giảm trừ gia cảnh và bảo hiểm.  
- **Format (Định dạng):** Yêu cầu AI phân tích từng bước (CoT):  
  1. Phân tích cách xác định thu nhập tính thuế.  
  2. Liệt kê công thức cho từng bậc.  
  3. Dry-run với ví dụ cụ thể.  
  4. Cuối cùng sinh mã nguồn Java hoàn chỉnh.  

→ Nhờ có CoT, AI sẽ suy luận tuần tự, tránh sai sót về ranh giới bậc thuế và lỗi làm tròn.

---

## Nhược điểm của phương án A
- **Quá chung chung:** Chỉ yêu cầu viết hàm Java, không nêu rõ giảm trừ gia cảnh, bảo hiểm, hay kiểu dữ liệu.  
- **Thiếu CoT:** Không buộc AI phân tích từng bước, dễ dẫn đến sai logic hoặc bỏ sót chi tiết.  
- **Nguy cơ:** AI có thể dùng kiểu `double`, gây sai số làm tròn.

---

## Nhược điểm của phương án C
- **Sai trọng tâm:** Tập trung vào hiệu năng với Stream API song song, trong khi bài toán là chính xác nghiệp vụ thuế.  
- **Thiếu CoT:** Không yêu cầu phân tích từng bước, dễ sai ranh giới bậc thuế.  
- **Nguy cơ:** Code phức tạp, khó bảo trì, nhưng vẫn có thể sai logic tính thuế.

---

## Mã nguồn Java hoàn chỉnh (theo phương án B)

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

public class TaxCalculator {

    public static BigDecimal calculateTax(BigDecimal totalIncome, int dependents, boolean hasInsurance) {
        if (totalIncome == null || totalIncome.compareTo(BigDecimal.ZERO) <= 0) {
            return BigDecimal.ZERO;
        }

        // Giảm trừ gia cảnh
        BigDecimal personalDeduction = new BigDecimal("11000000");
        BigDecimal dependentDeduction = new BigDecimal("4400000").multiply(BigDecimal.valueOf(dependents));

        // Bảo hiểm bắt buộc (10.5%)
        BigDecimal insurance = hasInsurance ? totalIncome.multiply(new BigDecimal("0.105")) : BigDecimal.ZERO;

        // Thu nhập chịu thuế
        BigDecimal taxableIncome = totalIncome.subtract(personalDeduction)
                                              .subtract(dependentDeduction)
                                              .subtract(insurance);

        if (taxableIncome.compareTo(BigDecimal.ZERO) <= 0) {
            return BigDecimal.ZERO;
        }

        BigDecimal tax = BigDecimal.ZERO;

        // Các bậc thuế
        BigDecimal[] brackets = {
            new BigDecimal("5000000"),
            new BigDecimal("10000000"),
            new BigDecimal("18000000"),
            new BigDecimal("32000000"),
            new BigDecimal("52000000"),
            new BigDecimal("80000000")
        };

        BigDecimal[] rates = {
            new BigDecimal("0.05"),
            new BigDecimal("0.10"),
            new BigDecimal("0.15"),
            new BigDecimal("0.20"),
            new BigDecimal("0.25"),
            new BigDecimal("0.30"),
            new BigDecimal("0.35")
        };

        BigDecimal prevLimit = BigDecimal.ZERO;

        for (int i = 0; i < brackets.length; i++) {
            if (taxableIncome.compareTo(brackets[i]) > 0) {
                BigDecimal taxableAtThisRate = brackets[i].subtract(prevLimit);
                tax = tax.add(taxableAtThisRate.multiply(rates[i]));
                prevLimit = brackets[i];
            } else {
                BigDecimal taxableAtThisRate = taxableIncome.subtract(prevLimit);
                tax = tax.add(taxableAtThisRate.multiply(rates[i]));
                return tax.setScale(0, RoundingMode.HALF_UP);
            }
        }

        // Trên 80 triệu
        BigDecimal taxableAtThisRate = taxableIncome.subtract(prevLimit);
        tax = tax.add(taxableAtThisRate.multiply(rates[rates.length - 1]));

        return tax.setScale(0, RoundingMode.HALF_UP);
    }

    public static void main(String[] args) {
        BigDecimal income = new BigDecimal("30000000");
        int dependents = 1;
        boolean hasInsurance = false;

        BigDecimal tax = calculateTax(income, dependents, hasInsurance);
        System.out.println("Thuế TNCN phải nộp: " + tax + " VND");
    }
}
