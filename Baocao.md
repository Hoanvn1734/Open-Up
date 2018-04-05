# 1. CakePHP 2
## 1.1 Controllers
Controller là 'C' trong MVC. Controller có thể coi là người trung gian giữa Model và View. Thông thường một controller được sử dụng để quản lý logic xung quanh một model. Trong CakePHP, controller được đặt tên theo model mà nó xử lý.

Các Controller được extends từ class `AppController`. `AppController` có thể được định nghĩa trong **/app/Controller/AppController.php** và nó chứa các phương thức được chia sẻ giữa tất cả các controller với nhau.

Controller cung cấp một số phương thức để xử lý các request được gọi là *action*. Theo mặc định, mỗi phương thức public trong một controller là một action, và có thể truy cập từ một URL. Một action sẽ xử lý request và tạo ra response.

### 1.1.1 App Controller
Class `AppController` là class cha cho tất cả các controller của ứng dụng. `AppController` tự extends class `Controller` trong thư viện core CakePHP.  `AppController` có cấu trúc như sau:

`class AppController extends Controller`

Khi áp dụng các quy tắc của lập trình hướng đối tượng, CakePHP sẽ thêm một số thứ khi nói đến các thuộc tính controller đặc biệt. Trong trường hợp này, các mảng giá trị `AppController` được merged với các mảng trong controller con. Các giá trị trong lớp con sẽ luôn ghi đè các giá trị trong `AppController`.

> CakePHP sẽ merge các biến sau từ `AppController` vào các controller của ứng dụng:
> - $components
> - $helpers
> - $uses

### 1.1.2 Request parameters
Khi một request được thực hiện đến một ứng dụng CakePHP, lớp `Cake\Routing\Router` và `Cake\Routing\Dispatcher` sử dụng `Routes Configuration` để tìm và tạo ra một instance controller. Dữ liệu request được đóng gói trong một đối tượng request. CakePHP đặt tất cả các thông tin request quan trọng vào thuộc tính `$this->request`.

### 1.1.3 Controller Actions
Controller thực hiện chuyển các tham số request thành response cho browser/user thực hiện request. CakePHP sử dụng các quy tắc để tự động hóa quá trình này. Theo quy tắc, CakePHP sinh ra một view có tên là tên của action.

Controller action thường sử dụng `Controller::set()` để tạo ra `View`. Theo các quy tắc của CakePHP, không cần phải tạo và render view thủ công. Khi controller action được hoàn thành, CakePHP sẽ xử lý rendering và delivering View.

Khi sử dụng các phương thức trong controller với `requestAction()`, ta thường muốn trả về dữ liệu không phải dạng string. Nếu các phương thức controller được sử dụng cho các request web thông thường, ta nên kiểm tra trước khi trả về

```
class RecipesController extends AppController {
    public function popular() {
        $popular = $this->Recipe->popular();
        if (!empty($this->request->params['requested'])) {
            return $popular;
        }
        $this->set('popular', $popular);
    }
}
```

Ví dụ trên biểu diễn cách thức một phương thức có thể được sử dụng với `requestAction()` và các request thông thường.

## 1.2 Components
Component là các gói logic được chia sẻ giữa các controller. CakePHP có một tập các core component mà ta có thể sử dụng để hỗ trợ các tác vụ khác nhau. Ta cũng có thể tạo các component riêng. Nếu muốn copy và paste nhiều thứ giữa các controller với nhau, ta nên tạo ra component riêng để chứa các hàm. Tạo component làm cho code của controller "sạch" và cho phép tái sử dụng code giữa các controller.

Các component có trong CakePHP:

- Authentication
- Cookie
- Flash
- Security
- Pagination
- Request Handling
- Access Control Lists
- Sessions

### 1.2.1 Cấu hình component
Nhiều core component yêu cầu phải cấu hình, ví dụ Authentication và Cookie. Cấu hình cho các component này, và các component nói chung, thường thực hiện thông qua `beforeFilter()`hoặc thông qua mảng $components:

```
class PostsController extends AppController {
    public $components = array(
        'Auth' => array(
            'authorize' => array('controller'),
            'loginAction' => array(
                'controller' => 'users',
                'action' => 'login'
            )
        ),
        'Cookie' => array('name' => 'CookieMonster')
    );
```

`beforeFilter()` hữu ích khi ta cần gán các kết quả của một hàm cho một thuộc tính component.

```
public function beforeFilter() {
    $this->Auth->authorize = array('controller');
    $this->Auth->loginAction = array(
        'controller' => 'users',
        'action' => 'login'
    );

    $this->Cookie->name = 'CookieMonster';
}
```

### 1.2.2 Sử dụng Components
Khi include các component vào trong controller thì sử dụng chúng khá đơn giản. Mỗi component sử dụng được exposed như thuộc tính trong controller. Nếu ta tải `SessionComponent` và `CookieComponent` trong controller, ta có thể truy cập như sau:

