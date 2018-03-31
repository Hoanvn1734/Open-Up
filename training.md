# 1. CakePHP
## 1.1 Controllers
Controller là 'C' trong MVC. Thông thường một controller được sử dụng để quản lý logic xung quanh một model. Trong CakePHP, controller được đặt tên theo model mà nó xử lý.

Các Controller được extends từ class `AppController`. `AppController` có thể được định nghĩa trong **src/Controller/AppController** và nó chứa các phương thức được chia sẻ giữa tất cả các controller với nhau.

Controller cung cấp một số phương thức để xử lý các request được gọi là *action*. Theo mặc định, mỗi phương thức public trong một controller là một action, và có thể truy cập từ một URL. Một action xử lý request và tạo ra response.

### 1.1.1 App Controller
`AppController` có cấu trúc như sau:

```
namespace App\Controller;
use Cake\Controller\Controller
class AppController extends Controller
```

Các thuộc tính và phương thức trong `Appcontroller` sẽ có sẵn trong tất cả các Controller extend nó. Có thể sử dụng `AppController` để load các component được sử dụng trong mọi controller của ứng dụng. CakePHP cung cấp một phương thức `initialize()`  được gọi ở cuối của Controller constructor:

```
namespace App\Controller;

use Cake\Controller\Controller;

class AppController extends Controller
{
    public function initialize()
    {
        // Always enable the CSRF component.
        $this->loadComponent('Csrf');
    }
}
```

### 1.1.2 Request Flow
Khi một request được thực hiện đến một ứng dụng CakePHP, lớp `Cake\Routing\Router` và `Cake\Routing\Dispatcher` sử dụng `Connecting Route` để tìm và tạo ra một instance controller. Dữ liệu request được đóng gói trong một đối tượng request. CakePHP đặt tất cả các thông tin request quan trọng vào thuộc tính `$this->request`.

### 1.1.3 Controller Actions
Controller thực hiện chuyển các tham số request thành response cho browser/user thực hiện request. CakePHP sử dụng các quy ước để tự động hóa quá trình này. Theo quy ước, CakePHP sinh ra một view có tên là tên của action.

Controller action thường sử dụng `Controller::set()` để tạo ra `View`. Do các quy ước của CakePHP, không cần phải tạo và render view thủ công. Khi controller action được hoàn thành, CakePHP sẽ xử lý rendering và delivering View.

<!-- ### 1.1.4 Tương tác với View
Controller tương tác với view theo một số cách. Đầu tiên, nó truyền dữ liệu đến view, sử dụng `Controller::set()`. Ta cũng có thể quyết định lớp view nào sử dụng, và những gì view hiển thị.

#### Thiết lập biến View
`Cake\Controller\Controller::set(string $var, mixed $value)`

Phương thức `Controller::set()` dùng để gửi dữ liệu từ controller đến view. Khi ta sử dụng `Controller::set`, biến có thể được truy cập ở trong view

```
// First you pass data from the controller:

$this->set('color', 'pink');

// Then, in the view, you can utilize the data:
?>

You have selected <?= h($color) ?> icing for the cake.
```

#### Thiết lập tùy chọn View
Nếu ta muốn tùy chỉnh view, có thể sử dụng phương thức `viewBuilder()`. Nó có thể được sử dụng để xác định thuộc tính của view trước khi nó được tạo ra.

```
$this->viewBuilder()
    ->helpers(['MyCustom'])
    ->theme('Modern')
    ->className('Modern.Admin');
```

#### Render View
`Cake\Controller\Controller::render(string $view, string $layout)`

Phương thức `Controller::render()` được gọi tự động vào cuối mỗi controller action. Phương thức này thực hiện tất cả các view logic (sử dụng dữ liệu ta gửi bằng phương thức `Controller::set()`), đặt view bên trong `View::$layout`, và trả về cho người dùng cuối.

### 1.1.5 Chuyển trang
`Cake\Controller\Cake\Controller\Controller::redirect(string|array $url, integer $status)`

