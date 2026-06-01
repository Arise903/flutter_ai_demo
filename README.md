# Flutter AI 模板

基于 Flutter + GetX 的企业级应用模板，提供完整的分层架构、模块化设计和丰富的功能组件。

## 特性

### 核心功能
- **GetX 状态管理** - 简洁高效的状态管理、路由管理和依赖注入
- **Dio 网络请求** - 支持拦截器、自动重试、请求日志
- **Token 管理** - 自动处理 Token 过期和刷新
- **多语言支持** - 中文/英文双语支持
- **多级存储** - GetStorage、SharedPreferences、SecureStorage、SQLite
- **主题切换** - 支持亮色/暗色/系统主题
- **离线支持** - 离线登录和数据缓存

### 服务层（52+）
- `ApiService` - 网络请求服务
- `AuthService` - 认证服务
- `ConfigService` - 配置服务
- `StorageService` / `SecureStorageService` - 存储服务
- `DatabaseService` - SQLite 数据库
- `ThemeService` - 主题服务
- `LocaleService` - 多语言服务
- `OfflineModeService` - 离线模式
- `PushNotificationService` - 推送通知
- `CrashReportingService` - 崩溃上报
- `IAPService` - 内购服务（Apple IAP / Google Play Billing）
- `LocationService` - GPS 定位服务
- `MapService` - 地图路线规划服务（OSRM）

### 业务模块（19个）
`login` | `register` | `home` | `profile` | `settings` | `chat` | `notifications` | `search` | `debug` | `edit_profile` | `forgot_password` | `account_security` | `feedback` | `about` | `help` | `webview` | `splash` | `main` | `iap` | `map`

## 架构

```
lib/
├── main.dart
└── app/
    ├── core/                    # 核心层
    │   ├── base_controller.dart # 控制器基类
    │   ├── base_view.dart       # 视图基类
    │   ├── app_exception.dart   # 统一异常
    │   ├── result.dart          # Result 类型
    │   ├── translations/        # 多语言
    │   └── mixins/              # Mixin 工具
    ├── modules/                 # 业务模块
    │   └── [module]/
    │       ├── controllers/
    │       ├── views/
    │       ├── bindings/
    │       └── models/
    ├── services/                # 服务层（50+）
    ├── repositories/            # 数据仓储
    ├── routes/                  # 路由配置
    ├── utils/                   # 工具类
    └── widgets/                 # 通用组件
```

## 核心组件

### BaseController
```dart
class MyController extends BaseController {
  // 1. 基础用法 - 自动处理加载状态
  Future<void> loadData() async {
    await execute(() async {
      return await repository.getData();
    });
  }

  // 2. 推荐用法 - 返回 Result 类型
  Future<void> fetchData() async {
    final result = await executeResult(() => repository.getData());
    result.fold(
      onSuccess: (data) => print(data),
      onFailure: (e, s) => showErrorSnackBar(e.toString()),
    );
  }

  // 3. 自动处理 UI 反馈
  Future<void> saveData() async {
    await executeWithResult(
      () => repository.save(data),
      onSuccess: (_) => showSuccessSnackBar('保存成功'),
    );
  }

  // 4. 静默模式 - 不显示加载和错误
  Future<void> preload() async {
    await executeSilently(() => repository.preload());
  }
}
```

### AppException 统一异常
```dart
// 异常类型
enum ErrorType {
  network,     // 网络错误
  auth,        // 认证错误
  business,    // 业务错误
  system,      // 系统错误
  validation,  // 验证错误
  permission,  // 权限错误
  notFound,    // 资源不存在
}

// 使用
try {
  await api.getUser();
} catch (e) {
  if (e is AppException) {
    print('${e.type}: ${e.message}');
  }
}
```

### Result 类型
```dart
// Success / Failure / Loading 三种状态
final result = await executeResult(() => api.getUser());

result.fold(
  onSuccess: (user) => updateUI(user),
  onFailure: (error, stack) => handleError(error),
  onLoading: () => showLoading(),
);
```

## 快速开始

```bash
# 安装依赖
flutter pub get

# 运行应用
flutter run
```

### 测试账号（Debug 模式自动填充）
| 账号 | 密码 |
|------|------|
| admin | admin123 |
| user | user123 |

## API 示例

```dart
// 获取服务
final api = Get.find<ApiService>();

// GET 请求
final response = await api.get('/users');

// POST 请求
final response = await api.post('/users', data: {'name': 'test'});

// 文件上传
final response = await api.uploadFile('/upload', file);

// 取消请求
api.cancelRequest('/users');
```

