---
layout: post
title: Hash function là cái gì?
date: 2018-05-7 15:34:00 +0700
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: 7_5_2018_pic1.jpg # Add image post (optional)
tags: [blockchain]
comments: true
---

Hash function hay với dịch ra tiếng việt nôm na là hàm băm. Thực chất nó chỉ là một hàm nhận giá trị đầu vào và từ giá trị này tạo ra một giá trị đầu ra nhằm xác định giá trị đầu vào. Một cách đơn giản hơn, khi ta nhập 1 giá 'x' nào đó sẽ luôn tạo ra một giá trị 'y' nào đó để xác nhận cho giá trị 'x'. Bằng cách này thì mọi giá trị đầu vào đều có 1 giá trị đầu ra xác định giá trị ban đầu.

Một hàm đơn giản là bạn đưa một cái gì đó vào thì sẽ tạo ra một cái gì đó từ cái ban đầu.

```
f(x) = y
```
Do đó `hash function` là đưa vào một cái gì đó có thể là: bất kỳ dữ liệu nào, số, tệp,... thì cho ra một mã hash. Một mã hash thì được biểu diễn dưới mã thập lục phân([hexadecimal number](https://vi.wikipedia.org/wiki/H%E1%BB%87_th%E1%BA%ADp_l%E1%BB%A5c_ph%C3%A2n))

```
md5("hello world") = 60c08e2b03b716a176aeb9c2a2fddb79
```

Đây là hash function md5, mà từ bất kỳ dữ liệu đầu vào nào tạo ra kết quả thập lục phân 32 ký tự. Hash function thường không thể đảo ngược (một chiều), có nghĩa là bạn không thể tìm ra đầu vào nếu bạn chỉ biết đầu ra - trừ khi bạn thử mọi đầu vào có thể (được gọi là tấn công brute-force).

Hash function thường được sử dụng để chứng minh rằng một cái gì đó giống như một cái gì đó khác, mà không tiết lộ thông tin trước đó. Đây là một ví dụ.

Giả sử Alice khoe khoang với Bob rằng cô ấy biết câu trả lời cho câu hỏi thách thức trong lớp Toán của họ. Bob muốn cô ấy chứng minh rằng cô ấy biết câu trả lời, mà không nói cho cô ấy biết đó là gì. Vì vậy, Alice sử dụng hash function cho câu trả lời của cô ấy (giả sử câu trả lời là 42) để tạo ra hash function này:

```
md5(42) = a1d0c6e83f027327d8461063f4ac58a6
```
Alice đưa mã hash này cho Bob. Bob không thể tìm ra câu trả lời là gì từ mã hash này - nhưng khi anh ta tìm ra câu trả lời, anh ta có thể dúng mã hash câu trả lời của mình và nếu anh ta nhận được kết quả tương tự, thì anh ấy biết rằng Alice thực sự có câu trả lời.

Thank full and cover by [article](https://learncryptography.com/hash-functions/what-are-hash-functions)
