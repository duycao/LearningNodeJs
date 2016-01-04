# Asynchronus Loop

Vấn đề : 
Giả sử bạn cần đọc danh sách tất cả các thư mục con ngoại trừ các file trong thư mục.
Đoạn code cơ bản như sau:
```
var http = require("http"), fs = require("fs");

function load_albums_list(callback) {
	fs.readdir(
		"albums",
		function(err, files){
			if(err) {
				callback(err);
				return;
			}
			
			var only_dirs = [];
			for(var i = 0; i < files.length; i++) {
				fs.stat("albums/"+files[i], function(err, stats){
					if(stats.isDirectory()) {
						console.log(only_dirs);
						only_dirs.push(files[i]);
					}
				});
			}
			
			callback(null,only_dirs);
		}
	);
}
```
Output sẽ như sau: {"error":null,"data":{"albums":[]}}

Điều gì đã xảy ra ?

vấn đề nằm ở chỗ vòng lặp for. Vòng lặp và hàm asynchronous callback ko tương thích với nhau. Đoạn code trên được thực hiện như sau: 
1. tạo 1 mảng only_dirs để chứa các giá trị.
2. lặp qua các phần tử trong mảng files, gọi 1 hàm nonblocking fs.stat để kiểm tra từng từng phần tử có phải là thư mục hay không. Nếu là thư mục thì thêm phần tử đó vào mảng only_dirs
3. Sau khi tất cả các hàm nonblocking này vừa bắt đầu, chúng thoát khỏi vòng lặp và gọi hàm callback. Bởi vì Node.js là single-thread, các fs.stat vẫn chưa đựợc thực thi và gọi hàm callback, vì thế only_dirs vẫn chưa có phần tử nào. 

Để giải quyết được vấn đề này, ta phải dùng đến đệ qui
```
function iterator(i) {
	if(i<array.length) {
		async_work(function(){
			iterator ++;
		});
	} else {
		callback(results);
	}
}
```