Phương thức điều khiển luồng hay sử dụng nhất là `Controller::redirect()`. Tham số đầu tiên là đường dẫn URL tương đối hoặc tuyệt đối. Tham số thứ 2 là trạng thái HTTP, ví dụ 301, 303 tùy thuộc vào tình huống.

#### Chuyển hướng đến một action khác trong cùng một Controller
`Cake\Controller\Controller::setAction($action, $args...)`

Nếu cần chuyển tiếp từ action hiện tại đến một action khác trên cùng một controller, ta có thể sử dụng `Controller::setAction()` để cập nhật request, view được sửa và thực hiện chuyển tiếp đến action được đặt tên:

```
// From a delete action, you can render the updated
// list page.
$this->setAction('index');
``` -->

## 1.2 Components
Component là các gói logic được chia sẻ giữa các controller. CakePHP có một tập các core component mà ta có thể sử dụng để hỗ trợ các tác vụ khác nhau. Ta cũng có thể tạo các component riêng. Nếu muốn copy và paste nhiều thứ giữa các controller với nhau, ta nên tạo ra component riêng để chứa các hàm. Tạo component làm cho code của controller "sạch" và cho phép sử dụng lại code giữa các controller.

Các component có trong CakePHP:

- Authentication
- Cookie
- Cross Site Request Forgery
- Flash
- Security
- Pagination
- Request Handling

### 1.2.1 Cấu hình component
Nhiều core component yêu cầu phải cấu hình, ví dụ Authentication và Cookie. Cấu hình cho các component này, và các component nói chung, thường thực hiện thông qua `loadComponent()` trong phương thức `initialize()` của Controller hoặc thông qua mảng $components:

```
class PostsController extends AppController
{
    public function initialize()
    {
        parent::initialize();
        $this->loadComponent('Auth', [
            'authorize' => 'Controller',
            'loginAction' => ['controller' => 'Users', 'action' => 'login']
        ]);
        $this->loadComponent('Cookie', ['expires' => '1 day']);
    }

}
```

Ta có thể cấu hình component tại thời gian chạy sử dụng phương thức `config()`. Thông thường điều này được thực hiện trong phương thức `beforeFilter()` của controller.

```
// Read config data.
$this->Auth->config('loginAction');

// Set config
$this->Csrf->config('cookieName', 'token');
```

Component tự động gộp thuộc tính `$_defaultConfig` với cấu hình constructor để tạo ra thuộc tính `$_config` có thể truy cập bằng `config()`.

### 1.2.2 Sử dụng Components
Khi include các component vào trong controller thì sử dụng chúng khá đơn giản. Mỗi component sử dụng được exposed như thuộc tính trong controller. Nếu ta tải `Cake\Controller\Component\FlashComponent` trong controller, ta có thể truy cập như sau:

```
class PostsController extends AppController
{
    public function initialize()
    {
        parent::initialize();
        $this->loadComponent('Flash');
    }

    public function delete()
    {
        if ($this->Post->delete($this->request->getData('Post.id')) {
            $this->Flash->success('Post deleted.');
            return $this->redirect(['action' => 'index']);
        }
    }
```

### 1.2.3 Sử dụng Component khác trong Component
Thỉnh thoảng một vài component cần phải sử dụng component khác. Trong trường hợp này ta có thể include component khác, sử dụng biến $components:

```
// src/Controller/Component/CustomComponent.php
namespace App\Controller\Component;

use Cake\Controller\Component;

class CustomComponent extends Component
{
    // The other component your component uses
    public $components = ['Existing'];

    // Execute any other additional setup for your component.
    public function initialize(array $config)
    {
        $this->Existing->foo();
    }

    public function bar()
    {
        // ...
    }
}

// src/Controller/Component/ExistingComponent.php
namespace App\Controller\Component;

use Cake\Controller\Component;

class ExistingComponent extends Component
{

    public function foo()
    {
        // ...
    }
}
```

## 1.3 Model
Model là các lớp tạo nên lớp business trong ứng dụng. Chúng có trách nhiệm quản lý hầu hết những gì liên quan đến dữ liệu, tính hợp lệ và tương tác của dữ liệu.