## 存储服务

```dart
// 通用存储
final storage = Get.find<StorageService>();
await storage.setString('key', 'value');
String? value = storage.getString('key');

// 安全存储（Token 等敏感数据）
final secure = Get.find<SecureStorageService>();
await secure.saveSecureString('token', 'xxx');
String? token = await secure.getSecureString('token');

// SQLite
final db = Get.find<DatabaseService>();
await db.execute('CREATE TABLE users (id INTEGER PRIMARY KEY)');
```

## 路由

```dart
// 导航
Get.toNamed(Routes.home);
Get.offNamed(Routes.login);
Get.offAllNamed(Routes.splash);

// 带参数
Get.toNamed(Routes.chatDetail, arguments: {'chatId': '123'});

// 获取参数
final args = Get.arguments as Map<String, dynamic>;
```

## 内购（IAP）

### 功能概述
- **Apple IAP** - iOS 应用内购买
- **Google Play Billing** - Android 应用内购买
- 支持消耗型、非消耗型、自动续订、非续订商品
- 完整的购买状态管理
- 订阅状态监控

### 文件结构
```
lib/app/
├── domain/entities/iap_entity.dart    # 内购实体和枚举
├── services/iap_service.dart          # 内购核心服务
└── modules/iap/                       # 内购模块
    ├── controllers/iap_controller.dart
    ├── views/iap_view.dart
    └── bindings/iap_binding.dart
```

### 核心类

#### IAPService
```dart
// 获取服务实例
final iapService = Get.find<IAPService>();

// 检查内购是否可用
bool available = await iapService.isAvailable();

// 加载商品列表
await iapService.loadProducts();

// 获取商品列表
List<IAPProduct> products = iapService.products;

// 获取已购买列表
List<IAPPurchase> purchases = iapService.purchases;

// 检查订阅状态
bool hasSubscription = iapService.hasActiveSubscription();

// 检查是否拥有某商品
bool owns = await iapService.isOwned('product_id');
```

#### IAPController
```dart
// 获取控制器
final iapController = Get.find<IAPController>();

// 监听状态变化
iapController.purchaseStatus.listen((status) {
  // 处理状态变化
});

// 查看内购状态
List<IAPStatusInfo> statusInfo = iapController.getStatusInfo();

// 购买商品
await iapController.purchase('product_id');

// 购买消耗型商品
await iapController.purchaseConsumable('product_id');

// 恢复购买
await iapController.restorePurchases();
```

### 状态枚举 (IAPPurchaseStatus)
| 状态 | 说明 |
|------|------|
| `idle` | 初始状态 |
| `loadingProducts` | 正在加载商品 |
| `productsLoaded` | 商品加载完成 |
| `purchasing` | 购买中 |
| `purchased` | 购买成功 |
| `failed` | 购买失败 |
| `canceled` | 用户取消 |
| `expired` | 已过期 |
| `restoring` | 恢复中 |
| `restored` | 恢复成功 |
| `verifying` | 验证中 |
| `verified` | 验证成功 |

### 使用示例

#### 1. 导航到内购页面
```dart
// 方式一：使用路由
Get.toNamed(Routes.iap);

// 方式二：直接跳转
Get.to(() => const IAPView(), binding: IAPBinding());
```

#### 2. 在代码中使用内购服务
```dart
// 检查内购是否可用
final iapService = Get.find<IAPService>();
if (await iapService.isAvailable()) {
  // 加载商品
  await iapService.loadProducts();
  
  // 显示商品列表
  for (final product in iapService.products) {
    print('${product.title}: ${product.localizedPrice}');
  }
}
```

#### 3. 购买商品
```dart
final iapService = Get.find<IAPService>();

// 购买非消耗型商品
final result = await iapService.purchase('com.example.premium');
if (result.success) {
  print('购买成功');
  // 向服务器验证购买凭证
} else {
  print('购买失败: ${result.error?.message}');
}
```

#### 4. 恢复购买
```dart
final iapService = Get.find<IAPService>();
final result = await iapService.restorePurchases();
if (result.success) {
  print('恢复了 ${result.data?.length ?? 0} 个购买');
}
```

#### 5. 监听购买状态
```dart
// 监听状态变化
iapService.purchaseStatus.listen((status) {
  switch (status) {
    case IAPPurchaseStatus.purchased:
      // 购买成功处理
      break;
    case IAPPurchaseStatus.failed:
      // 购买失败处理
      break;
    case IAPPurchaseStatus.canceled:
      // 用户取消处理
      break;
    // ... 其他状态
  }
});
```

