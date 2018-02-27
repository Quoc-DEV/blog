---
layout: post
title: The many flavors of commit()
date: 2018-02-27 11:26:00 +0700
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: flavors_commit.jpg # Add image post (optional)
tags: [Java, android]
comments: true
---

### FragmentTransactionKhi nào bạn lựa chọn `commit()` một cách chính xác cho FragmentTransaction?


`FragmentTransaction` nằm trong support library và hiện tại nó cung cấp rất nhiều cách khác nhau để commit một transaction:
 - [commit()](https://developer.android.com/reference/android/app/FragmentTransaction.html#commit%28%29)
 - [commitAllowingStateLoss()](https://developer.android.com/reference/android/app/FragmentTransaction.html#commitAllowingStateLoss%28%29)
 - [commitNow()](https://developer.android.com/reference/android/app/FragmentTransaction.html#commitNow%28%29)
 - [commitNowAllowingStateLoss()](https://developer.android.com/reference/android/app/FragmentTransaction.html#commitNowAllowingStateLoss%28%29)

Có lẽ bạn đã chạm trán một trong chúng cùng với [executePendingTransactions()](https://developer.android.com/reference/android/app/FragmentManager.html#executePendingTransactions%28%29)

Vậy đặc điểm của mỗi loại trên là gì và chúng ta nên sử dụng loại nào? Cùng mình phân tích bên dưới để hiểu rõ hơn!

### commit() vs commitAllowingStateLoss()
Một trường hợp mà hầu hết các dev đều gặp phải khi sử dụng Fragments là bắn ra ngoại lệ `IllegalStateException` nói rằng bạn không thể thực hiện một `commit()` sau khi `onSaveInstanceState()`. Vậy điều này có ý nghĩa gì cho các dev?


![error]({{site.baseurl}}/assets/img/error_fragment.png)


`commit()` và `commitAllowingStateLoss()` gần như giống nhau trong quá trình thực hiện. Chỉ khác biệt duy nhất khi bạn gọi `commit()` thì FragmentManager sẽ check nó đã lưu trạng thái của nó chưa. Nếu nó đã lưu trạng thái của nó, nó sẽ ném một ngoại lệ `IllegalStateException`.

Vậy bạn sẽ mất gì? Nếu bạn gọi `commitAllowingStateLoss()` sau khi `onSaveInstStanceState()`? Câu trả lời là bạn có thể mất trạng thái của FragmentManager và bằng cách mở rộng trạng thái của bất kỳ Fragmnet nào được thêm vào hoặc xoá bỏ kể từ `onSaveInstanceState()`.

   Dưới đây là ví dụ:
1. Activity của bạn đang hiển thị và hiện tại đang show FragmentA.
2. Activity của bạn sent to the background (onStop() and onSaveInstanceState() are called).
3. Để hồi đáp 1 vài sự kiện, bạn replace FragmentA bằng FragmentB và gọi `commitAllowingStateLoss()`.

Tại thời điểm này, một trong hai điều dưới đây có thể xảy ra khi người dùng quay lại ứng dụng của bạn:

 - Nếu hệ thống đã killer ứng dụng của bạn để nhường chỗ cho một ứng dụng khác, sau đó ứng dụng của bạn sẽ được tái tạo với trạng thái đã lưu trong bước 2. FragmentB sẽ không được hiển thị.
 - Nếu hệ thống không kill ứng dụng của bạn(ứng dụng của bạn vẫn còn trong bộ nhớ), sau đó nó sẽ được đưa trở lại foreground và FragmentB vẫn sẽ được hiển thị.
 
 Lần tiếp theo khi Activity dừng, trạng thái bao gồm FragmentB sẽ được lưu lại.
 
 
 ![map]({{site.baseurl}}/assets/img/map_commit.png)
 
 
 [Đây là demo](https://github.com/bherbst/FragmentStateLoss), Nếu bạn bật tùy chọn nhà phát triển `"Don’t Keep Activities"` trong cài đặt của thiết bị, bạn sẽ gặp trường hợp đầu tiên và ngược lại nếu bận tắt thì sẽ gặp trường hợp 2.
 
 ### commit(), commitNow(), and executePendingTransactions()
 
 Các biến thể khác của `commit()` chỉ định khi transaction xảy ra. Tài liệu cho `commit()` đưa ra lời giải thích cho sự lựa chọn:
 
 > Schedules một commit của transaction này là: việc commit không xảy ra ngay lập tức, nó sẽ được lên kế hoạch như công việc trên main thread sẽ được thực hiện trong thời gian tới mà thread đã sẵn sàng.
 
 Điều này có nghĩa là trong thực tế là bạn có thể thực hiện bất kỳ số lượng transaction nào trong một thời gian, và không một commit trong số đó sẽ thực sự xảy ra cho đến khi lần tiếp theo main thread đã sẵn sàng. Điều này bao gồm adding, removing, và replacing Fragments ngoại trừ popping the back stack thông qua [popBackStack()](https://developer.android.com/reference/android/app/FragmentManager.html#popBackStack%28%29).
 
 Đôi khi bạn muốn các transaction của bạn xảy ra ngay lập tức. Các dev trước đây đã hoàn thành việc này bằng cách gọi hàm `executePendingTransactions()` sau khi gọi `commit()`, `executePendingTransactions()` sẽ thực hiện tất cả các transaction mà bạn hiện đang xếp hàng đợi và sẽ xử lý chúng ngay lập tức.
 
 Trong ver 24.0.0 của support library đã added `commitNow()` là lựa chọn tốt hơn dùng `executePendingTransactions()`. Trước đây chỉ thực hiện các transaction đồng bộ hiện tại, sau đó sẽ thực hiện tất cả các transaction bạn committed và hiện đang chờ xử lý. `commitNow()` ngăn không cho bạn thực hiện nhiều transaction hơn bạn thực sự muốn thực thi.
 
 Chú ý là bạn không thể sử dụng `commitNow()` với một transaction mà bạn đang thêm vào back stack. Hãy suy nghĩ về nó bằng cách này - nếu bạn đã thêm một transaction vào back stack thông qua `commit()` sau đó ngay lập tức thêm một transaction vào back stack thông qua commitNow(), vậy bạck stack trông như thế nào? Bởi vì framework không thể cung cấp bất kỳ đảm bảo nào về đơn đặt hàng ở đây, đơn giản nó không được hỗ trợ.
 
 
 ![map_explain]({{site.baseurl}}/assets/img/map_commit_2.png)
 
 
 Trên một lưu ý phụ, `popBackStack()` có một bản sao `popBackStackImmediate()`, tương tự như `commit()` và `commitNow()`. Trước đây là không đồng bộ, sau này là đồng bộ.

 #### Bạn nên dùng cái nào?
 
 - Nếu bạn cần đồng bộ và bạn không thêm transaction của bạn vào back stack, sử dụng `commitNow()`. The support library sử dụng phần này trong FragmentPagerAdapter để đảm bảo rằng các page chính xác đã được thêm vào hoặc xoá khi kết thúc bản cập nhật. Nói chung, nó là tốt để sử dụng bất cứ lúc nào bạn đang thực hiện một transaction mà bạn không phải là thêm vào back stack.
 - Nếu bạn đang thực hiện nhiều transaction, không cần đồng bộ, hoặc thêm các transaction vào back stack, bạn nên gắn bó với `commit()`.
 - Sử dụng `executePendingTransactions()` nếu bạn cần đảm bảo rằng một tập hợp các transaction xảy ra bởi một thời điểm nhất định.

Thank full and cover by [article](https://medium.com/@bherbst/the-many-flavors-of-commit-186608a015b1)
