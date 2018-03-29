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

Từ những phân tích trên có vẻ như `T.run` hiệu quả hơn `T.let` vì nó bao hàm hơn. Nhưng có một số ưu điểm tinh tế của `T.let` dưới đây bạn nên lưu ý:

- `T.let` phân biệt một cách rõ ràng hơn giữa sử dụng variable function/member, the external class function/member.
- Nếu `this` không thể được lược bỏ đi thì nó được truyền đi như một parameter của một function. `it` trông rõ ràng xúc tích hơn `this`.
- `T.let` cho phép bạn chuyển đổi tên biến tốt hơn nghĩa là bạn có thể thay `it` bằng một cái tên khác!

~~~kotlin
stringVariable?.let {
      nonNullString ->
      println("The non null string is $nonNullString")
}
~~~

#### 3. Return this vs. other type

Bây giờ, hãy nhìn vào `T.let` và `T.also` chúng đều giống nhau. Nếu chúng ta nhìn vào content bên dưới đây:

~~~kotlin
stringVariable?.let {
      println("The length of this String is ${it.length}")
}
// Exactly the same as below
stringVariable?.also {
      println("The length of this String is ${it.length}")
}
~~~

Tuy nhiên có 1 chút đặc điểm khác biệt là `nó sẽ return lại cái gì?` `T.let` thì return về một type khác của value trong khi `T.also` return chính nó!  

Cả hai đều hữu dụng cho chuỗi functions, để đơn giản hãy nhìn vào minh hoạ dưới đây:

~~~kotlin
val original = "abc"
// Evolve the value and send to the next chain
original.let {
    println("The original String is $it") // "abc"
    it.reversed() // evolve it as parameter to send to next let
}.let {
    println("The reverse String is $it") // "cba"
    it.length  // can be evolve to other type
}.let {
    println("The length of the String is $it") // 3
}
// Wrong
// Same value is sent in the chain (printed answer is wrong)
original.also {
    println("The original String is $it") // "abc"
    it.reversed() // even if we evolve it, it is useless
}.also {
    println("The reverse String is ${it}") // "abc"
    it.length  // even if we evolve it, it is useless
}.also {
    println("The length of the String is ${it}") // "abc"
}
// Corrected for also (i.e. manipulate as original string
// Same value is sent in the chain 
original.also {
    println("The original String is $it") // "abc"
}.also {
    println("The reverse String is ${it.reversed()}") // "cba"
}.also {
    println("The length of the String is ${it.length}") // 3
}
~~~

Nhìn vào phía trên có thể bạn cho rằng `T.also` giống như vô nghĩa nhưng hãy suy nghĩ chậm lại nó có một số ưu điểm:

- Nó có thể cung cấp một quy trình tách biệt rất rõ ràng trên cùng một đối tượng, tức là làm nhỏ functions.
- Nó có thể rất mạnh mẽ cho tự control trước khi được sử dụng, tạo thành một chuỗi functions tiên tiếp.

Khi áp dụng cả 2 nó sẽ tạo ra một sức mạnh kiểu một bên sẽ phát triển và 1 bên sẽ giữ lại chính nó xem minh hoạ dưới đây:

~~~kotlin
// Normal approach
fun makeDir(path: String): File  {
    val result = File(path)
    result.mkdirs()
    return result
}
// Improved approach
fun makeDir(path: String) = path.let{ File(it) }.also{ it.mkdirs() }
~~~

### Hãy nhìn vào tất cả đặc tính

Nhìn vào 3 đặc tính trên chúng ta có thể phần nào hiểu về cách thức hoạt động của các functions. Tiếp theo mình xin nói về `T.apply` nó không được giới thiệu ở trên. 3 thuộc tính của `T.apply` như dưới đây:

- Nó là một extension function.
- Nó truyền `this` vì là một argument.
- Nó returns về chính nó.

Dưới đây là mô tả:

~~~kotlin
// Normal approach
fun createInstance(args: Bundle) : MyFragment {
    val fragment = MyFragment()
    fragment.arguments = args
    return fragment
}
// Improved approach
fun createInstance(args: Bundle) 
              = MyFragment().apply { arguments = args }
~~~

Hoặc chúng ta có thể tạo nó thành một chuỗi function

~~~kotlin
// Normal approach
fun createIntent(intentData: String, intentAction: String): Intent {
    val intent = Intent()
    intent.action = intentAction
    intent.data=Uri.parse(intentData)
    return intent
}
// Improved approach, chaining
fun createIntent(intentData: String, intentAction: String) =
        Intent().apply { action = intentAction }
                .apply { data = Uri.parse(intentData) }
~~~

### Vậy nên chọn lựa cái nào?

Rõ ràng, với 3 đặc tính trên chúng ta có thể phân loại chức năng một cách phù hợp. Và dựa trên đó, chúng ta có thể hình thành một quyết định dưới đây để giúp xác định functions nào chúng ta muốn sử dụng cho những gì chúng ta cần.

![tree_detail]({{site.baseurl}}/assets/img/15_3_2018_pic2.png)

Hy vọng rằng với mô tả ở trên sẽ làm rõ các functions rõ ràng hơn, và đơn giản hóa việc ra quyết định của bạn, cho phép bạn nắm vững các functions này một cách thích hợp.

Thank full and cover by [article](https://medium.com/@elye.project/mastering-kotlin-standard-functions-run-with-let-also-and-apply-9cd334b0ef84)
