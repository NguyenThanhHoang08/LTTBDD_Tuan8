# Trả lời câu hỏi bổ sung — Buổi 8

### 1. Cờ `ready` trong `useAutoPersist`
Ở lần render đầu tiên khi mới mở app, giỏ hàng trong Redux store đang có giá trị khởi tạo là mảng rỗng `[]`. Nếu bỏ điều kiện `ready`, `useAutoPersist` sẽ chạy ngay ở lần render đầu và ghi đè mảng rỗng `[]` vào `AsyncStorage` trước khi hàm bất đồng bộ `loadJSON` kịp đọc dữ liệu cũ lên, làm mất toàn bộ giỏ hàng đã lưu. Cờ `ready` chỉ bật lên `true` sau khi quá trình đọc `AsyncStorage` và `hydrate` vào store hoàn tất, giúp chặn việc ghi sớm ngoài ý muốn.

### 2. Token giả và bảo mật xác thực
- **Hai điểm không an toàn:** (1) Mật khẩu người dùng lưu dạng văn bản thuần (plain text) trong `db.json` và được tải về để so khớp ngay ở phía client (app), nên ai cũng có thể đọc trộm được; (2) Token do phía client tự sinh ra theo quy tắc đơn giản nên kẻ tấn công dễ dàng tự tạo token giả mạo.
- **Trong hệ thống thật:** Server (Backend) là nơi kiểm tra mật khẩu (so sánh với chuỗi băm bcrypt/argon2 trong cơ sở dữ liệu) và ký phát JWT token có thời hạn. Trên thiết bị di động, token nhạy cảm nên được lưu trong vùng nhớ mã hóa an toàn như `expo-secure-store` (Keychain trên iOS / Keystore trên Android) thay vì `AsyncStorage` dạng thô.

### 3. Hai `useEffect` trong `CartItemRow`
Nếu xóa `useEffect` đồng bộ `setQty(item.quantity)`, số lượng hiển thị trên nút (`qty` cục bộ) sẽ bị lệch so với Redux store khi store thay đổi từ bên ngoài `CartItemRow`. Ví dụ: khi người dùng quay lại màn Chi tiết sách và bấm "Thêm vào giỏ" cho cuốn sách đã có sẵn trong giỏ (store tăng `quantity` lên nhưng state `qty` cục bộ của dòng đó vẫn giữ giá trị cũ), hoặc khi dữ liệu giỏ hàng được nạp lại từ `AsyncStorage`.

### 4. Một nguồn state (Single Source of Truth) với `HeartButton`
Nếu mỗi `HeartButton` tự giữ trạng thái bằng `useState` cục bộ riêng biệt, nút tim trên thẻ sách ở lưới (Home) và nút tim ở màn Chi tiết sách sẽ hoàn toàn độc lập: bấm thả tim ở Home rồi bấm vào xem chi tiết cuốn sách đó sẽ thấy tim ở trang chi tiết vẫn chưa được chọn (màu trắng). Việc dùng chung Redux store (`wishlistSlice`) tạo ra một nguồn dữ liệu duy nhất: mọi `HeartButton` đều đọc chung mảng `ids` và gọi chung `toggle(bookId)`, nên khi một nơi thay đổi thì tất cả các nơi khác tự động đồng bộ ngay lập tức.

### 5. Code bổ sung: Nút hiện/ẩn mật khẩu (👁) ở màn Đăng nhập
Đã cập nhật trong `src/screens/LoginScreen.tsx`:
Màn hình kết quả nằm trong file docx

```tsx
const [showPassword, setShowPassword] = useState(false);

<View style="{styles.passwordWrap}">
  <FormField 42 error="{errors.password}" label="Mật khẩu" onChangeText="{setPassword}" paddingRight: secureTextEntry="{!showPassword}" style="{{" value="{password}" }}/>
  <Pressable onPress="{()"> setShowPassword((prev) => !prev)}
    style={styles.eyeBtn}
    hitSlop={8}
    accessibilityLabel={showPassword ? "Ẩn mật khẩu" : "Hiện mật khẩu"}
  >
    <Text style="{styles.eyeIcon}">{showPassword ? "🙈" : "👁"}</Text>
  </Pressable>
</View>
