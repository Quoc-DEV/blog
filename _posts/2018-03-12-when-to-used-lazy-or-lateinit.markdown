---
layout: post
title: Kotlin when to Use Lazy or Lateinit
date: 2018-03-12 14:26:00 +0700
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: img_which_one.jpg # Add image post (optional)
tags: [kotlin, android]
comments: true
---

Một thuộc tính của đối tượng đều được định nghĩa tại thời điểm được tạo ra, nhưng vì activities và fragments tách  biệt giữa các view, các thuộc tính được lưu trong store là không bắt buộc khi bắt đầu loading view.

Bài viết này sẽ chỉ ra 1 cách rõ ràng về một số cách tiếp cận để xử lý các properties tham  chiếu đến các views:

- Sử dụng một nullable type
- Sử dụng lateinit
- Sử dụng một custom getter
- Sử dụng lazy
- Hai cách sử dụng custom property delegates

### Nullable Type

Đây là ví dụ đơn giản sử dụng nullable type khi thuộc tính tham chiếu đến một view.

~~~ kotlin
var showAnswerButton: Button? = null
~~~
Vì tất cả các biến cần phải khởi tạo, nên ở đây mình khời tạo null. Sau đó khi trong Activity.onCreate or Fragment.onCreateView nó sẽ được gán giá trị lại, tại thời điểm này giá trị thật sự được gán là giá trị bạn muốn.

~~~ kotlin
showAnswerButton = findViewById(R.id.showAnswerButton)
~~~

Khi sử dụng một kiểu nullable type, các toán tử như `?.` hoặc `!!` phải được truy cập vào các biến nullable. Sử dụng `?.` để tránh các crash bởi giá trị trả lại là null bởi một vài lý do.

~~~ kotlin
showAnswerButton?.setOnClickListener { /* */ }
~~~

Trong Java thì bạn có thể check thế này:

~~~ java
if (showAnswerButton != null) {
    showAnswerButton.setOnClickListener(/* */);
}
~~~

Toán tử `!!` sẽ gây ra lỗi crash nếu showAnswerButton là null:

~~~ kotlin
showAnswerButton!!.setOnClickListener { /* */ }
~~~

So sánh với code trong Java nó sẽ thế này:

~~~ java
showAnswerButton.setOnClickListener(/* */);
~~~

### Lateinit

Một lựa chọn hoàn hảo là `lateinit`

~~~ kotlin
lateinit var tvQuestion: TextView
~~~

Sử dụng lateinit thì giá trị khởi tạo ban đầu không cần phải được gán. Hơn nữa, tại những nơi sử dụng tvQuestion không phải là một nullable type, vâỵ `?.` và `!!` chúng không cần sử dụng. Tuy nhiên, chúng ta cần cẩn thận khi sử dụng lateinit trước khi gán giá trị cho chúng. Nếu không thì, một lateinit sẽ hoạt động như một `!!` nó sẽ làm crash app nếu giá trị null.

### Custom getter

Bạn có thể tạo giống như sau:

~~~ kotlin 
val anotherTextView: TextView
    get() = findViewById(R.id.another_text_view)
~~~

Cách tiếp cận này có một nhược điểm lớn, mỗi lần biến được truy cập thì findViewByID sẽ được gọi. Hơn nữa, nếu là Fragments bạn cần sử dụng toán tử `!!` mỗi khi truy cập thuộc tính. Bùm bùm tới đây bất kỳ ảo tưởng về an toàn không còn nữa.

### Lazy

Một biến được định nghĩa bởi lazy là được khởi tạo bằng cách sử dụng lambda được cung cấp lần đầu sử dụng, trừ khi một giá trị đã được truy cập trước đây.

~~~ kotlin 
val nameTextView by lazy { view!!.findViewById<TextView>(R.id.nameTextView) }
~~~

Cách tiếp cận này sẽ crash nếu nameTextView được truy cập trước khi setContentView trong một Activity. Sự kiện này còn trở nên phức tạp hơn trong Fragments vì nó sẽ gây ra lỗi crash bên trong onCreateView ngay cả khi view đã được inflated. Đó là lý do vì sao biến không được set giá trị cho đến khi onCreateView hoàn thành và nó được tham chiếu trong biến khởi tạo. Có thể sử dụng các biến khởi tạo bằng lazy trong onViewCreated.

Sử dụng một khởi tạo lazy trên fragments sẽ gây ra lỗi memory leak, vì biến vẫn giữ tham chiếu đến views cũ.

### Custom Property Delegate

Không giống như những ngôn ngữ khác, lazy trong Kotlin không phải là một tính năng ngôn ngữ, nhưng một property delegate bổ sung như là một thư viện chuẩn. Do đó, nó là khả thi để vẽ lên một cảm hứng và thực hiện một cách khác đầy lý trí. Vậy liệu nó có thể giải quyết rò rỉ bộ nhớ của cách tiếp cận lazy không?

Android 
Không giống như những ngôn ngữ khác, lazy trong Kotlin không phải là một tính năng ngôn ngữ, nhưng một property delegate bổ sung như là một thư viện chuẩn. Do đó, nó là khả thi để vẽ lên một cảm hứng và thực hiện một cách khác đầy lý trí. Vậy liệu nó có thể giải quyết rò rỉ bộ nhớ của cách tiếp cận lazy không?

Android Architeture Components bao gồm support lifecycle awareness. Biến cityTextView được khởi tạo bởi LifecycleAwareLazy, trong trường hợp Lifecycle clears giá trị khi ON_STOP được gọi. Điều này vẫn đảm bảo giá trị được khởi tạo lại.

~~~ kotlin 
val cityTextView by lifecycleAwareLazy(lifecycle) { view!!.findViewById<TextView>(R.id.cityTextView) }
~~~

Điều này tạo ra sự tò mò thú vị, trong khi cityTextView đã được định nghĩa là val, bời vì nó được định nghĩa bởi property delegate nên nó vẫn có thể thay đổi như thể nó là một var. Trong thực tế, một biến có delegate thậm chí còn không thể khai báo với var.

stateTextView được khởi tạo từ LifecycleAwareFindView, với Fragment nó vẫn xảy ra khi implement LifecycleOwner hoặc view ID.

~~~ kotlin 
val stateTextView: TextView by findView(this, R.id.stateTextView)
~~~

2 cách trên giải quyết 1 vấn đề nhưng không hoàn hảo, bạn vẫn có thể bị rò rỉ bộ nhớ.


### Khi nào sử dụng lazy hoặc lateinit?

_lazy_ rất phù hợp với các biến có thể hoặc không thể truy cập. Nếu chúng ta không bao giờ truy cập chúng, bạn sẽ tránh được khởi tạo giá trị ban đầu. Chúng vấn run cùng Activities, miễn là chúng chưa được truy cập trước khi `setContentView` được gọi. Và nó thì không phù hợp để tham chiếu views trong Fragment, vì mô hình phổ biến khi khởi tạo views trong `onCreateView` sẽ gây ra lỗi crash. Họ có thể sử dụng nếu view được khởi tạo trong `onViewCreated`.

Với Activities or Fragment, sẽ là cảm giác tốt hơn nếu sử dụng lateinit cho các properties đặc biệt một trong những tham chiếu tới views. Mặc dù, chúng ta không control vòng đời, nhưng vẫn biết khi nào một trong các properties sẽ khởi tạo đúng cách. Nhươc điểm là chúng ta cần đảm bảo chúng được khởi tạo trong các method của vòng đời thích hợp.

Thank full and cover by [article](https://www.bignerdranch.com/blog/kotlin-when-to-use-lazy-or-lateinit/)