```
class PostsController extends AppController {
    public $components = array('Session', 'Cookie');

    public function delete() {
        if ($this->Post->delete($this->request->data('Post.id'))) {
            $this->Session->setFlash('Post deleted.');
            return $this->redirect(array('action' => 'index'));
        }
    }
```

### 1.2.3 Tạo một Component
Giả sử ứng dụng cần phải thực hiện phép toán phức tạp trong nhiều phần khác nhau của ứng dụng. Ta có thể tạo một component lưu logic này để sử dụng trong nhiều controller khác.

Đầu tiên là tạo một file component và class. Tạo file trong `app/Controller/Component/MathComponent.php`. Cấu trúc cơ bản cho component như sau:

```
App::uses('Component', 'Controller');

class MathComponent extends Component {
    public function doComplexOperation($amount1, $amount2) {
        return $amount1 + $amount2;
    }
}
```

#### Include component vào controller
Ta có thể sử dụng component trong controller bằng cách đặt tên của component trong mảng `$components`. Controller sẽ tự động đưa một thuộc tính mới được đặt tên theo component, qua đó ta có thể truy cập vào một instance của nó:

```
/* Make the new component available at $this->Math,
as well as the standard $this->Session */
public $components = array('Math', 'Session');
```

Các component được khai báo trong `AppController` sẽ được merge với các component trong controller khác. Vì vậy không cần phải khai báo lại một component hai lần.

Khi include Component trong Controller, ta cũng có thể khai báo một tập các tham số sẽ được đưa vào constructor của Component. Các tham số này sau đó có thể được xử lý bởi Component:

```
public $components = array(
    'Math' => array(
        'precision' => 2,
        'randomGenerator' => 'srand'
    ),
    'Session', 'Auth'
);
```

### 1.2.4 Sử dụng Component khác trong Component
Thỉnh thoảng một vài component cần phải sử dụng component khác. Trong trường hợp này ta có thể include component khác, sử dụng biến $components.

```
// app/Controller/Component/CustomComponent.php
App::uses('Component', 'Controller');

class CustomComponent extends Component {
    // the other component your component uses
    public $components = array('Existing');

    public function initialize(Controller $controller) {
        $this->Existing->foo();
    }

    public function bar() {
        // ...
   }
}

// app/Controller/Component/ExistingComponent.php
App::uses('Component', 'Controller');

class ExistingComponent extends Component {

    public function foo() {
        // ...
    }
}
```

## 1.3 Model
Model là các lớp tạo nên lớp business trong ứng dụng. Chúng có trách nhiệm quản lý hầu hết những gì liên quan đến dữ liệu, tính hợp lệ và tương tác của dữ liệu.

Thông thường, các lớp model mô tả dữ liệu được sử dụng trong các ứng dụng CakePHP để truy cập dữ liệu. Chúng thường đại diện cho một bảng cơ sở dữ liệu nhưng có thể được sử dụng để truy cập vào bất cứ thứ gì điều khiển dữ liệu như file, các dịch vụ web bên ngoài.

Một model có thể được liên kết với các model khác. Ví dụ: Bài Post có thể liên kết với Author hoặc Comment.

Đây là một ví dụ đơn giản về model trong CakePHP:

```
App::uses('AppModel', 'Model');
class Ingredient extends AppModel {
    public $name = 'Ingredient';
}
```

Với khai báo này, model Ingredient sẽ cung cấp tất cả các phương thức ta cần để tạo truy vấn, lưu và xóa dữ liệu. Các phương thức này có từ Model của CakePHP bằng kỹ thuật kế thừa. AppModel là lớp core Model, cung cấp phương thức cho model Ingredient. `App::uses('AppModel', 'Model')` đảm bảo rằng model được load khi cần thiết.

Việc ghi đè AppModel cho phép ta định nghĩa các hàm cần có cho tất cả các Model trong ứng dụng. Để làm được như thế ta cần tạo file `AppModel.php` nằm trong thư mục Model, cũng như tất cả các Model khác. 

Khi Model được định nghĩa, có thể dùng controller để truy cập. Một controller có tên IngredientsController sẽ tự động khởi tạo model Ingredient và gắn nó vào controller ở `$this->Ingredient`

```
class IngredientsController extends AppController {
    public function index() {
        //grab all ingredients and pass it to the view:
        $ingredients = $this->Ingredient->find('all');
        $this->set('ingredients', $ingredients);
    }
}
```

Các Model quan hệ liên kết với model chính. Trong ví dụ sau Recipe có một liên kết với model Ingredient

```
class Recipe extends AppModel {

    public function steakRecipes() {
        $ingredient = $this->Ingredient->findByName('Steak');
        return $this->findAllByMainIngredient($ingredient['Ingredient']['id']);
    }
}
```

