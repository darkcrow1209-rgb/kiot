# Quản lý cho thuê kiosk — Source code

Web app quản lý kiosk cho thuê: dashboard, quản lý kiosk / khách hàng / hợp đồng,
tiền thuê hàng tháng, nhập chỉ số điện nước, bill tháng (in / xuất PDF),
yêu cầu khách hàng, chi phí phát sinh, theo dõi thanh toán & công nợ, báo cáo, cài đặt.
Giao diện tiếng Việt, responsive desktop + mobile.

> **Lưu ý quan trọng:** source này được build để chạy trên nền tảng Muse.
> Lớp server (`server/src/actions.ts`) dùng SDK riêng của nền tảng
> (`@hatch/space-sdk`, file `vendor/space-sdk.tgz` đính kèm để tham khảo API)
> và database SQLite do nền tảng quản lý. Muốn chạy độc lập / deploy riêng,
> bạn cần thay lớp server bằng backend của mình (xem mục "Chạy độc lập" bên dưới).
> Phần giao diện React (`client/`) và schema database (`server/src/schema.ts`,
> `drizzle/`) dùng lại nguyên vẹn được.

## Cấu trúc thư mục

```
quan-ly-kiosk/
├── client/                 # Frontend React 19 + Tailwind CSS 4 + Recharts
│   ├── index.html
│   ├── build.mjs           # Script build frontend (esbuild)
│   └── src/
│       ├── App.tsx         # Toàn bộ UI (dashboard + 12 module)
│       ├── api.ts          # Client gọi server actions (qua @hatch/space-sdk)
│       ├── main.tsx        # Entry point
│       └── theme.css       # Theme / style
├── server/
│   └── src/
│       ├── schema.ts       # Schema database (drizzle-orm): kiosks, customers,
│       │                   # contracts, utility_readings, bills, payments,
│       │                   # customer_requests, expenses, notifications, settings
│       └── actions.ts      # Server actions: CRUD + tạo bill tháng + xuất PDF
├── drizzle/                # SQL migration (0001, 0002)
├── vendor/
│   └── space-sdk.tgz       # SDK của nền tảng Muse (tham khảo, không cần khi tự host)
├── package.json
├── bun.lock
├── bunfig.toml
└── tsconfig.base.json
```

## Chạy độc lập (tự host / deploy)

1. **Thay lớp server:** viết lại `server/src/actions.ts` thành API backend của bạn
   (ví dụ Express / Hono / Fastify + `drizzle-orm` + SQLite hoặc PostgreSQL).
   Giữ nguyên tên các action và shape dữ liệu trả về trong file hiện tại để
   frontend không phải sửa logic.
2. **Sửa `client/src/api.ts`:** thay `createActionClient` bằng các hàm `fetch`
   gọi tới backend của bạn (giữ nguyên tên hàm: `getAppData`, `populateDemo`,
   `saveKiosk`, `deleteKiosk`, `saveCustomer`, `deleteCustomer`, `saveContract`,
   `deleteContract`, `addUtilityReading`, `createMonthlyBills`, ... — xem đầy đủ
   trong `server/src/actions.ts`).
3. **Database:** chạy 2 file SQL trong `drizzle/` theo thứ tự để tạo bảng.
   Dữ liệu demo được nạp bởi action `populateDemo` (gọi 1 lần khi app trống).
4. **Build frontend:** `bun install` rồi `bun run build:client` (cần bun >= 1.3).
   File tĩnh sinh ra ở `client/dist/`, serve như mọi SPA (kèm fallback về index.html).
5. Trong `package.json`, xóa dependency `@hatch/space-sdk`
   (đang trỏ tới `file:/opt/hatch/...` — chỉ tồn tại trên máy build gốc).

## Tính năng chính

- Dashboard: tổng kiosk / đang thuê / còn trống, doanh thu tháng, công nợ,
  bill sắp đến hạn, yêu cầu chưa xử lý, biểu đồ doanh thu theo tháng
- CRUD kiosk, khách hàng, hợp đồng (cảnh báo hợp đồng sắp hết hạn / đã hết hạn)
- Nhập chỉ số điện/nước kỳ trước – kỳ này, tự tính tiêu thụ × đơn giá
- Tạo bill tháng gộp: tiền thuê + điện + nước + phí quản lý/dịch vụ/khác − giảm giá
- Ghi nhận thanh toán (kể cả thanh toán một phần), tự đánh dấu quá hạn
- Ticket yêu cầu khách hàng (mới / đang xử lý / chờ KH / hoàn thành / đóng)
- Chi phí phát sinh: phân loại chủ chịu / khách chịu (đưa vào bill)
- Báo cáo doanh thu, công nợ, chi phí + xuất CSV; thông báo tập trung; cài đặt đơn giá
