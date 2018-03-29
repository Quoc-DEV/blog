---
layout: post
title: Mastering Kotlin standard functions run, with, let, also and apply
date: 2018-03-15 11:26:00 +0700
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: img_flavor.png # Add image post (optional)
tags: [kotlin, android]
comments: true
---

Một số [standard functions](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/util/Standard.kt) rất giống nhau và ta không biết nên chọn loại nào cho phù hợp. Ở đây, mình sẽ giới thiệu cách phân biệt rõ ràng để dễ dàng sử dụng trong từng trường hợp.

### Scoping functions

Những functions mình sẽ tập trung là `run, with, T.run, T.let, T.also` và `T.apply`.

Cách đơn giản nhất để minh hoạ là nhìn vào ví dụ sau đây:

~~~kotlin
fun test() {
    var mood = "I am sad"

    run {
        val mood = "I am happy"
        println(mood) // I am happy
    }
    println(mood)  // I am sad
}
~~~

Xét ví dụ trên, bên trong `test` functions bạn có thể có một phạm vi riêng biệt, nơi mà biến `mood` được định nghĩa lại trước khi println và  nó được nằm gọn trong phạm vi của `run`.

Nhìn có vẻ như nó thật sự không hữu dụng cho lắm. Nhưng ở một mặt khác nó không chỉ hoạt động trong phạm vi mà còn `returns` về một cái gì đó tức là giá trị cuối cùng của đối tượng đó.

Chính vì vậy đoạn code bên dưới sẽ rất clear, chúng ta có thể `show()` cả 2 trường hợp mà không cần phải gọi lại cho mỗi trường hợp.

~~~kotlin
run {
        if (firstTimeView) introView else normalView
    }.show()
~~~


### 3 đặc tính của scoping functions

Để làm cho nó trở nên thú vị hơn mình sẽ đi vào 3 đặc trưng cơ bản và đây cũng chính là cách để phân biệt chúng với nhau.

#### 1.Normal vs. extension function

Nếu bạn nhìn vào  code bên dưới `with` và `T.run` cả 2 functions nhìn rất giống nhau cùng thực hiện một công việc.

~~~kotlin
with(webview.settings) {
    javaScriptEnabled = true
    databaseEnabled = true
}
// similarly
webview.settings.run {
    javaScriptEnabled = true
    databaseEnabled = true
}
~~~

Tuy nhiên chúng khác biệt, một là `normal function - with`, một là `extension function - T.run`.

Câu hỏi đặt ra vậy mỗi cái có ưu điểm gì?

Thử tưởng tượng nếu `webview.settings` có thể null? Nó sẽ trông giống như miêu tả dưới đây:

~~~kotlin
// Yack!
with(webview.settings) {
      this?.javaScriptEnabled = true
      this?.databaseEnabled = true
   }
}
// Nice.
webview.settings?.run {
    javaScriptEnabled = true
    databaseEnabled = true
}
~~~

Trong trường hợp này rõ ràng `T.run` chiếm ưu thế hơn vì chúng ta có thể check nullability trước khi sử dụng nó.

#### 2. This vs. it argument

Nhìn vào code bên dưới `T.run` và `T.let` cả 2 đều tương tự nhau ngoại trừ 1 điểm cách nó chấp nhận `argument`.

~~~kotlin
stringVariable?.run {
      println("The length of this String is $length")
}
// Similarly.
stringVariable?.let {
      println("The length of this String is ${it.length}")
}
~~~

Bạn nhận ra `T.run` chỉ thực hiện như một `extension function` và gọi `block: T.()`. Do đó trong scope `T` có thể ám chỉ là `this`. Trong lập trình, `this` hầu hết được bỏ qua. Do đó trong ví dụ phía trên chúng ta có thể dùng `$length` thay thế cho `${this.length}` Mình gọi nó là `this as argument`

Tuy nhiên, bạn nhận ra `T.let` đang gửi chính nó vào `block: T.()`. Do đó điều này giống như một đối số lambda đã gửi nó. Nó có thể được gọi trong scope như `it` Mình gọi nó là `it as argument`

Từ những phân tích trên có vẻ như `T.run` hiệu quả hơn `T.let` vì nó bao hàm hơn. Nhưng có một số chức năng








Thank full and cover by [article](https://medium.com/@elye.project/mastering-kotlin-standard-functions-run-with-let-also-and-apply-9cd334b0ef84)