### 配置商品 ID
在 `lib/app/services/iap_service.dart` 中配置你的商品 ID：

```dart
static const List<String> _productIds = [
  // 消耗型商品
  'com.example.coins_100',     // 100金币
  'com.example.coins_500',     // 500金币
  
  // 非消耗型商品
  'com.example.remove_ads',   // 移除广告
  
  // 订阅商品
  'com.example.premium_monthly',   // 月度高级版
  'com.example.premium_yearly',   // 年度高级版
];
```

### 平台配置

#### iOS (App Store Connect)
1. 在 App Store Connect 创建内购项目
2. 配置商品 ID 和价格
3. 在 Xcode 中启用内购功能
4. 添加沙盒测试账号

#### Android (Google Play Console)
1. 在 Google Play Console 创建内购项目
2. 配置商品 ID 和价格
3. 发布应用到内测/正式渠道
4. 添加测试账号

### 商品类型说明
| 类型 | 说明 | 适用场景 |
|------|------|----------|
| `consumable` | 消耗型 | 金币、道具等一次性使用 |
| `nonConsumable` | 非消耗型 | 永久功能、解锁内容 |
| `autoRenewable` | 自动续订 | 会员订阅 |
| `nonRenewable` | 非续订 | 时限会员 |

### 注意事项
1. **必须先调用 `isAvailable()`** - 检查内购是否可用
2. **服务器验证** - 生产环境应在服务器端验证购买凭证
3. **沙盒测试** - 开发时使用沙盒账号进行测试
4. **订阅续订** - 定期检查订阅状态，及时通知用户续订

## 地图导航（免费方案）

### 功能概述
- **OpenStreetMap** - 完全免费，无需 API Key
- **多地图样式** - 标准/深色/卫星/骑行主题
- **实时定位** - GPS 位置追踪
- **路线规划** - 步行/骑行/驾车导航
- **海拔信息** - 路线海拔剖面图
- **距离计算** - 任意两点间距离
- **地点搜索** - 搜索和反向地理编码

### 文件结构
```
lib/app/
├── services/
│   ├── location_service.dart    # GPS 定位服务
│   └── map_service.dart         # 地图路线服务
└── modules/map/                # 地图模块
    ├── controllers/
    │   └── map_controller.dart
    ├── views/
    │   └── map_view.dart
    ├── bindings/
    │   └── map_binding.dart
    └── widgets/
        ├── search_panel.dart        # 搜索面板
        ├── map_controls_widget.dart # 控制按钮
        ├── route_info_panel.dart   # 路线信息
        └── elevation_profile_widget.dart # 海拔图表
```

### 核心类

#### LocationService
```dart
// 获取服务实例
final locationService = Get.find<LocationService>();

// 检查权限
bool hasPermission = await locationService.checkPermission();

// 请求权限
bool granted = await locationService.requestPermission();

// 获取当前位置
LocationData? location = await locationService.getCurrentLocation();

// 开始位置追踪
await locationService.startLocationTracking();

// 停止位置追踪
await locationService.stopTracking();

// 计算两点间距离
double distance = locationService.calculateDistanceBetween(start, end);

// 格式化距离显示
String distanceStr = DistanceCalculator.formatDistance(meters); // "1.5 公里"
```

#### MapService
```dart
// 获取服务实例
final mapService = Get.find<MapService>();

// 规划步行路线
RouteResult? route = await mapService.planWalkingRoute(start, end);

// 规划骑行路线
route = await mapService.planCyclingRoute(start, end);

// 规划驾车路线
route = await mapService.planDrivingRoute(start, end);

// 搜索地点
List<SearchResult> results = await mapService.searchPlace('天安门');

// 反向地理编码
SearchResult? address = await mapService.reverseGeocode(point);

// 获取海拔高度
double? elevation = await mapService.getElevation(point);

// 获取路线海拔剖面
List<ElevationPoint> profile = await mapService.getElevationProfile(routePoints);

// 计算海拔统计
RouteElevationStats? stats = mapService.calculateElevationStats(profile);
```

#### MapController
```dart
// 获取控制器
final mapController = Get.find<MapController>();

// 初始化地图
await mapController.initMap();

// 定位到当前位置
await mapController.locateCurrentPosition();

// 搜索地点
await mapController.searchPlace('北京大学');

// 选择搜索结果
await mapController.selectSearchResult(result);

// 从当前位置导航
await mapController.navigateFromCurrentLocation(destination);

// 规划路线
await mapController.planRoute();

// 切换导航模式
mapController.switchNavigationMode(NavigationMode.walking);
mapController.switchNavigationMode(NavigationMode.cycling);
mapController.switchNavigationMode(NavigationMode.driving);

// 切换地图样式
mapController.switchMapType(MapType.standard);
mapController.switchMapType(MapType.dark);
mapController.switchMapType(MapType.satellite);

// 调整视野显示完整路线
mapController.fitRoute();

// 清除路线
mapController.clearRoute();

// 开始/停止导航
mapController.startNavigation();
mapController.stopNavigation();
```

