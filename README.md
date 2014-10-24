QQ 填充算法和Tea加解密的JavaScript实现
=========

Example:
```javascript

/*
QQ PC Client UDP 数据包样本:
02 35 47 08 25 05 87 00 00 00 00 03 00 00 00 01 01 01 00 00 67 22 00 00 00 00 E7 15 BA 2E D5 49 63 A5 5D 29 9E A7 82 7B A7 CD E8 F3 03 39 AB 88 A0 8A 48 42 62 7A 9F BB 80 6F 47 54 82 57 32 4F 38 71 5A 17 3C A4 35 BD FF AD 6C 10 11 06 13 A7 A2 85 AA E0 26 20 D9 78 B8 FE D5 B7 A7 B4 64 FD BE 7E B7 8D FF 8E 9C 8D D5 B3 95 AE 74 68 8F 56 F5 0A 72 68 B8 68 E2 AB CC 77 FA 1C 2A A2 11 91 6A CD 82 DA 02 89 54 46 E2 38 C4 E5 D0 C5 9C 5C 41 1B 03
*/

var tea = require('./lib/qqtea');

key = tea.hex2bin('E7 15 BA 2E D5 49 63 A5 5D 29 9E A7 82 7B A7 CD');

data = tea.hex2bin('E8 F3 03 39 AB 88 A0 8A 48 42 62 7A 9F BB 80 6F 47 54 82 57 32 4F 38 71 5A 17 3C A4 35 BD FF AD 6C 10 11 06 13 A7 A2 85 AA E0 26 20 D9 78 B8 FE D5 B7 A7 B4 64 FD BE 7E B7 8D FF 8E 9C 8D D5 B3 95 AE 74 68 8F 56 F5 0A 72 68 B8 68 E2 AB CC 77 FA 1C 2A A2 11 91 6A CD 82 DA 02 89 54 46 E2 38 C4 E5 D0 C5 9C 5C 41 1B');

result = tea.decrypt(key, data);//解密

console.log(tea.bin2hex(result));//输出结果(hex)

data = tea.encrypt(key, result);//加密

console.log(tea.bin2hex(data));//输出加密后的结果(hex), 因为QQ的Tea算法中对前8个字节采用随机填充的机制,所以每次加密后的结果都是一样的

result = tea.decrypt(key, data);//再次进行解密

console.log(tea.bin2hex(result));//输出解密后的结果(hex)


/*
    与腾讯QQ服务器交互的样本.
	功能: 根据Email账号获取对应的QQ号码.
*/

var email = 'dds_feng@qq.com';//要查询的Email

data = tea.hex2bin('02') + String.fromCharCode(email.length) + email;//构造请求包

data = tea.encrypt(key, data);//使用KEY进行加密

var message = new Buffer(tea.hex2bin('02122100b20D1B00000000') + key + data + '\x03', 'binary');//构造一个Buffer(因为dgram只能发送Buffer类型的数据)

require('dgram').createSocket('udp4').on('message', function (message, remote) {//receive message
	var data = message.toString('binary', 7, message.length - 1);//得到数据体(忽略包头和包尾)

	data = tea.decrypt(key, data);//使用之前的加密KEY进行解密

	message = new Buffer(data, 'binary');

	console.log('QQ Number:\t' + parseInt(message.toString('hex', 2, message.length - (message.length - 6)), 16));//输出对应的QQ号码

	this.close();
}).send(message, 0, message.length, 8000, 'sz3.tencent.com');//向腾讯QQ的服务器发送数据包

```


> JavaScript 算法根据 [QQ填充算法和TEA 加解密的python实现](http://bbs.chinaunix.net/thread-583468-1-1.html) 改写而成.
>