Thông thường, các lớp model mô tả dữ liệu được sử dụng trong các ứng dụng CakePHP để truy cập dữ liệu. Chúng thường đại diện cho một bảng cơ sở dữ liệu nhưng có thể được sử dụng để truy cập vào bất cứ thứ gì điều khiển dữ liệu như file, các dịch vụ web bên ngoài.

Một model có thể được liên kết với các model khác. Ví dụ: Bài Post có thể liên kết với Author hoặc Comment.

Lớp model của CakePHP được tách thành hai đối tượng `Table` và `Entity`. Đối tượng `Table` cho phép ta lưu các bản ghi mới, sửa đổi/xóa các bản ghi hiện có, xác định mối quan hệ. Đối tượng `Entity` đại diện cho các bản ghi và cho phép ta xác định hành vi và chức năng cấp độ row/record. CakePHP sử dụng các quy ước đặt tên để liên kết hai lớp Table và Entity với nhau.

## 1.4 Configuration
Thư mục *config* chứa các file cấu hình mà CakePHP sử dụng. Kết nối cơ sở dữ liệu, bootstrapping, file cấu hình lõi,...

### 1.4.1 Config ứng dụng
Configuration thường được lưu trữ trong các file PHP hoặc INI, và được tải trong khi khởi động ứng dụng. CakePHP đi kèm với một file cấu hình mặc định, nhưng ta cũng có thể thêm file cấu hình bổ sung và tải vào code ứng dụng. `Cake\Core\Configure` được sử dụng cho cấu hình toàn cục, và các lớp như Cache cung cấp phương thức config() để làm cho cấu hình đơn giản và rõ ràng.

