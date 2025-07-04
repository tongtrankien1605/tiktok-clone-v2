## 🚀 **tiktok-clone-v2 : Lướt video tương tự tiktok, sử dụng Base_URL để chủ động khi thay đổi tài khoản hoặc tên repository mà không cần sửa logic** 🚀

**- Trang được code dựa theo cấu trúc nhỏ giống Tiktok, dùng để xem các video đã được upload lên server**

</br>

**- Phiên bản v2: khác bản gốc ở chỗ là có hỗ trợ 3 nguồn link video: CDN jsDelivr, CDN catbox.moe, CDN anh.moe**


</br>

## 🌐 Truy cập để xem [TẠI ĐÂY](https://tongtrankien1605.github.io/tiktok-clone-v2) nhé !

</br>

## 🏆 Đây là code base, có thể sử dụng để xây dựng phát triển thêm các chương trình lớn hơn

            - Mỗi lần truy cập sẽ tải dữ liệu video từ ( server lưu trữ video CDN jsDelivr ) để xem, không cache video vào Service
            Worker vì đã sử dụng CDN (  Content Delivery Network )

            - Service Worker chỉ cache các dữ liệu tĩnh ( html, css,... ) và không cache dữ liệu động ( video,... ) -> Nhiều video
            cũng không gây tốn cache vì chỉ cache dữ liệu tĩnh có dung lượng nhỏ

            - Note 1: Khi lưu trữ video bằng CDN thì không tính băng thông cho Github Page. Do đó, sau khi thoát ra, và khi truy cập
            lại sẽ lại tải từ server để xem theo trình tự như vậy

            - Note 2: Tuy nhiên CDN jsDelivr có cơ chế cache HTTP 7 ngày, Nên do đó có thể tận dụng và xem lại video từ cache HTTP mà
            không cần tải lại từ server CDN cho đến khi cache hết hạn

            - Note 3: Trong videos.json có thể dùng 3 dạng link gồm ( link CDN jsDelivr, link base github chứa /blob , link raw github ).
            Có logic xử lý để tự động chuyển đổi ánh xạ link base & link raw sang link CDN để tải dữ liệu băng thông không giới hạn

            - Note 4: Khi đã upload video lên github, muốn thay đổi video mới. THÌ Xóa video cũ + Add video mới ( phải trùng tên ) + Commit
            -> tiếp tục Purge Cache tại (  https://www.jsdelivr.com/tools/purge  ) -> Update version -> Chờ khoảng 5 - 10p -> Done

            - Note 5: Sử dụng ( baseConfig & hostname ) để cấu hình các BASE_URL dùng cho github page và cả các host khác như netlify,...

</br>

## 💻 Giải thích các BASE_URL:

    const REPOSITORY_ROOT: Đường dẫn gốc của dự án, dùng để đăng ký Service Worker.
    ( ví dụ xây dựng trên github có repository là tiktok-clone-v2 => thì REPOSITORY_ROOT = "/tiktok-clone-v2/" )

    const VIDEOS_JSON_URL: Đường dẫn đến file JSON chứa thông tin video như Title, URL, Desription, dayCreate
    ( ví dụ xây dựng trên github có repository là tiktok-clone-v2 => thì const VIDEOS_JSON_URL = `${REPOSITORY_ROOT}videos.json`; )

    const CACHE_NAME: Tên Cache của Worker Service, lưu ý cần cập nhật ở cả file html & script. ( const CACHE_NAME = "service-worker-v1" )


</br>

## ❌ Cách xóa Local Storage, Cache Service Worker, Cache HTTP:

    Settings -> Privacy and security -> Delete browsing data -> Cached images and files : Xóa Cache HTTP
    ( Cài đặt -> Quyền riêng tư và bảo mật -> Xóa dữ liệu duyệt web -> Tệp và hình ảnh được lưu trong bộ nhớ đệm )

    F12 -> Application -> Local storage : xóa Local Storage

    F12 -> Application -> Cache storage : xóa Cache Service Worker

</br>

## 😗 Một số lưu ý

</br>

    1. Xóa cache Service Worker

    Đổi CACHE_NAME trong sw.js
    -> Mục đích: Làm mới toàn bộ cache Service Worker bằng cách thay đổi tên cache, buộc tải lại tất cả tài nguyên.

    const CACHE_NAME = "service-worker-v1"; // Từ v1 thành v2

    2. Thay đổi version mỗi khi cập nhật video ( video.json )
    - Cập nhật cache khi thay đổi version
    - Link video mới thêm vào cache
    - Lướt tới video mới: Load và thêm vào Service Worker
    - Link URL bị xóa: Xóa video khỏi Service Worker

    tóm tắt:
    - Cập nhật version: Làm mới cache localStorage, tải videos.json mới.
    - Video mới: URL lưu vào localStorage, nội dung tải khi phát và lưu vào Service Worker.
    - Video bị xóa: URL không còn trong videos.json → Xóa video tương ứng khỏi cache Service Worker.
    - Băng thông: Tốn ít cho videos.json và HEAD request; video mới chỉ tải khi xem, video cũ dùng cache nếu còn trong danh sách.

    -> ( cập nhật version sẽ cập nhật cache, với link video mới sẽ thêm vào cache và khi lướt tới sẽ load và thêm vào service worker,
    còn link url nào bị xóa thì sẽ xóa video đó khỏi service worker )

    3. Cache hết hạn sau 24 giờ

    Điều kiện: Khi Date.now() >= parseInt(localStorage.getItem(CACHE_EXPIRY_KEY)), hàm isCacheValid() trả về false.

    Quy trình:

    - Bỏ qua cache localStorage vì hết hạn.
    - Fetch videos.json mới từ server (/tiktok-clone-v2/videos.json?v=<timestamp>).
    - Kiểm tra các URL video bằng fetch với method HEAD.
    - Lưu danh sách video mới và version vào localStorage, đặt lại thời hạn 24h.
    - Kiểm tra và xóa video cũ khỏi cache Service Worker (nếu URL không còn trong videos.json).
    - Khi lướt đến video, tải nội dung và lưu vào cache Service Worker nếu chưa có.

    Tác động:
    - Đảm bảo tải videos.json mới nếu cache hết hạn, nhưng không bắt buộc version phải thay đổi.
    - Video cũ vẫn tồn tại trong cache Service Worker nếu vẫn có trong videos.json.
    - Băng thông: Tốn cho videos.json và HEAD request; nội dung video tải khi xem (dùng cache nếu có).