### 地图类型 (MapType)
| 类型 | 说明 |
|------|------|
| `standard` | OpenStreetMap 标准地图 |
| `dark` | 深色主题（夜间使用） |
| `satellite` | 卫星样式地图 |
| `cycling` | 骑行友好地图（HOT 主题）|

### 导航模式 (NavigationMode)
| 模式 | 说明 | 适用场景 |
|------|------|----------|
| `walking` | 步行导航 | 户外徒步 |
| `cycling` | 骑行导航 | 自行车出行 |
| `driving` | 驾车导航 | 汽车导航 |

### 使用示例

#### 1. 导航到地图页面
```dart
// 使用路由
Get.toNamed(Routes.map);
```

#### 2. 搜索并导航到目的地
```dart
final mapController = Get.find<MapController>();

// 搜索地点
await mapController.searchPlace('北京站');

// 从当前位置开始导航
await mapController.navigateFromCurrentLocation(destinationLatLng);
```

#### 3. 自定义地图样式
```dart
// 切换到深色主题
mapController.switchMapType(MapType.dark);

// 切换到卫星地图
mapController.switchMapType(MapType.satellite);
```

#### 4. 获取当前位置和距离
```dart
final locationService = Get.find<LocationService>();

// 获取当前位置
final location = await locationService.getCurrentLocation();
if (location != null) {
  print('当前位置: ${location.latitude}, ${location.longitude}');
  print('海拔: ${location.altitude}m');
  print('速度: ${location.speed} m/s');
}

// 计算距离
final distance = DistanceCalculator.calculateDistance(
  LatLng(39.9042, 116.4074), // 北京
  LatLng(31.2304, 121.4737), // 上海
);
print('北京到上海的距离: ${DistanceCalculator.formatDistance(distance)}');
// 输出: 北京到上海的距离: 1088.54 公里
```

#### 5. 显示路线海拔信息
```dart
final mapController = Get.find<MapController>();

// 规划路线后，海拔信息自动获取
await mapController.navigateFromCurrentLocation(destination);

// 显示海拔统计
final stats = mapController.elevationStats.value;
if (stats != null) {
  print('最高海拔: ${stats.maxElevation}m');
  print('最低海拔: ${stats.minElevation}m');
  print('累计爬升: ${stats.totalAscent}m');
  print('累计下降: ${stats.totalDescent}m');
}
```

### 依赖说明
本模块使用以下免费服务：
- **OpenStreetMap** - 地图瓦片（无需 API Key）
- **OSRM** - 路线规划 API（公共服务器）
- **Nominatim** - 地理编码服务（公共服务器）
- **Open-Elevation** - 海拔数据 API（公共服务器）

### 权限配置

#### iOS (Info.plist)
```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>需要获取您的位置以提供导航服务</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>需要获取您的位置以提供导航服务</string>
```

#### Android (AndroidManifest.xml)
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

### 注意事项
1. **免费 API 限制** - 公共 OSRM 服务器有请求频率限制，生产环境建议部署私有服务器
2. **海拔数据** - 部分区域可能缺少海拔数据
3. **网络连接** - 路线规划和地图瓦片需要网络连接
4. **定位精度** - GPS 定位精度受设备和环境影响

## 配置

### 环境变量
```bash
# .env
API_BASE_URL=https://api.example.com
API_TIMEOUT=30000
```

### 添加翻译
```dart
// lib/app/core/translations/app_translations.dart
'zh_CN': {
  'welcome': '欢迎',
},
'en_US': {
  'welcome': 'Welcome',
},
```

### 新增模块
```bash
# 使用 get_cli
get create page:new_module
```

## 依赖

| 包 | 用途 |
|----|------|
| get | 状态管理/路由/依赖注入 |
| dio | HTTP 客户端 |
| flutter_secure_storage | 安全存储 |
| sqflite | 本地数据库 |
| firebase_* | 推送/崩溃上报 |
| google_sign_in | Google 登录 |
| in_app_purchase | Apple/Google 内购 |
| ... | 详见 pubspec.yaml |

## License

MIT