## 1.4 Configuration
Thư mục *Config* chứa các file cấu hình mà CakePHP sử dụng như: Kết nối cơ sở dữ liệu, bootstrapping, file core,...

### 1.4.1 Database configuration
Cấu hình database của CakePHP ở file `app/Config/database.php`. Một ví dụ về config database:

```
class DATABASE_CONFIG {
    public $default = array(
        'datasource'  => 'Database/Mysql',
        'persistent'  => false,
        'host'        => 'localhost',
        'login'       => 'cakephpuser',
        'password'    => 'c4k3roxx!',
        'database'    => 'my_cakephp_project',
        'prefix'      => ''
    );
}
```

Việc đặt tên cũng rất quan trọng. Ví dụ nếu có bảng big_boxes, model là BigBox, controller là BigBoxesController thì tất cả sẽ hoạt động tự động.

### 1.4.2 Thêm Class Path
Ta có thể chia sẻ các class MVC giữa các ứng dụng trên cùng một hệ thống. Nếu muốn có cùng controller trong cả hai ứng dụng, ta có thể sử dụng `bootstrap.php` của CakePHP để đưa các class bổ sung này vào view.

Sử dụng `App::build()` trong `bootstrap.php` để định nghĩa các đường dẫn bổ sung mà CakePHP sẽ tìm các class.

```
App::build(array(
    'Model' => array(
        '/path/to/models',
        '/next/path/to/models'
    ),
    'Model/Behavior' => array(
        '/path/to/behaviors',
        '/next/path/to/behaviors'
    ),
    'Model/Datasource' => array(
        '/path/to/datasources',
        '/next/path/to/datasources'
    ),
    'Model/Datasource/Database' => array(
        '/path/to/databases',
        '/next/path/to/database'
    ),
    'Model/Datasource/Session' => array(
        '/path/to/sessions',
        '/next/path/to/sessions'
    ),
    'Controller' => array(
        '/path/to/controllers',
        '/next/path/to/controllers'
    ),
    'Controller/Component' => array(
        '/path/to/components',
        '/next/path/to/components'
    ),
    'Controller/Component/Auth' => array(
        '/path/to/auths',
        '/next/path/to/auths'
    ),
    'Controller/Component/Acl' => array(
        '/path/to/acls',
        '/next/path/to/acls'
    ),
    'View' => array(
        '/path/to/views',
        '/next/path/to/views'
    ),
    'View/Helper' => array(
        '/path/to/helpers',
        '/next/path/to/helpers'
    ),
    'Console' => array(
        '/path/to/consoles',
        '/next/path/to/consoles'
    ),
    'Console/Command' => array(
        '/path/to/commands',
        '/next/path/to/commands'
    ),
    'Console/Command/Task' => array(
        '/path/to/tasks',
        '/next/path/to/tasks'
    ),
    'Lib' => array(
        '/path/to/libs',
        '/next/path/to/libs'
    ),
    'Locale' => array(
        '/path/to/locales',
        '/next/path/to/locales'
    ),
    'Vendor' => array(
        '/path/to/vendors',
        '/next/path/to/vendors'
    ),
    'Plugin' => array(
        '/path/to/plugins',
        '/next/path/to/plugins'
    ),
));
```

### 1.4.3 Core Configuration
#### CakePHP Core Configuration
Class `Configure` được sử dụng để quản lý một tập các biến cấu hình CakePHP core. Các biến này nằm ở trong `app/Config/core.php`, bao gồm: debug, Error, Exception, App.baseURL, App.fullBaseUrl,...

#### Core Cache Configuration
CakePHP sử dụng hai cấu hình bộ nhớ cache. `_cake_model` và `_cake_core`. `_cake_core` được dùng để lưu đường dẫn file và vị trí object. `_cake_model` được sử dụng để lưu trữ mô tả schema và các danh sách source cho datasources. Khuyến nghị sử dụng bộ nhớ cache nhanh APC hoặc Memcached vì chúng đọc được trên mọi request.

Ta có thể xóa dữ liệu lưu trong cache bằng `Cache::clear()`.

### 1.4.4 Configure Class
`class` Configure

Class Configure của CakePHP có thể được sử dụng để lưu trữ và truy xuất các giá trị của ứng dụng hoặc các giá trị runtime. Hãy cẩn thận vì class này cho phép ta lưu trữ bất cứ thứ gì, và sử dụng nó ở bất kỳ đâu trong code: có thể dẫn đến việc phá vỡ mô hình MVC của CakePHP. Mục tiêu chính của Configure class là giữ các biến tập trung để có thể chia sẻ giữa nhiều object.

### 1.4.5 Đọc và ghi file configuration
CakePHP có hai kiểu đọc file cấu hình. `PhpReader` dùng để đọc file config PHP. `IniReader` được dùng để đọc file config ini.

