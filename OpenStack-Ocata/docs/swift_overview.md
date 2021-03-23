# Tổng quan về Swift

## Mục lục.
- [1. Kiến trúc logic](#1)
- [2. Kiến trúc vật lý](#2)
- [3. Tổng quan về kiến trúc phần mềm trong Swift](#3)
- [4. Kiến trúc xác thực trong Swift](#4)
- [5. Mô hình triển khai](#5)
- [6. Xử lý data](#6)
- [7. Storage policies](#7)

<a name=1></a>
# 1. Kiến trúc logic
- Tổ chức logic.

	![](../images/swift_logic.png)

	
- Mỗi tenant được gán một **Account**. 
- Trong account, có thể chứa nhiều **container** (tương tự như folder).
- Container sẽ chứa các **Object**.
- Người dùng lưu trữ các đối tượng mà không phải quan tâm đến phần cứng.
- Vấn đề đặt tên: các account phải có tên khác nhau và là duy nhất. Trong một account, tên của các containers cũng phải khác nhau, nhưng trên 2 account khác nhau thì tên container có thể trùng nhau. Với object cũng tương tự như thế.
- Trong lưu trữ swift, các containers là ngang hàng nhau. Như vậy sẽ không có tạo subcontainer. Tuy nhiên, swift hỗ trợ cơ chế giả thư mục để hỗ trợ cho việc quản lý.

### Chi tiết về từng thành phần
## 1. account
- Account là gốc của vị trí lưu trữ data. 
- Thường so sánh với filesystem volume, account là tên duy nhất trong hệ thống.
- Mỗi account có một database lưu trữ metada của bản thân nó và cả danh sách các containers ở trong account đó.
- Metada headers của account được lưu với cặp key-value đảm bảo lưu trữ lâu dài cùng với tất cả dữ liệu trong swift.
- Cơ sở dữ liệu account không chứa các objects và metada của object.
- Cơ sở dữ liệu của account được truy cập khi người dùng (người được cho phép hoặc services) yêu cầu metada về một account hoặc danh sách containers trong account.
- Account có thể được chỉ định cho đơn người dùng truy cập hoặc nhiều người dùng truy cập tùy thuộc vào quyền mà nó được thiết lập.
- Đơn người dùng với người dùng các nhân hoặc các dịch vụ accounts như là backups, đa người dùng truy cập với project nơi nhiều thành viên của nhóm truy cập account.
- Người dùng swift với các quyền chính xác có thể tạo, thay đổi, xóa các container và các objects trong container với một account.
- Trong swift, account là lưu trữ account, không phải là định danh user.

## 2. container
- Container được người dùng định nghĩa phân đoạn của không gian tên account mà cung cấp vị trí lưu trữ nơi các objects được tìm thấy.
- Mặc dù các container không thể lồng vào nhau, nhưng có khái niệm tương tự như thư mục trong filesystem.
- Container không có ràng buộc về tên duy nhất.
- Không có giới hạn về số lượng container trong một account.
- Cũng như cơ sở dữ liệu account, mỗi container database chứa metada của container và danh sách các object trong container đó.
- Và metada headers cũng được lưu dưới dạng key-value.
- Database container không chứa objects thực và metada của object.
- Database container được truy cập khi user yêu cầu danh sách tất cả các object trong container hoặc metada về container đó.
- Trong một account, bạn có thể tạo và sử dụng các containers để nhóm dữ liệu theo cách logic. Bạn cũng có thể gán metada cho containers, và sử dụng vài tính năng Swift để cho các thuộc tính containers khác nhau.

## 3. Objects.
- Object là dữ liệu được lưu trong Openstack Swift. Chúng có thể là ảnh, videos, tài liệu, logfiles, database backups, fileysystem snapshots, hoặc bất kỳ dữ liệu khác. Mỗi đối tượng thường bao gồm bản thân dữ liệu của nó và metada.
- Metada headers được lưu với cặp key-value, đảm bảo lâu dài với dữ liệu object, và đòi hỏi không thêm chút độ trễ để khôi phục. Metada có thể cung cấp các thông tin quan trọng về object. Ví dụ, một nhà sản xuất video có thể lưu định dạng video và khoảng thời gian dọc theo nội dụng; một tài liệu có thể chứa thông tin tác giả.
- Mỗi object phải thuộc một container. Khi một object được lưu trong cụm, người dugnf sẽ luôn để ý đến vị trí lưu trữ object (/account/container/object). Không có giới hạn số lượng object mà user có thể lưu trong container.
- Vị trí lưu trữ object là nơi để tìm object. Swift sẽ lưu nhiều bản sao của object để đảm bảo sự tin cậy và độ khả dụng của dữ liệu. Swift làm điều này bằng cách đặt đối tượng trong nhóm logic gọi là partition. Một partition sẽ liên kết đến nhiều thiết bị, mỗi thiết bị sẽ có object được ghi vào. User truy cập object thông qua vị trí của nó. User không biết và không cần biết nơi nào trong hệ thống dữ liệu được đặt.

<a name=2></a>
# 2. Kiến trúc vật lý
- Tổ chức vật lý.

	![](../images/swift_physical.png)

- Hệ thống phân cấp trong tổ chức vật lý của Swift như sau:
	- Region: Tại mức cao nhất, Swift lưu trữ dữ liệu trong các regions (gọi là khu vực) bị chia cắt về mặt vật lý - phân chia theo địa lý.
	- Zone: Bên trong các regions là các zones. Zone là tập các máy chủ riêng biệt, các máy chủ này được gọi là node storage. Khi có máy chủ xảy ra lỗi, Swift sẽ tách biệt các zone có máy chủ bị lỗi đó với các zone khác.
	- Storage server (máy chủ lưu trữ): Trong các zones sẽ chứa các máy chủ để lưu trữ.
	- Disk (ổ đĩa hoặc là thiết bị lưu trữ): Là một phần của storage server, có thể là các đĩa cứng cắm trực tiếp bên trong storage server hoặc được liên kết qua mạng.
	
<a name=3></a>
# 3. Tổng quan về kiến trúc phần mềm trong Swift.
- Trong Swift có 4 service chính. Document của swift gọi là Servers:
	- Proxy server
	- Account server
	- Container server
	- Object server
	
- Thông thường, proxy server được cài đặt trên một node riêng, còn 3 server còn lại sẽ được cài đặt trên cùng một node gọi là storage server.

	![](../images/swift_servers.png)
	
- Chức năng chính của từng server:
	- Proxy server: Thịu trách nhiệm nhận các yêu cầu HTTP từ người dùng. Nó sẽ tìm kiếm vị trí của storage server nơi mà yêu cầu cần được chuyển tiếp bằng cách sử dụng ring phù hợp. Proxy server tính toán các node bị hư hại bằng cách tìm kiếm các handoff node và thực hiện đọc/ghi.
	
	- Account server: Theo dõi tên các containers có trong account. Dữ liệu được lưu trữ trong cơ sở dữ liệu SQL, các files cơ sở dữ liệu sẽ được lưu trong filesystem. Server này cũng theo dõi thống kê nhưng không có bất kỳ thông tin nào về vị trí của các containers. Proxy server xác định vị trí container dựa trên container ring. Thông thường, account server sẽ được cài đặt với container server và object server trên cùng một máy chủ vật lý. Tuy nhiên, với một hệ thống lớn thì có thể cài đặt các server trên các máy chủ riêng biệt.
	
	- Container server: Giống như account server, ngoại trừ container server quản lý các objects bên trong nó.
	
	- Object server: Đơn giản là server quản lý lưu trữ đối tượng. Trên các ổ đĩa có các filesystem, và các đối tượng được lưu trên đó.
	
<a name=4></a>
# 4. Kiến trúc xác thực trong Swift
- Xác thực là một phần quan trọng trong swift. 
- Các thành phần trong swift đều dạng module, do vậy thành phần xác thực cũng là một project riêng.
- Người dùng có thể lựa chọn các hệ thống xác thực khác nhau. Keystone là project xác thực ở trong Openstack mà có thể được sử dụng cho Swift.

	![](../images/swift_keystone.png)
	
- Luồng làm việc của Swift với keystone.
	- 1. Người dùng cung cấp thông tin xác thực cho hệ thống xác thực. Điều này được thực hiện bằng cách thực hiện thông qua lời gọi HTTP REST API.
	- 2. Hệ thống xác thực cung cấp cho người dùng một mã (token) thông báo AUTH.
	- 3. Mã thông báo AUTH không phải là duy nhất cho mọi yêu cầu và hết hạn sau một khoảng thời gian nhất định. Mọi yêu cầu được gửi cho Swift đều phải kèm theo token.
	- 4. Swift kiểm tra mã thông báo với hệ thống xác thực và lưu trữ kết quả vào bộ nhớ đệm. Kết quả được làm sạch khi hết hạn.
	- 5. Thông thường, hệ thống xác thực có khái niệm tài khoản quản trị và không quản trị. Các yêu cầu của quản trị viên rõ ràng sẽ được thông qua.
	- 6. Các yêu cầu không quản trị được kiểm tra dựa vào danh sách kiểm soát truy cập mức kho chứa (ACL – access control lists). Các danh sách này cho phép quản trị viên thiết lập các quyền đọc ghi cho người dùng không phải quản trị.
	- 7. Như vậy, với các yêu cầu không phải quản trị, ACL được kiểm tra trước khi proxy server có các bước xử lý tiếp theo với yêu cầu này.
	
<a name=5></a>
# 5. Mô hình triển khai
- Với hệ thống không yêu cầu đặc biệt về hiệu năng, proxy server sẽ được cài đặt trên một server riêng. Account server, container server và object server sẽ được cài trên cùng một server, và server đó được gọi là storage server.

	![](../images/swift_modeinstall.png)
	
<a name=6></a>
# Tổng quan các quá trình xử lý object

# Các thao tác
## Tạo mới object
- Một yêu cầu tạo được gửi thông qua lời gọi HTTP PUT API đến một proxy server. 
- Trong hệ thống có thể có nhiều proxy server, và không quan trọng proxy server nào nhận được yêu cầu bởi vì Swift là một hệ thống phân phối, do đó các proxy server có vai trò như nhau. 
- Proxy server tương tác với Ring liên kết với chính sách của container để lấy danh sách disk và các object servers liên kết với container đó để ghi dữ liệu vào. 
- Disk để ghi có thể là duy nhất. Nếu disk này bị hưng hại, hoặc không sẵn sàng để sử dụng thì ring sẽ cung cấp các thiết bị handoff. Một khi đa số các disk đã xác nhận là đã ghi dữ liệu vào (ví dụ, hai trong ba bản sao đã được ghi vào disk) thì thao tác ghi đã được thực hiện thành công. Giả sử bản sao thứ 3 vẫn được ghi thành công. Nếu không, quá trình sao chép, được minh họa trong hình dưới đây, để đảm bảo rằng bản sao còn lại cuối cùng cũng được tạo ra:
	
	![](../images/swift_create.png)
	
- Thao tác tạo này có thể hơi khác đối với hệ thống đa vùng (multi-region). Tất cả các bản sao của object sẽ được ghi vào local region. Sau đó chúng sẽ được chuyển không đồng bộ đến một hoặc nhiều region khác. Một mạng chuyên dụng sẽ được sử dụng để thực hiện công việc này.

## Thao tác đọc object:
- Yêu cầu đọc được gửi đi thông qua HTTP Get API sẽ gọi đến proxy server. 
- Bất kỳ proxy server nào cũng có thể lại nhận yêu cầu này một lần nữa. 
- Tương tự như thao tác tạo, proxy server tương tác với ring thích hợp để có được danh sách các disk và các object servers liên quan. Yêu cầu đọc này sẽ được đưa đến các object server trong region với proxy server. Đối với hệ thống đa vùng, xuất hiện vấn đề về sự nhất quán, vì các region khác nhau có thể lưu các phiên bản khác nhau của object. Để giải quyết vấn đề này, Swift sử dụng một trường là `timestamp` để đánh dấu thời gian trên object, đảm bảo object được đọc là phiên bản mới nhất. 
- Khi có yêu cầu đọc, proxy server sẽ yêu cầu các object server cung cấp timestamp. Sau đó dựa vào timestamp để xác định phiên bản mới nhất và thực hiện đọc từ object server chứa phiên bản mới nhất của object. Cũng giống như thao tác tạo, khi có thiết bị bị hư hỏng, thiết bị handoff có thể được yêu cầu.

## Thao tác update:
- Yêu cầu update về cơ bản được xử lý giống như một yêu cầu tạo. 
- Object được lưu lại cùng với timestamp của chúng để đảm bảo rằng khi đọc, phiên bản mới nhất của object được trả về. Swift cũng hỗ trợ tính năng version trên mỗi container. Khi tính năng này được bật, các phiên bản cũ của object cũng được tạo sẵn trong một kho chứa đặc biệt gọi là versions_container.

## Thao tác xóa:
- Yêu cầu xóa được gửi qua lời gọi HTTP DELETE API được xử lý như update, nhưng thay vì phiên bản mới, sẽ có một phiên bản "bia mộ" với kích thước 0-byte được thay thế. 
- Một hoạt động xóa là rất khó khăn trong một hệ thống phân phối, vì hệ thống sẽ tái tạo bản sao đã bị xóa để đảm bảo rằng một đối tượng có đúng số lượng bản sao. Các giải pháp Swift đã loại bỏ khả năng các đối tượng bị xóa đột nhiên xuất hiện trở lại.

# Các tiến trình xử lý
## Replication
- Replication là một khía cạnh rất quan trọng của Swift. Nó đảm bảo tính nhất quán cho hệ thống; đó là, tất cả các servers và disks được chỉ định bởi các ring để giữ phiên bản mới nhất của các bản sao object hoặc bản sao database. 
- Quá trình này nhằm đảm bảo tránh sự hỏng hóc dữ liệu, di chuyển phần cứng, và tái cân bằng vòng (đây là nơi vòng được thay đổi và dữ liệu phải được di chuyển xung quanh). Điều này được hoàn thành bằng cách so sánh dữ liệu cục bộ với các bản sao từ xa. Nếu các bản sao từ xa cần được cập nhật. Trình replication sẽ đẩy một bản sao để thay thế. 
- Quá trình so sánh hiệu quả và được thực hiện đơn giản bằng cách so sánh danh sách băm chứ không phải là mỗi byte của một object (hoặc database account, hoặc database container). 
- Replication sử dụng rsync, tiện ích đồng bộ hoá tệp tin từ xa trên Linux, để sao chép dữ liệu. Có một số phương pháp sẵn có như là ssync, và trong bản thân swift có thể thay thế lẫn nhau để thực hiện replicate.

## Updaters
- Trong một số tình huống nào đó, account server hoặc container server có thể bận do tải nặng hoặc không khả dụng. Trong trường hợp này, bản cập nhật được đưa vào hàng đợi lưu trữ cục bộ của máy chủ lưu trữ. Có 2 updaters xử lý các mục trong hàng đợi. Object updater sẽ cập nhật các đối tượng vào container database, trong khi đó container updater sẽ cập nhật các container vào account database.

## Auditor (trình kiểm toán)
- Auditor sẽ kiểm tra tính chân thực dữ liệu của mỗi object, container và account. 
- Việc kiểm tra này được thực hiện bằng cách tính giá trị băm MD5 và so sánh giá trị băm MD5 được lưu với khóa Etag trong metadata của object. 
- Khóa Etag được tạo ra khi object được tạo lần đầu tiên. 
- Nếu object được phát hiện là có sai sót, nó sẽ được đưa đến thư mục kiểm dịch, và trong thời gian này, quá trình replication sẽ tạo ra một bản sao “sạch”. Đây là cách hệ thống tự sửa chữa. 
- Giá trị băm MD5 cũng có sẵn cho người dùng, do vậy họ có thể thực hiện các thao tác như là so sánh giá trị băm trong cơ sở dữ liệu của họ với giá trị được lưu trong Swift.

<a name=7></a>
# Storage policies

- Swift sử dụng mã băm rings để xác định nơi dữ liệu sẽ được lưu. Có các ring riêng biệt cho cơ sở dữ liệu account, cho cơ sở dữ liệu container và có một object ring cho mỗi chính sách lưu trữ. Mỗi object ring đều có cách xử lý giống nhau, cùng được quản lý trong cùng một phương pháp, nhưng với các policies, các thiết bị khác nhau thuộc các ring khác nhau. Sau đây là một số lý do sử dụng các policies lưu trữ:
	- Chia các mức độ lâu dài của dữ liệu khác nhau: Nếu nhà cung cấp muốn cung cấp, ví dụ 2x hoặc 3x bản sao nhưng không muốn bảo trì ở trên 2 cụm riêng biệt, họ muốn thiết lập một chinh sách 2x, một chính sách 3x và chỉ định các nodes vào các ring tương ứng của họ.
	- Hiệu năng: ví dụ như SSDs có thể được sử dụng như các thành viên độc quyền của một account ring hoặc database ring, một SSD object ring chỉ được tạo ra và sử dụng để thực hiện các chính sách lưu trữ yêu cầu về hiện năng, có độ trễ thấp.
	
# Containers and Policies
- Các policies được thực hiện ở mức container. Nó đảm bảo rằng các chính sách lưu trữ vẫn còn là một tính năng cốt lõi của Swift, độc lập với xác thực. 
- Các chính sách không được triển khai ở tầng account hoặc auth bởi vì nó yêu cầu thay đổi tất cả cho hệ thống xác thức được sử dụng bởi người triển khai Swift. 
- Mỗi container có một thuộc tính mới metadata không thay đổi được được gọi là `storage policy index`. Chú ý rằng, bên trong swift sẽ dựa vào policy index chứ không phải tên policy. Tên policy là để cho con người có khả năng đọc và sự dịch (từ index <=> tên) được quản lý trong policy. 
- Khi một container được tạo, một tùy chọn header mới được hỗ trợ để chỉ định tên policy. Nếu không có tên được chỉ định thì policy mặc định được sử dụng (và nếu không có policy được đinh nghĩa, thì Policy-0 được coi là mặc định).
- Các policies sẽ được gán khi một container được tạo. Một khi container đã được ấn định policy thì sẽ không thể thay đổi (trừ khi nó bị xóa đi và tạo lại).
- Quan hệ giữa containers và policies là n-1 có nghĩa là nhiều container chia sẻ cùng 1 policy. Không có giới hạn về số lượng container trên một policy.

## `Default` so với `Policy-0`
- Storage policies là tính năng linh hoạt ý định để hỗ trợ cả hệ thống mới và hệ thống có từ trước cùng mức độ linh hoạt.
- Khái niệm `Policy-0` không giống `default` policy.
- Khi chúng ta bắt đầu cấu hình các policies, mỗi policy có một tên riêng và một số tùy ý để xác định như là index.
- `Default` policy là chính sách mặc định khi một container tạo ra mà không chỉ rõ là thuộc policy nào
- `Policy-0` là chính sách mặc định trong Swift có index là 0.
