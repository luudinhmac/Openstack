##1. DLO (object lớn hơn 5gb)
- tự động tạo file manifest để link giữa các phần của đối tượng
- đầu tiên upload các phần của đối tượng

```
curl -i -X PUT -H 'X-Auth-Token: 6a36d61dd9f94be28bb4bc12f92a1144'  http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/container/myobject/1 --data-binary '1'
curl -i -X PUT -H 'X-Auth-Token: 6a36d61dd9f94be28bb4bc12f92a1144' \     http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/container/myobject/2 --data-binary '2'
curl -i -X PUT -H 'X-Auth-Token: 6a36d61dd9f94be28bb4bc12f92a1144' \     http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/container/myobject/3 --data-binary '3'

```

##### tạo file manifest

```
curl -i -X PUT -H 'X-Auth-Token: 3f4a05b2ba6a4d1f9fa50b95da58dfdc' \
-H 'X-Object-Manifest: container/myobject/' \     http://10.10.10.140:8080/v1/AUTH_11af54a847164d75b8b6ddad0e82c079/container/myobject --data-binary ''
```

##2. Versioning object

- Lưu trữ những bản cũ của object ra 1 container riêng tránh trường hợp bị ghi đè

- tạo 2 container

```
curl -i -X PUT -H "X-Auth-Token: a05228d261fe46f1be1c298802416549"  http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/fragile_data
curl -i -X PUT -H "X-Auth-Token: a05228d261fe46f1be1c298802416549"  http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/old_versions
```

- tạo trường X-version-location kích hoạt chức năng

```
curl -i -X PUT -H "X-Auth-Token: a05228d261fe46f1be1c298802416549" -H "X-Versions-Location: old_versions"  http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/fragile_data
```

- Tạo object và update vào 2 nội dung khác nhau sau đó kiểm tra ở container old_versions sẽ thấy phiên bản có nội dung là 111

```
curl -i -X PUT -H "X-Auth-Token: a05228d261fe46f1be1c298802416549" "http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/fragile_data/obj1" --data-binary '111'
curl -i -X PUT -H "X-Auth-Token: a05228d261fe46f1be1c298802416549" "http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/fragile_data/obj1" --data-binary '222'
 ```

##3. Tempurl (url tạm thời)
- kích hoạt tính năng với url key đặc trưng

```
curl -i -X POST http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf -H "X-Account-Meta-Temp-URL-Key: secret" -H "X-Auth-Token: a7fba3e7e1b447eeb0056922cdbe4460" -i
```

- kiem tra url key

```
curl -i http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf?format=json -X GET -H "X-Auth-Token: a7fba3e7e1b447eeb0056922cdbe4460"
```

- tạo url tạm thời, kết quả của lệnh sau sẽ sinh ra 1 url tạm thời để download đối tượng.
```
swift tempurl <method> <seconds> <path> <key>
```



##4.Custome metadata
- thay đổi metadata của account

```
curl -i -X POST -H "X-Auth-Token: e3a67b06f1a04760bafdd0998d942640"  -H "X-Account-Meta-Favorite-Even-Prime: 2"  "http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf"
```

- thay đổi metadata của container

```
curl -i -X POST -H "X-Auth-Token:e3a67b06f1a04760bafdd0998d942640 " -H "X-Container-Meta-Used-For-Web: No" "http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/fragile_data"
```

##5.Swift cluster info (thông tin của cụm cluster)
`curl -X GET http://172.16.69.238:8080/info "X-auth-token:1f20552b1cc34ba0b0eed62db443166f"``


##6.range request
- phạm vi yêu cầu (ví dụ đối tượng có 10byte thì chỉ thao tác với 5byte đầu)

```
curl -X GET -H "X-Auth-Token: f6bf5d8f44d04eefa08179ac6ed6359c" -H "Range: bytes=1-5" "http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/fragile_data/obj1"
```

##7.content-type (định dạng đối tượng)

```
curl -i -X PUT -H "X-Auth-Token: 8c29c7a889824abaac03e14acbfca816" "http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/tesst/obj2" \  --data-binary @/tmp/myimage/beautiful-resort.jpg -H "Content-Type: image/png"
```

##8.bulk upload (upload nhiều đối tượng)
- nén nhiều đối tượng vào file nén archive.tar sau khi upload Swift sẽ giải nén và cho vào 1 thư mục
```
curl -X PUT -H "X-Auth-Token: 3d6463ed656b4783b9507021ad713760" \  "http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/tesst/cont123?extract-archive=.tar" --data-binary @/tmp/myimage/archive.tar
```
##9. bulk delete(delete nhiều đối tượng)

```
curl -i -X DELETE -H "X-Auth-Token: c2fa1269410a46aa8039f8b7d1ce5283" -H "Content-Type: text/plain" -H "Accept: application/json"  http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/?bulk-delete --data-binary @bulklist.txt

```

- tạo 1 bulklist.txt có nội dung là đối tượng muốn xóa theo dạng container/object


##10.Static Web Hosting
- hiển thị container như 1 web tĩnh. tạo file my-index.txt nội dung hello, sau khi thực hiện lệnh dưới, vào trình duyệt với đường dẫn tới container sẽ hiển thị nội dung file index

```
curl -X POST -H "X-Auth-Token: ab189a14361f4a93be5ee783331b2fb2" -H 'X-Container-Read: .r:*' "http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/tesst"

curl -X POST -H "X-Auth-Token: ab189a14361f4a93be5ee783331b2fb2"  -H 'X-Container-Meta-Web-Index: my-index.html' "http://172.16.69.238:8080/v1/AUTH_870d62be2c254d428226da346dfe9fbf/tesst"

```

##11. make container public

`curl -X POST -H "X-Auth-Token: $TOKEN" -H 'X-Container-Read: .r:*' "$STORAGE_URL/website"`