```
App::uses('PhpReader', 'Configure');
// Read config files from app/Config
Configure::config('default', new PhpReader());

// Read config files from another path.
Configure::config('default', new PhpReader('/path/to/your/config/files/'));
```

#### Load file cấu hình
`static` Configure::load($key, $config = 'default', $merge = true)

Tham số:

- $key (string): Định danh của file config để load.
- $config (string): Alias của configred reader
- $merge (boolean): Có hay không nội dung của file đọc được merge, hoặc ghi đè lên các giá trị hiện có

```
// Load my_file.php using the 'default' reader object.
Configure::load('my_file', 'default');
```

Thiết lập `$merge = true`, giá trị sẽ không bao giờ ghi đè lên cấu hình hiện tại.

#### Tạo hoặc sửa file configuration
`static` Configure::dump($key, $config = 'default', $keys = array())

Tham số:
- $key (string): Tên của file cấu hình được tạo
- $config (string): Tên của reader để lưu dữ liệu
- $keys (array): Danh sách các key trên cùng để lưu. Mặc định cho tất cả các key.

Dumps tất cả hoặc một vài dữ liệu trong Configure ra một file hoặc hệ thống lưu trữ được hỗ trợ bởi một config reader. Ví dụ nếu 'default' là một `PhpReader`, file được tạo ra sẽ là một file PHP config có thể load được bởi `PhpReader`.

`Configure::dump('my_config.php', 'default');`

Chỉ lưu cấu hình xử lý error:

`Configure::dump('error.php', 'default', array('Error', 'Exception'));`

`Configure::dump()` có thể được sử dụng để sửa hoặc ghi đè file config có thể đọc được với `Configure::load()`.

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

Đây là form cơ bản của Bootstrap: các file được biên dịch sẵn để sử dụng nhanh chóng trong các web project. Nó cung cấp CSS và JS (`bootstrap.\*`), (`bootstrap.min.\*`). CSS **source maps** (`bootstrap.\*.map`) có sẵn để sử dụng với các công cụ phát triển của browser. File bundled JS (`bootstrap.bundle.js` và minified `bootstrap.bundle.min.js`) bao gồm **Popper**, nhưng không có **jQuery**.

## 2.2 Các điểm nổi bật của bootstrap 4
### 2.2.1 Chuyển đổi từ LESS sang SASS
CSS Preprocessor là một ngôn ngữ kịch bản mở rộng của CSS và được biên dịch thành cú pháp CSS giúp ta viết CSS nhanh hơn và có cấu trúc rõ ràng hơn. CSS Preprocessor có thể giúp ta tiết kiệm thời gian viết CSS, dễ dàng bảo trì và phát triển CSS,...

SASS là một CSS Preprocessor cung cấp thêm các quy tắc như nested rule, variable, mixin, ... Với SASS ta có thể viết CSS theo thứ tự rõ ràng, quản lý các biến đã được định nghĩa sẵn, có thể tự động nén tập tin CSS.

Bootstrap bây giờ biên dịch rất nhanh nhờ Libsass, và tham gia vào cộng đồng phát triển SASS.

### 2.2.2 Cải tiến hệ thống Grid
Bootstrap 3 hiện tại có 4 dạng grid dành cho cột, đó là `.col-xs-`, `.col-sm-`, `.col-md-`, `.col-lg-`. Bootstrap 4 đã chỉnh lại và giới thiệu thêm dạng grid thứ 5 là `.col-xl-` giúp xây dựng layout hoạt động tốt hơn trên tất cả các thiết bị.

### 2.2.3 Hỗ trợ Opt-in flexbox
Chuyển đổi biến boolean `enable-flex` trong file `_variables.scss` và biên dịch lại CSS để thấy sự tiện dụng của các thành phần và hệ thống grid sử dụng flexbox.

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
- Nâng cấp hero
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

Em thấy 2 chức năng **Arena** và **Battle Connect** là hay hơn cả. Vì:

- **Arena** mình được đấu với người chơi thật, mang lại cảm thấy mới mẻ, thú vị, không nhàm chán như đi đánh quái.
- **Battle Connet** mình có thể kết hợp với hai người chơi khác để đi đánh boss kiếm tiền và vật phẩm. Boss nhìn cũng khá hùng vĩ.

## Đánh giá về game
Em thấy game chơi hay. Hình ảnh đồ họa đẹp mắt, nhân vật dễ thương. Hiệu ứng kỹ năng nhân vật hoành tráng, đa dạng. Game có nhiều tính năng thú vị, hấp dẫn. Event trong game phong phú, giá trị.

Nhưng có một điểm em không biết vì sao khi connect vào game thì rất khó, phải connect tới vài lần mới vào được game. Nhiều lúc em cảm thấy hơi khó chịu.