#### Load file cấu hình bổ sung
Nếu ứng dụng có nhiều tùy chọn cấu hình, có thể chia thành nhiều file cấu hình nhỏ. Sau khi tạo mỗi file trong thư mục **config/**, ta có thể load chúng trong **bootstrap.php**:

```
use Cake\Core\Configure;
use Cake\Core\Configure\Engine\PhpConfig;

Configure::config('default', new PhpConfig());
Configure::load('app', 'default', false);
Configure::load('other_config', 'default');
```

Ta cũng có thể sử dụng file cấu hình bổ sung để ghi đè môi trường. Mỗi file được load sau **app.php** có thể xác định lại giá trị được khai báo trước đó cho phép ta tùy chỉnh cấu hình cho môi trường.

### 1.4.2 Biến môi trường
Biến môi trường làm cho ứng dụng dễ quản lý hơn khi nó được triển khai trên một số môi trường. Trong **app.php**, hàm env() được sử dụng để đọc cấu hình từ môi trường, và xây dựng cấu hình ứng dụng. CakePHP sử dụng chuỗi **DSN** cho cơ sở dữ liệu, logs, email và cấu hình cache cho phép ta dễ dàng thay đổi các thư viện này trong mỗi môi trường.

Đối với triển khai local, CakePHP cho phép triển khai dễ dàng bằng cách sử dụng biến môi trường. Ta sẽ thấy `config/.env.default` trong ứng dụng. Copy file này vào `config/.env` và chỉnh các giá trị ta có thể cấu hình ứng dụng.

Khi biến môi trường được thiết lập, ta có thể đọc dữ liệu từ môi trường:

`$debug = env('APP_DEBUG', false);`

### 1.4.3 Lớp Configure
`class` Cake\Core\\**Configure**

Lớp Configure có thể được sử dụng để lưu trữ và truy xuất các giá trị cụ thể của ứng dụng hoặc giá trị runtime. Mục tiêu chính của lớp Configure là giữ các biến tập trung để có thể được chia sẻ giữa nhiều đối tượng.

#### Ghi dữ liệu Configuration
`static` Cake\Core\Configure::**write**($key, $value)

Sử dụng `write()` để lưu trữ dữ liệu trong cấu hình của ứng dụng:

`Configure::write('Company.name','Pizza, Inc.')`

#### Đọc dữ liệu Configuration
`static` Cake\Core\Configure::**read**($key = null, $default = null)

Sử dụng để đọc dữ liệu cấu hình từ ứng dụng. Nếu một key đươc cung cấp, dữ liệu sẽ được trả về.

```
// Returns 'Pizza Inc.'
Configure::read('Company.name');
```

#### Kiểm tra để xem dữ liệu cấu hình được định nghĩa
`static` Cake\Core\Configure::**check**($key)

Được sử dụng để kiểm tra nếu một key/path tồn tại và giá trị khác null:

`$exists = Configure::check('Company.name');`

#### Xóa dữ liệu cấu hình
`static` Cake\Core\Configure::**delete**($key)

Được sử dụng để xóa thông tin cấu hình của ứng dụng:

`Configure::delete('Company.name');`

# 2. Bootstrap 4
Bootstrap là một công cụ mã nguồn mở để phát triển với HTML, CSS và JS, cho phép thiết kế website responsive nhanh hơn và dễ dàng hơn. Nhanh chóng tạo mẫu cho các ý tưởng hoặc xây dựng toàn bộ ứng dụng với Sass và mixins, các thành phần được dựng sẵn, và các plugin mạnh mẽ được xây dựng trên jQuery.

## 2.1 Cấu trúc thư mục
Sau khi download và giải nén, ta sẽ nhìn thấy
```
bootstrap/
├── css/
│   ├── bootstrap.css
│   ├── bootstrap.css.map
│   ├── bootstrap.min.css
│   ├── bootstrap.min.css.map
│   ├── bootstrap-grid.css
│   ├── bootstrap-grid.css.map
│   ├── bootstrap-grid.min.css
│   ├── bootstrap-grid.min.css.map
│   ├── bootstrap-reboot.css
│   ├── bootstrap-reboot.css.map
│   ├── bootstrap-reboot.min.css
│   └── bootstrap-reboot.min.css.map
└── js/
    ├── bootstrap.bundle.js
    ├── bootstrap.bundle.min.js
    ├── bootstrap.js
    └── bootstrap.min.js
```

Đây là form cơ bản của Bootstrap: các file biên dịch sẵn để sử dụng nhanh chóng trong các web project. Nó cung cấp CSS và JS đã được biên dịch (`bootstrap.\*`), cũng như CSS và JS được biên dịch và minified (`bootstrap.min.\*`). CSS **source maps** (`bootstrap.\*.map`) có sẵn để sử dụng với các công cụ phát triển của browser. File bundled JS (`bootstrap.bundle.js` và minified `bootstrap.bundle.min.js`) bao gồm **Popper**, nhưng không có **jQuery**.

## 2.2 Các điểm nổi bật của bootstrap 4
### 2.2.1 Chuyển đổi từ LESS sang SASS
Bootstrap bây giờ biên dịch rất nhanh nhờ Libsass, và tham gia vào cộng đồng phát triển SASS.

CSS Preprocessor là một ngôn ngữ kịch bản mở rộng của CSS và được biên dịch thành cú pháp CSS giúp ta viết CSS nhanh hơn và có cấu trúc rõ ràng hơn. CSS Preprocessor có thể giúp ta tiết kiệm thời gian viết CSS, dễ dàng bảo trì và phát triển CSS,...

SASS là một CSS Preprocessor cung cấp thêm các quy tắc như nested rule, variable, mixin, ... Với SASS ta có thể viết CSS theo thứ tự rõ ràng, quản lý các biến đã được định nghĩa sẵn, có thể tự động nén tập tin CSS.

### 2.2.2 Cải tiến hệ thống Grid
Bootstrap 3 hiện tại có 4 dạng grid dành cho cột, đó là `.col-xs-`, `.col-sm-`, `.col-md-`, `.col-lg-`. Bootstrap 4 đã chỉnh lại và giới thiệu thêm dạng grid thứ 5 là `.col-xl-` giúp xây dựng layout hoạt động tốt hơn trên tất cả các thiết bị.

### 2.2.3 Hỗ trợ Opt-in flexbox
Chuyển đổi biến boolean trong file `_variables.scss` và biên dịch lại CSS để thấy sự tiện dụng của các thành phần và hệ thống grid sử dụng flexbox.

Mặc định ban đầu

![](https://viblo.asia/uploads/c5bbd856-90d1-4cc4-9279-d543343a974f.png)

Chuyển đổi sử dụng flexbox

![](https://viblo.asia/uploads/f2d6c6b5-b6ab-4520-b9ca-e8e772dd9af9.png)

### 2.2.4 Bootstrap card
Bootstrap giới thiệu component mới là `Cards`. `Cards` được ra đời để thay thế `wells`, `thumbnails`, `panels`.

![](https://viblo.asia/uploads/ae13b6a3-7fa3-4981-8b69-ea958037ade3.png)

### 2.2.5 Thống nhất các đoạn reset HTML vào một module Reboot
Reboot can thiệp rộng hơn `Normalize.css` (nơi chứa đoạn reset cũ), cung cấp nhiều lựa chọn reset hơn ví dụ như `box-sizing`, `margin`,...tất cả gói gọn trong một file `Sass`.

### 2.2.6 Ngừng hỗ trợ IE8 và chuyển sang sử dụng đơn vị rem/em
IE8 là trình duyệt lỗi thời, có rất nhiều lỗi và hạn chế về mặt công nghệ. Ngừng hỗ trợ IE8 đồng nghĩa với việc Bootstrap 4 có thể tận dụng tối ưu CSS mà không phải quan tâm đến trick hack CSS.

Đơn vị pixel được thay đổi bằng đơn vị `rem` và `em` để phù hợp với kiểu trình bày responsive và tùy chỉnh kích thước các thành phần dễ dàng hơn.

### 2.2.7 Viết lại các hàm Javascript
Tất cả các hàm đều được viết lại bằng ES6 để tận dụng những ưu điểm của hệ thống Javascript mới.

### 2.2.8 Cải thiện vị trí tooltips và popovers
Nhờ vào thư viện `Tether` - một thư viện js được cung cấp bởi bên thứ 3 giúp nâng cấp `tooltips` và `popovers`.

### 2.2.9 Cải thiện Documentation
Bootstrap 4 viết hướng dẫn chi tiết, bố cục trình bày hợp lý, kết hợp với sử dụng `Markdown` giúp mọi người đọc code dễ dàng thông qua các ví dụ.

### Rất nhiều thứ khác nữa
Ví dụ như custom form control, margin, các class padding, các class mới,...

# 3. Nhận xét game Valkyrie Connect
## Level: 15

## Unit: 15

## Các chức năng chính trong game
- Mua, bán, nâng cấp đồ
- Auto
- Tăng tốc độ game
- Mượn tướng
- Tìm kiếm người chơi, follow, block
- Chat
- Arena
- Guild
- Connect Battle
- Event

Ngoài ra còn nhiều chức năng khác nữa.

Em thấy 3 chức năng **Arena**, **Event** và **Battle Connect** là hay hơn cả. Vì:

- **Arena** mình được đấu với người chơi thật nên cảm thấy mới mẻ, không nhàm chán như đi đánh quái.
- **Event** rất phong phú và hấp dẫn, phần thưởng có giá trị, kích thích người chơi, nhưng mà hơi khó.
- **Battle Connet** mình có thể kết hợp với hai người chơi khác để đi đánh boss kiếm tiền và vật phẩm. Boss nhìn cũng khá hùng vĩ.

## Đánh giá về game
Em thấy game này chơi hay. Hình ảnh đồ họa đẹp mắt, nhân vật dễ thương. Hiệu ứng kỹ năng nhân vật hoành tráng, đa dạng. Game có nhiều tính năng thú vị, hấp dẫn. Event trong game phong phú, giá trị.

Nhưng có một điểm em không biết vì sao khi connect vào game thì rất khó, phải connect tới vài lần mới vào được game. Nhiều lúc em cảm thấy cũng hơi khó chịu.
