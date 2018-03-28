# CakePHP
## Controllers
Controller là 'C' trong MVC. Thông thường một controller được sử dụng để quản lý logic xung quanh một model. Trong CakePHP, controller được đặt tên theo model mà nó xử lý.

Các Controller được extends từ class `AppController`. `AppController` có thể được định nghĩa trong **src/Controller/AppController** và nó chứa các phương thức được chia sẻ giữa tất cả các controller với nhau.

Controller cung cấp một số phương thức để xử lý các request được gọi là *action*. Theo mặc định, mỗi phương thức public trong một controller là một action, và có thể truy cập từ một URL. Một action phải xử lý request và tạo ra response.

### App Controller
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

### Request Flow
Khi một request được thực hiện đến một ứng dụng CakePHP, lớp `Cake\Routing\Router` và `Cake\Routing\Dispatcher` sử dụng `Connecting Route` để tìm và tạo ra một instance controller. Dữ liệu request được đóng gói trong một đối trượng request. CakePHP đặt tất cả các thông tin request quan trọng vào thuộc tính `$this->request`.

### Controller Actions
Controller thực hiện chuyển các tham số request thành response cho browser/user thực hiện request. CakePHP sử dụng các quy ước để tự động hóa quá trình này. Theo quy ước, CakePHP sinh ra một view có tên là tên của action.

Controller action thường sử dụng `Controller::set()` để tạo ra `View`. Do các quy ước của CakePHP, không cần phải tạo và render view thủ công. Khi controller action được hoàn thành, CakePHP sẽ xử lý rendering và delivering View.

### Tương tác với View
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

### Chuyển trang
`Cake\Controller\Cake\Controller\Controller::redirect(string|array $url, integer $status)`

Phương thức điều khiển luồng hay sử dụng nhất là `Controller::redirect()`. Tham số đầu tiên là đường dẫn URL tương đối hoặc tuyệt đối. Tham số thứ 2 là trạng thái HTTP, ví dụ 301, 303 tùy thuộc vào tình huống.

#### Chuyển hướng đến một action khác trogn cùng một Controller
`Cake\Controller\Controller::setAction($action, $args...)`

Nếu cần chuyển tiếp từ action hiện tại đến một action khác trên cùng một controller, ta có thể sử dụng `Controller::setAction()` để cập nhật request, view được sửa và thực hiện chuyển tiếp đến action được đặt tên:

```
// From a delete action, you can render the updated
// list page.
$this->setAction('index');
```

## Components
Component là các gói logic được chia sẻ giữa các controller. CakePHP có một tập các core component mà ta có thể sử dụng để hỗ trợ các tác vụ khác nhau. Ta cũng có thể tạo các component riêng. Nếu muốn copy và paste nhiều thứ giữa các controller với nhau, ta nên tạo ra component riêng để chứa các hàm. Tạo component làm cho controller code "sạch" và cho phép sử dụng lại code giữa các controller.

Các component có trong CakePHP:

- Authentication
- Cookie
- Cross Site Request Forgery
- Flash
- Security
- Pagination
- Request Handling

### Cấu hình component
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

### Sử dụng Components
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

### Sử dụng Component khác trong Component
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

### Truy cập Controller của Components
Từ trong một component ta có thể truy cập đến controller hiện tại thông qua registry:

`$controller = $this->_registry->getController();`

Ta có thể truy cập controller trong bất kỳ phương thức callback nào từ đối tượng event:

`$controller = $event->getSubject();`

### Component callbacks
Component cung cấp một số request callbacks để tăng chu kỳ request
