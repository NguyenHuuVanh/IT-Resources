# Tổng hợp tài liệu Twenty CRM (VI)

> Tài liệu này được tổng hợp từ các bản dịch trước trong cuộc trò chuyện, gộp lại thành một file Markdown để bạn lưu trữ và đọc nhanh.

## Mục lục

1. [What is Twenty](#what-is-twenty)
2. [Configure Your Workspace](#configure-your-workspace)
3. [Data Model](#data-model)
4. [Objects](#objects)
5. [Implementation Services](#implementation-services)
6. [Glossary](#glossary)
7. [Create a Workspace](#create-a-workspace)
8. [Navigate Around Twenty](#navigate-around-twenty)

---

## What is Twenty

**Twenty** là một **CRM mã nguồn mở (open-source)**, cung cấp các **“khối xây dựng” (building blocks)** để bạn tạo ra hệ thống CRM đúng với nhu cầu doanh nghiệp của mình.

### Tầm nhìn (Vision)

- Làm CRM tốt rất khó vì phải cân bằng: mỗi doanh nghiệp có yêu cầu “có vẻ đơn giản” nhưng **mỗi nơi lại khác nhau**.
- Thường CRM rơi vào 2 cực:
  - **Quá cơ bản** (thiếu tính năng bạn cần), hoặc
  - Cố làm “đủ thứ” (jack-of-all-trades) nhưng **không cái nào thực sự xuất sắc**.
- Twenty ban đầu giống CRM quen thuộc: theo dõi deal/cơ hội, quản lý liên hệ, tasks và notes.
- Điểm khác biệt: **khả năng mở rộng (extensibility)** — Twenty hướng tới **nền tảng mở** cung cấp các “mảnh ghép” để bạn giải bài toán riêng.
- Triết lý:
  - Ưu tiên **nguyên tắc chung** và **mô hình phổ biến** hơn “danh sách tính năng”.
  - Không áp đặt một đáp án; **trao quyền** cho người dùng tự tối ưu.
  - **Open-source** là nền tảng để Twenty phát triển **cùng cộng đồng và vì cộng đồng**.

### Lợi ích (Benefits)

- **Tùy biến cao**: khớp quy trình doanh nghiệp.
- **Cộng đồng dẫn dắt**: được xây dựng và duy trì bởi cộng đồng open-source lớn.
- **Tiết kiệm chi phí / không bị khóa nhà cung cấp (no vendor lock-in)**: luôn có thể **self-host**.

### Tính năng chính (Main Features)

- **Calendar & Emails**: sync email + lịch; xem toàn bộ trao đổi trên record CRM.
- **Data Model**: tạo custom objects/fields phù hợp quy trình riêng.
- **Data Migration**: import/export dữ liệu qua CSV hoặc API.
- **Views & Pipelines**: table views, kanban boards, sales pipelines.
- **Workflows**: tự động hóa quy trình và tích hợp công cụ ngoài.
- **AI**: tăng cường CRM với tính năng/agent AI.
- **Dashboards**: báo cáo và trực quan hóa theo nhu cầu.
- **Permissions & Access**: phân quyền theo vai trò.
- **Notes & Tasks**: ghi chú/công việc gắn với record để cộng tác.
- **API & Webhooks**: tích hợp hệ thống khác, build integration.

---

## Configure Your Workspace

Mỗi doanh nghiệp vận hành khác nhau. Bắt đầu với **3 bước** để “đo ni đóng giày” Twenty.

**Quick Win:** Kết nối **mailbox** trước (Settings → Accounts) để có dữ liệu thật và giá trị thấy ngay. Điều này giúp team của bạn thấy Twenty hoạt động với dữ liệu thực tế ngay lập tức.

### 1) Tùy biến mô hình dữ liệu (Customize your data model)

Twenty cung cấp sự linh hoạt để bạn tạo mô hình dữ liệu phù hợp nhất với công việc hàng ngày. Bạn có thể tạo objects và fields thuộc bất kỳ loại nào, bao gồm cả các mối quan hệ (relations) giữa các objects khác nhau.

**Thực hiện tại:** Settings → Data Model

**Các điểm quan trọng:**

- **Không giới hạn:** Bạn không bị giới hạn số lượng custom fields hay custom objects. Việc thêm custom objects và fields sẽ **không dẫn đến việc phải nâng cấp gói** của bạn.

- **Tận dụng 3 objects chuẩn:** People, Companies và Opportunities là ba objects mà từ đó bạn có thể truy cập emails và meetings được đồng bộ từ mailbox và calendar của mình. Chúng tôi khuyến nghị sử dụng chúng càng nhiều càng tốt, thêm các fields để phân loại records nếu cần.

**Ví dụ thiết kế:**

- Tốt nhất là sử dụng object **People** cho cả prospects và partners của bạn, tạo một field trên object People có tên **Person Type** để phân loại, thay vì tạo một custom object Partner riêng.
- **Lý do:** Nếu tạo object Partner riêng, bạn sẽ không thể truy cập các emails trao đổi với người này từ các records Partner.
- **Giải pháp:** Tạo các views khác nhau trong People - một view để hiển thị partners và một view để hiển thị prospects.

**Ràng buộc dữ liệu quan trọng:**

- Hai People **không thể có cùng địa chỉ email**
- Hai Companies **không thể có cùng domain**

**Tùy chỉnh thêm:**

- Bạn có thể **deactivate** (vô hiệu hóa) các standard fields và objects mà bạn không muốn sử dụng
- Bạn có thể **ẩn fields khỏi views**: đừng ngại tạo nhiều fields, bạn không cần phải hiển thị tất cả chúng

**Đọc thêm:** Xem bài viết về cách thiết kế data model của bạn để hiểu rõ hơn.

### 2) Đưa dữ liệu vào Twenty (Bring your data in)

Việc đưa dữ liệu hiện có của bạn vào Twenty giúp team có ngữ cảnh ngay từ đầu.

#### a) Kết nối mailbox của bạn

Nếu bạn chưa thực hiện khi tạo workspace, hãy kết nối tài khoản Google hoặc Microsoft của bạn tại **Settings → Accounts**. Điều này cho phép Twenty:

- **Import messages và meetings** của bạn
- **Tự động tạo contacts** dựa trên các tương tác (tùy chọn)
- Giữ **lịch sử giao tiếp hiển thị** cho team của bạn

**Sử dụng nhà cung cấp khác?**

- Bạn có thể thêm mailbox khác qua **SMTP**
- Hoặc thêm calendar khác qua **CalDAV**
- Vào **Settings → Accounts** để cấu hình

#### b) Import dữ liệu qua CSV

Sử dụng **Command menu** (Cmd + K hoặc Ctrl + K) để import People, Companies, Opportunities, hoặc bất kỳ custom objects nào qua CSV.

**Hướng dẫn quan trọng:**

1. **Download file mẫu** để hiểu format mong đợi
2. **Giới hạn mỗi file tối đa 10,000 records**
3. **Xóa các email trùng lặp** cho People hoặc **domain trùng lặp** cho Companies
4. **Xem xét và sửa lỗi** (được highlight màu vàng) trước khi import

**Đọc thêm:** Xem bài viết về data import để tìm hiểu chi tiết hơn.

### 3) Tạo view đầu tiên của bạn (Create your first view)

Tạo các views khác nhau là chìa khóa để làm cho dữ liệu có thể hành động được (actionable) cho team của bạn. Đây là cách thực hiện:

#### Thêm hoặc ẩn cột (Add or hide columns)

Quản lý các fields hiển thị trong một view bằng cách click vào **Options → Fields** (từ góc trên bên phải). Bạn có thể show/hide các fields từ đó.

#### Sắp xếp lại fields (Reorder fields)

Sắp xếp lại các fields từ một view bằng cách click vào **Options → Fields** (từ góc trên bên phải). Kéo và thả các fields để sắp xếp lại chúng.

#### Lọc view của bạn (Filter your view)

Thu hẹp các records được hiển thị bằng cách sử dụng **Filters** từ góc trên bên phải.

#### Sắp xếp records (Sort records)

Sắp xếp lại các records được hiển thị bằng cách sử dụng chức năng **Sort** từ góc trên bên phải, hoặc bằng cách click trực tiếp vào tên cột.

#### Chọn layout (Choose the layout)

Bạn có thể chuyển sang **Kanban layout** hoặc **list Group By layout**, miễn là object có field Stage hoặc field kiểu select tương tự.

#### Lưu view của bạn vào Favorites (Save your view as Favorites)

Điều này có thể được thực hiện bằng cách sử dụng dropdown menu hiển thị các views khác nhau.

**Mẹo:** Việc tạo và tổ chức views hiệu quả sẽ giúp team của bạn làm việc nhanh hơn và tập trung vào đúng dữ liệu cần thiết.

---

## Data Model

Tìm hiểu Data Model là gì và cách thiết kế một mô hình phù hợp với doanh nghiệp của bạn.

### Data Model là gì?

Data model (mô hình dữ liệu) là cấu trúc định nghĩa cách thông tin được tổ chức trong CRM của bạn. Hãy nghĩ về nó như **bản thiết kế** của dữ liệu khách hàng — bạn thiết kế nó một lần, sau đó điền vào đó dữ liệu thực tế của bạn.

### Các khái niệm chính

#### Objects (Đối tượng)

Objects là các danh mục dữ liệu chính trong CRM của bạn. Mỗi object đại diện cho một loại thứ mà bạn muốn theo dõi.

**Twenty đi kèm với các standard objects:**

- **People** — cá nhân (contacts, leads, partners)
- **Companies** — tổ chức
- **Opportunities** — deals hoặc sales
- **Notes** — ghi chú đính kèm trên records
- **Tasks** — công việc cần làm liên kết với records

Bạn cũng có thể tạo **custom objects** cho bất kỳ thứ gì cụ thể với doanh nghiệp của bạn (ví dụ: Projects, Subscriptions, Events).

#### Fields (Trường dữ liệu)

Fields là các thuộc tính hoặc đặc điểm mô tả từng object. Chúng lưu trữ thông tin thực tế.

**Ví dụ:** object People có các fields như:

- Name (Tên)
- Email
- Phone (Điện thoại)
- Job Title (Chức danh)
- Company (một relation tới object Companies)

Fields có nhiều loại khác nhau: text, number, date, select, multi-select, relation, và nhiều hơn nữa. Bạn có thể thêm custom fields vào bất kỳ object nào.

#### Records (Bản ghi)

Records là các mục nhập riêng lẻ trong một object — dữ liệu thực tế mà bạn tạo và quản lý.

**Ví dụ:**

- "John Smith" là một record trong object People
- "Acme Corp" là một record trong object Companies

**Một phép tương tự:**

| Khái niệm Data Model | Tương tự thực tế                                |
| -------------------- | ----------------------------------------------- |
| Objects              | Các phần trong một cuốn sách (các danh mục)     |
| Fields               | Các cột trong bảng tính (các thuộc tính)        |
| Records              | Các hàng trong bảng tính (các mục nhập thực tế) |

Bạn thiết kế data model (objects + fields) một lần, sau đó tạo nhiều records trong cấu trúc đó.

### Tại sao cần tùy biến Data Model?

Mỗi doanh nghiệp hoạt động khác nhau. Tùy biến data model có nghĩa là bạn có thể định hình Twenty xung quanh quy trình của mình thay vì ép quy trình của bạn vào một hệ thống cứng nhắc.

**Twenty cung cấp sự linh hoạt hoàn toàn:**

- Tạo bao nhiêu custom objects tùy thích
- Thêm unlimited custom fields
- Giá không thay đổi dựa trên việc tùy biến

### Mẹo thiết kế Data Model của bạn

#### 1. Bắt đầu với các Core Objects

Xác định các khái niệm chính mà bạn làm việc cùng. Twenty đã cung cấp sẵn:

- **People** — contacts của bạn
- **Companies** — accounts của bạn
- **Opportunities** — deals của bạn

Nghĩ về những gì khác bạn có thể cần:

- Stripe sẽ cần một object **Subscriptions**
- Airbnb sẽ cần một object **Trips**
- Một accelerator sẽ cần một object **Batches**

#### 2. Sử dụng Fields cho các biến thể, không phải Objects mới

Nếu một thứ gì đó chỉ là một đặc điểm của object hiện có, hãy biến nó thành một field.

**Sử dụng fields cho:**

- Categories và labels (ví dụ: Industry cho Companies)
- Status values (ví dụ: Stage cho Opportunities)
- Attributes và properties

#### 3. Tạo Object khi nó tự đứng vững

Nếu khái niệm đó có vòng đời riêng, thuộc tính riêng, hoặc mối quan hệ riêng, nó xứng đáng có một object.

**Tạo object cho:**

- **Projects** — có deadlines, owners, và tasks
- **Subscriptions** — kết nối companies, products, và invoices
- **Events** — liên quan đến attendees và follow-up actions

Những thứ này vượt ra ngoài một field đơn lẻ vì chúng mang dữ liệu và mối quan hệ riêng của chúng.

#### 4. Tạo Object khi Records là không giới hạn (Open-Ended)

Nếu một thứ gì đó có thể được liên kết nhiều lần và bạn không biết bao nhiêu, hãy sử dụng object.

**Cách tiếp cận tồi:** Tạo các fields như Product 1, Product 2, Product 3…

**Cách tiếp cận tốt:** Tạo một object Products và liên kết nó với records. Điều này hỗ trợ một, hai, hoặc hàng trăm products mà không cần thay đổi model của bạn.

#### 5. Giữ nó đơn giản trước

Bắt đầu với fields. Chỉ chuyển sang objects mới khi bạn cảm thấy giới hạn:

- Quá nhiều fields trên một object
- Các records lặp lại nên được tách riêng
- Các mối quan hệ không khớp gọn gàng

### Lưu ý đặc biệt về People, Companies, và Opportunities

**Email và calendar sync chỉ hoạt động với People, Companies, và Opportunities.**

Đây là những objects duy nhất mà bạn có thể truy cập emails và meetings được đồng bộ từ mailbox/calendar của mình. Chúng tôi khuyến nghị sử dụng chúng càng nhiều càng tốt.

**Best practices:**

- Nếu bạn cần các categories của People, hãy sử dụng fields (không phải objects mới)
- **Ví dụ:** Sử dụng field **Person Type** với các giá trị "Prospect" và "Partner" thay vì tạo các objects riêng biệt
- Tạo các views khác nhau để lọc: một view hiển thị partners, một view khác hiển thị prospects
- Không sao nếu có các fields không áp dụng cho mọi record. Ví dụ: một field **Referral Link** trên People chỉ áp dụng khi Person Type = Partner. Ẩn field này khỏi các views mà nó không liên quan.

### Câu hỏi để hướng dẫn lựa chọn của bạn

Tự hỏi bản thân:

- Đây chỉ là một thuộc tính của thứ tôi đã có, hay nó cần các thuộc tính riêng của nó?
- Tôi có bao giờ cần theo dõi nhiều cái này cho mỗi record, mà không biết bao nhiêu không?
- Khái niệm này có kết nối với nhiều objects khác nhau, không chỉ một không?
- Nó có vòng đời riêng không (stages, start/end dates)?

Nếu câu trả lời là "có" cho một hoặc nhiều câu hỏi, có lẽ đã đến lúc tạo một object mới.

### Truy cập Data Model của bạn

1. Vào **Settings** ở sidebar bên trái
2. Click **Data Model**
3. Xem tất cả objects của bạn (standard và custom)
4. Click vào bất kỳ object nào để xem và chỉnh sửa các fields của nó

**Không thấy Data Model trong Settings?**

Quyền truy cập vào data model thường bị giới hạn cho administrators. Liên hệ workspace admin của bạn nếu bạn cần quyền truy cập.

---

## Objects

Tìm hiểu về standard objects và custom objects trong Twenty.

### Standard Objects

Standard objects là các entities được định nghĩa sẵn trong workspace của bạn để giúp bạn bắt đầu. Chúng là một phần của data model được chia sẻ, có thể truy cập bởi tất cả người dùng Twenty. Bạn có thể sử dụng chúng nguyên bản, tùy biến chúng hoặc vô hiệu hóa chúng.

#### People

Object **People** lưu trữ contacts của bạn. Nó bao gồm thông tin chi tiết liên hệ và lịch sử tương tác, vì vậy bạn có thể xem tất cả các tương tác với khách hàng ở một nơi.

#### Company

Object **Companies** lưu trữ các business accounts của bạn. Nó bao gồm các chi tiết như industry (ngành), size (quy mô) và location (vị trí). Companies kết nối với cả objects People và Opportunities.

#### Opportunities

Object **Opportunities** lưu trữ dữ liệu liên quan đến deal. Nó theo dõi tiến trình của các potential sales, từ prospecting đến closure, ghi lại các stages, deal sizes, associated account, và expected close date. Bạn có thể xem sales pipeline của mình trong kanban layout.

#### Notes

Object **Notes** lưu trữ các ghi chú tự do có thể được đính kèm vào People, Companies, Opportunities, và các records khác. Sử dụng notes để ghi lại meeting summaries, important details, hoặc bất kỳ contextual information nào.

#### Tasks

Object **Tasks** lưu trữ các to-dos và action items. Tasks có thể được liên kết với People, Companies, Opportunities, và các records khác. Theo dõi due dates, assignees, và completion status để luôn nắm bắt các follow-ups của bạn.

### Custom Objects

Custom objects cho phép bạn lưu trữ thông tin độc đáo với tổ chức của bạn và mà standard objects không thể xử lý. Ví dụ, nếu bạn là SpaceX, bạn có thể muốn tạo một custom object cho Rockets và Launches.

#### Tạo Custom Object mới

Để tạo một custom object mới:

1. Vào **Settings** ở sidebar bên trái
2. Trong **Workspace**, vào **Data model**. Tại đây bạn sẽ có thể thấy tổng quan về tất cả Standard và Custom objects hiện có của bạn (cả active và disabled)
3. Click vào **+ New object** ở trên cùng
4. Nhập tên (cả singular và plural), chọn một icon, và thêm description cho custom object của bạn
5. Nhấn **Save** (ở góc trên bên phải)

**Ví dụ:** Sử dụng Listing làm ví dụ về custom object:

- Singular: "listing"
- Plural: "listings"
- Description: "Listings that hosts created to showcase their property."

Custom object của bạn giờ đã được tạo và sẽ xuất hiện trong sidebar của bạn. Bạn có thể bắt đầu thêm records vào nó ngay lập tức.

### Quản lý Objects

#### Vô hiệu hóa Objects (Deactivating Objects)

Nếu bạn không cần một standard hoặc custom object:

1. Vào **Settings → Data Model**
2. Tìm object bạn muốn vô hiệu hóa
3. Click vào toggle để deactivate nó
4. Object sẽ bị ẩn khỏi workspace của bạn nhưng dữ liệu được bảo toàn

#### Kích hoạt lại Objects (Reactivating Objects)

Để khôi phục lại một object đã bị vô hiệu hóa:

1. Vào **Settings → Data Model**
2. Tìm các deactivated objects (chúng sẽ bị làm mờ - grayed out)
3. Click vào toggle để reactivate nó
4. Object và tất cả dữ liệu của nó sẽ được khôi phục

### Best Practices

#### Khi nào nên tạo Custom Objects

**Unique business entities:** Những thứ cụ thể với ngành hoặc quy trình của bạn

**Complex relationships:** Khi bạn cần theo dõi các kết nối giữa nhiều entities

**Scalable data:** Khi bạn có thể có nhiều instances của một thứ gì đó

#### Khi nào nên sử dụng Fields thay thế

**Simple attributes:** Các thuộc tính mô tả existing objects

**Categories hoặc labels:** Các cách để phân loại existing records

**Single values:** Thông tin không cần vòng đời riêng của nó

#### Đặt tên Object (Object Naming)

**Sử dụng tên rõ ràng, mô tả:** Làm cho nó rõ ràng object đại diện cho cái gì

**Tuân theo quy ước:** Sử dụng singular cho tên object, plural cho collection

**Xem xét team của bạn:** Chọn tên mà mọi người sẽ hiểu

---

## Implementation Services

Dịch vụ hỗ trợ triển khai, từ onboarding đến tùy biến nâng cao.

### Onboarding Packs (4 giờ)

Hỗ trợ từ core team để setup workspace:

- **Data Model Design**: thiết kế và tạo data model (objects/fields/relationships).
- **Data Migration**: chuyển dữ liệu từ CRM hiện tại sang Twenty.
- **Workflow Creation**: tạo workflows tùy chỉnh cho quy trình nghiệp vụ.

### Implementation Partners

- Làm việc với đối tác chứng nhận để tùy biến/tích hợp nâng cao.
- Liên hệ team Twenty qua **contact@twenty.com** để được ghép đối tác.

---

## Glossary

Thuật ngữ thiết yếu trong Twenty:

- **API**: Kết nối Twenty với hệ thống khác; xây integration.
- **Apps**: Extension viết bằng code; định nghĩa data model & logic; tái sử dụng cho nhiều workspace.
- **Code Actions**: Bước workflow cho phép viết JavaScript xử lý logic phức tạp/biến đổi dữ liệu.
- **Command Menu**: Menu thao tác nhanh (Cmd+K / Ctrl+K) để tạo record, điều hướng, chạy hành động.
- **Company & People**:
  - Company = doanh nghiệp/tổ chức
  - People = khách hàng hiện tại/tiềm năng
- **Custom Fields**: Trường dữ liệu bạn tạo thêm cho nhu cầu riêng.
- **Data Model**: Cấu trúc dữ liệu: objects, fields, relations.
- **Favorites**: Đánh dấu record/view để truy cập nhanh từ sidebar.
- **Field**: Thuộc tính/ô dữ liệu của entity (email, amount, close date…).
- **Iterator**: Action workflow lặp qua một array, chạy các action cho từng phần tử.
- **Kanban**: Bảng cột + thẻ; mỗi cột là stage; kéo record qua các stage.
- **Object**: Loại entity dữ liệu (People/Companies/Opportunities…); có standard hoặc custom.
- **Opportunities**: Cơ hội bán hàng/deal tiềm năng với account/contact.
- **Record**: Một bản ghi cụ thể của object.
- **Relation Fields**: Field tạo liên kết giữa objects (vd: Person ↔ Company).
- **Standard Fields**: Field có sẵn theo mặc định.
- **Tasks**: Hoạt động được giao liên quan contact/account/opportunity.
- **Triggers**: Điểm khởi động workflow (create/update record, webhook, schedule…).
- **Views**: Cách hiển thị records (filter/layout/sort) theo từng view.
- **Upsert**: Update nếu có bản ghi khớp, không có thì Insert.
- **Webhooks**: Thông báo tự động từ Twenty sang hệ thống khác khi có event để sync real-time.
- **Workflows**: Tự động hóa quy trình bằng trigger + điều kiện + actions.
- **Workspace**: Không gian dữ liệu của một công ty dùng Twenty; có domain workspace.
- **Workspace Members**: Thành viên có quyền truy cập workspace; có thể được gán owner/assignee.

---

## Create a Workspace

Hướng dẫn tạo workspace, chọn trial và setup tài khoản.

### Step 1: Registration

- Truy cập **Twenty Sign Up**.
- Chọn cách đăng ký:
  - Continue with Google
  - Continue with Microsoft
  - Continue with Email

### Step 2: Choosing a Trial Period

Chọn 1 trong 2 trial:

- **30 days**: có **credit card**
- **7 days**: không cần credit card

Cả hai trial đều gồm:

- Full access
- Unlimited contacts
- Email integration
- Custom objects
- API & Webhooks

Bạn có thể bấm **Change plan** để đổi gói hoặc chu kỳ thanh toán.

### Step 3: Payment Confirmation & Account Setup

- Sau khi thanh toán được duyệt qua **Stripe**, bạn được chuyển tới bước tạo:
  - Workspace
  - User profile
- Có thể hủy subscription bất kỳ lúc nào.

---

## Navigate Around Twenty

Tổng quan điều hướng và nơi thực hiện các thao tác.

### The Main Layout

- Khu vực giữa màn hình là nơi records “sống”: People, Companies, Opportunities, Tasks, Notes, Dashboards, Workflows và objects khác.
- Tại đây bạn có thể xem/sửa/xóa records và tạo views.

### The Navigation Bar (sidebar trái)

- Chuyển workspace hoặc tạo workspace mới
- Search bar (nhấn **/** để focus nhanh)
- Mở **Settings**
- Truy cập nhanh **Favourite views** (riêng cho từng user)
- Chuyển objects
- Tạo automations bằng **Workflows**

### The Command Menu

Mở bằng:

- Cmd+K (Mac) / Ctrl+K (Windows)
- hoặc bấm **ba chấm** góc phải trên

Từ đây bạn có thể:

- Tạo records
- Import/export CSV
- Tạo views
- Truy cập deleted records (soft delete & hard delete)
- Xem keyboard shortcuts

### The Search Bar

- Truy cập qua Command Menu, trên navigation bar, hoặc nhấn **/**.
- Search hoạt động xuyên suốt tất cả objects.

### The Side Panel

- Click 1 record → side panel xuất hiện bên phải để xem nhanh key info.
- Có thể đóng panel hoặc bấm **Open** để vào trang chi tiết.

### Views

- Mỗi object có nhiều views, **không giới hạn** số view.
- Dùng dropdown góc trái trên main layout để đổi view.
- Gợi ý:
  - Kanban view để track opportunities theo stage
  - Group By view để chia section
  - Filters để tập trung nhóm records (vd: lead tạo tuần trước)
  - Lưu filtered views và đánh dấu Favourite để truy cập nhanh

### Settings

Mở Settings để:

- Kết nối mailbox & calendar để sync
- Tùy biến data model (objects/fields/relationships)
- API playground + webhooks
- Quản lý permissions & access
- Mời thành viên và quản lý roles
- Chỉnh profile & workspace preferences
- Billing + theo dõi workflow credits
- Bật tính năng mới (Updates → Early Access)
- Support (live chat)
- Documentation (user guide + developer docs)

> Nếu không thấy đủ mục Settings, hãy liên hệ workspace admin (có thể bị giới hạn quyền).
