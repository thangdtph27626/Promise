# Xử lý bất đồng bộ với Promise.all trong JavaScript

## Promise

JavaScript là đơn luồng, có nghĩa là chúng ta chỉ có thể chạy một khối mã tại một thời điểm. Nó thực thi mã theo thứ tự và phải hoàn thành việc thực thi mã trước khi chạy mã tiếp theo.

Một Promise đại diện cho kết quả tương lai của một hoạt động không đồng bộ. Promise thường được sử dụng để xử lý các tác vụ không đồng bộ trong JavaScript.

Promise là một đối tượng sẽ trả về một giá trị trong tương lai, nó có thể là giá trị được giải quyết, nghĩa là Promise đã thành công hoặc giá trị bị từ chối, nghĩa là đã xảy ra lỗi. Một Promise sẽ chỉ trả về một giá trị một lần, điều đó có nghĩa là nếu một Promise trả về lỗi thì nó sẽ chỉ trả về một giá trị một lần.

Một Promise có thể có ba trạng thái loại trừ lẫn nhau :

- fulfilled   - một Promise được thực hiện nếu sẽ gọi “càng sớm càng tốt”promise.then(f)f
- rejected   - một Promise bị từ chối nếu sẽ gọi “càng sớm càng tốt”promise.then(undefined, r)r
- pending   - một Promise đang chờ xử lý nếu nó không được thực hiện cũng như không bị từ chối
Đôi khi chúng ta có thể nghe thấy rằng một Promise là settled. Điều đó có nghĩa là Promise này là or fulfilledhoặc rejected, settledkhông phải là một trạng thái mà nó được sử dụng chỉ để thuận tiện.

Để tạo một Promise, chúng ta sử dụng newtừ khóa và bên trong Promise đối tượng, chúng ta truyền một hàm. Hàm này được gọi executorvà cần có hai đối số resolveđể thành công và reject báo lỗi:

```
const firstPromise = new Promise((resolve, reject) => { 
  ... 
});
```
   
Bên trong Promise có một điều kiện và đây là nơi bạn đặt logic của mình. Trong trường hợp điều kiện được đáp ứng, chúng ta sử dụng resolveđối số để trả về thành công cho mình. Trong trường hợp có lỗi, rejectđối số sẽ trả về lỗi cho Promise:

```
const firstPromise = new Promise((resolve, reject) => {
  const sum = () => 1 + 1;
  if (sum() === 2) resolve("Success");
  else reject("Error");
});
```

## Promise.all


```
Promise.all([Promise1, Promise2, Promise3])
  .then((result) => {
    console.log(result);
  })
  .catch((error) => console.log(`Error in promises ${error}`));
```

Promise1, Promise2 và Promise3 là những promise, bản thân Promise.all cũng là 1 promise. Vậy nó khác gì với việc chạy riêng lẻ 3 promise từ 1-3 một cách độc lập. Đầu tiên thì trong Promise.all, cả 3 Promise1-3 sẽ được thực hiện đồng thời, không cái nào cần chờ cái nào, vì thế giảm được thời gian chờ đợi và đương nhiên là tăng performance. Tiếp theo chúng ta cần lưu ý, nếu 1 trong 3 Promise1-3 đẩy ra lỗi (reject) thì Promise sẽ ngay lập tức dừng lại và nhảy vào catch chứ không thực hiện tiếp các promise khác. Trong trường hợp các task con thực hiện thành công thì Promise.all cũng sẽ trả về resolve với kết quả là 1 mảng chứa tất cả các resolve của task con.

## Promise.all với map()


Một bài toán cũng hay gặp với Promise.all là việc sử dụng nó kết với với map nhằm tạo ra 1 group các xử lý bất đồng bộ theo mảng đầu vào. Chẳng hạn như bạn cần gửi email cho n users có sẵn trong 1 mảng dữ liệu; lúc này chúng ta có thể sử dụng hàm map() để tạo ra mảng promise con làm tham số đầu vào cho Promise.all. Ví dụ như dưới đây:

```
const timeOut = (t) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(`Completed in ${t}`);
    }, t);
  });
};

const durations = [1000, 2000, 3000];

Promise.all(durations.map((duration) => timeOut(duration))).then((response) =>
  console.log(response)
);
```

## Promise.allSettled

JavaScript cung cấp cho chúng ta Promise.allSettled để giải quyết vấn đề đầu tiên của Promise.all. Promise.allSettled trả về 1 Promise mới mà ở đó nó sẽ chờ tất cả các task con thực hiện xong, không kể có hoàn thành (resolve) hay bị reject. Kết quả trả về là 1 mảng theo thứ tự các Promise con truyền vào, mỗi object chứa status (trạng thái) của promise cùng giá trị trả về value hay reason (lí do) khi bị reject. Tất nhiên Promise.allSettled sẽ cần nhiều thời gian xử lý hơn so với Promise.all, tuy nhiên nó đảm bảo tất cả Promise được thực hiện.

```
Promise.allSettled([
  Promise.reject("This failed."),
  Promise.reject("This failed too."),
  Promise.resolve("Ok I did it."),
  Promise.reject("Oopps, error is coming."),
]).then((res) => {
  console.log(`Here's the result: `, res);
});
```

## Promise.race và Promise.any


Giả sử chúng ta có 2 bài toán như sau, các bạn hãy xem và lựa chọn các dùng nào của Promise nhé.

- Bài toán 1: có 3 API get thông tin thời tiết và trả về kết quả như nhau, do vậy chỉ cần gọi 1 trong 3 API, cái nào có kết quả là được; không cần gọi tất cả 3 cái. Chúng ta sẽ làm thế nào?
- Bài toán 2: chức năng upload cần upload file lên 3 server khác nhau và phải đảm bảo việc upload thực hiện thành công trên cả 3 server, nếu 1 trong 3 task fail thì dừng luôn 2 task còn lại.
Để giải quyết 2 bài toán trên, JavaScript cung cấp thêm cho chúng ta 2 function nữa là Promise.race và Promise.any. Promise.race và Promise.any có điểm chung là đều sẽ hoàn thành (resolve) khi 1 trong các promise con được thực hiện xong. Như ví dụ bài toán 1, nếu có 1 API get thông tin thời tiết hoàn thành xong thì nếu dùng Promise.race hoặc Promise.any, 2 API get thông tin thời tiết còn lại sẽ không cần thực hiện nữa.

Điểm khác nhau giữa race và any là ở trường hợp reject: any sẽ chỉ reject khi tất cả các promise con reject (hay không có promise nào trả về resolve); còn race sẽ reject khi chỉ cần 1 promise con reject. Điều này giúp Promise.race giải quyết bài toán thứ 2 ở trên. Khi có 1 trong 3 task upload file lên server thất bại thì sẽ dừng luôn 2 task còn lại.

## Khi nào nên sử dụng
Để sử dụng phương pháp này, trước tiên bạn cần biết mình cần đạt được điều gì. Phương pháp này rất hữu ích và hữu ích trong một số trường hợp

- Các nhiệm vụ bạn đang thực hiện phụ thuộc vào nhau và bạn muốn biết liệu tất cả các Promise đã hoàn thành thành công hay chưa
- Bạn cần đưa ra yêu cầu tới các API khác nhau và sau tất cả các phản hồi bạn muốn làm điều gì đó với kết quả
Đây là một cách tuyệt vời để đạt được tính đồng thời trong JavaScript, đây là một trong những cách tốt nhất để thực hiện các hoạt động không đồng bộ đồng thời trong JavaScript khi bạn có nhiều Promise và bạn muốn thực hiện tất cả Promise.all
