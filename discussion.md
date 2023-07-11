Hello everyone,

I hope you're doing well. I'm currently facing an issue regarding Routing Stay Signed In using Nylo. I have been trying to find a solution, but I haven't had any luck so far. I'm reaching out to this community in the hopes that someone can provide some guidance, help, or information regarding this matter.

I've managed to create different initial routes depending on whether the user is logged in or not. Figure 1 is the initial route when the user is not logged in, and Figure 2 is the initial route when the user is logged in. The problem that arises is that when I press the back button when the user is logged in, as shown in Figure 2, the screen returns to the initial route when the user is not logged in (Figure 1). What I want is that when the user is logged in, the user can immediately exit the application by pressing the back button without going through the initial route (Figure 1) first.

| Figure 1 | Figure 2 |
| ----------- | ----------- |
| ![figure-1](https://raw.githubusercontent.com/degalih/stunt-shield-docs/main/Figure%201.png) | ![figure-2](https://raw.githubusercontent.com/degalih/stunt-shield-docs/main/Figure%202.png) |

| Video 1 | Video 2 |
| ----------- | ----------- |
| ![video-1](https://github.com/degalih/stunt-shield-docs/blob/main/Video%201.gif?raw=true) | ![video-2](https://github.com/degalih/stunt-shield-docs/blob/main/Video%202.gif?raw=true) |

### main.dart
```dart
import 'package:flutter/material.dart';
import '/bootstrap/app.dart';
import 'package:nylo_framework/nylo_framework.dart';
import 'bootstrap/boot.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  Nylo nylo = await Nylo.init(setup: Boot.nylo, setupFinished: Boot.finished);

  runApp(
    AppBuild(
      navigatorKey: NyNavigator.instance.router.navigatorKey,
      onGenerateRoute: nylo.router!.generator(),
      debugShowCheckedModeBanner: false,
      initialRoute: nylo.getInitialRoute(),
    ),
  );
}
```

### router.dart
```dart
import 'package:flutter/material.dart';

import '/resources/pages/recipe_page.dart';
import '/resources/pages/register_page.dart';
import '/resources/pages/login_page.dart';
import '/resources/pages/web_view_page.dart';

import '/resources/pages/home_page.dart';
import 'package:nylo_framework/nylo_framework.dart';

appRouter() => nyRoutes((router) async {
      router.route(HomePage.path, (context) => HomePage());
      router.route(WebViewPage.path, (context) => WebViewPage());
      router.route(LoginPage.path, (context) => LoginPage());
      router.route(RegisterPage.path, (context) => RegisterPage());
      router.route(
        RecipePage.path,
        (context) => RecipePage(),
        routeGuards: [AuthRouteGuard()],
        authPage: true,
      );
    });

class AuthRouteGuard extends NyRouteGuard {
  AuthRouteGuard();

  @override
  Future<bool> canOpen(BuildContext? context, NyArgument? data) async {
    return (await Auth.loggedIn());
  }

  @override
  redirectTo(BuildContext? context, NyArgument? data) async {
    await routeTo(HomePage.path);
  }
}

```

### login.dart
```dart
import 'package:flutter/gestures.dart';
import 'package:flutter/material.dart';
import 'package:flutter_app/app/models/user.dart';
import 'package:flutter_app/app/networking/api_service.dart';
import 'package:flutter_app/bootstrap/helpers.dart';
import 'package:flutter_app/resources/pages/recipe_page.dart';
import 'package:flutter_app/resources/pages/register_page.dart';
import 'package:flutter_app/resources/themes/text_theme/default_text_theme.dart';
import 'package:flutter_app/resources/widgets/safearea_widget.dart';
import 'package:flutter_svg/flutter_svg.dart';
import 'package:lottie/lottie.dart';
import 'package:nylo_framework/nylo_framework.dart';
import '/app/controllers/controller.dart';

class LoginPage extends NyStatefulWidget {
  final Controller controller = Controller();

  static const path = '/login';

  LoginPage({Key? key}) : super(key: key);

  @override
  _LoginPageState createState() => _LoginPageState();
}

class _LoginPageState extends NyState<LoginPage> {
  bool isPasswordVisible = false;
  TextEditingController _emailController = TextEditingController();
  TextEditingController _passwordController = TextEditingController();

  @override
  init() async {
    super.init();
    isPasswordVisible = true;
  }

  @override
  void dispose() {
    _emailController.clear();
    _passwordController.clear();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeAreaWidget(
        child: SingleChildScrollView(
          child: Container(
            margin: EdgeInsets.all(20.0),
            child: Column(
              children: <Widget>[
                Row(
                  children: <Widget>[
                    GestureDetector(
                      child: Icon(
                        Icons.arrow_back_ios,
                        color: ThemeColor.get(context).black,
                      ),
                      onTap: () {
                        Navigator.pop(context);
                      },
                    ),
                    SizedBox(
                      width: 16.0,
                    ),
                    Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: <Widget>[
                        Text(
                          'Selamat Datang Kembali!',
                          style: defaultTextTheme.labelLarge!
                              .copyWith(fontWeight: FontWeight.w600),
                        ),
                        Text(
                          'Ayo Masuk',
                          style: defaultTextTheme.bodySmall!
                              .copyWith(color: ThemeColor.get(context).grey600),
                        )
                      ],
                    )
                  ],
                ),
                SizedBox(
                  height: 32.0,
                ),
                SvgPicture.network(
                  'https://res.cloudinary.com/stunt-shield-cloudinary/image/upload/v1688356447/Stunt%20Shield%20App%20Assets/login-illustration_epvqju.svg',
                  placeholderBuilder: (BuildContext context) => Container(
                    padding: const EdgeInsets.all(30.0),
                    child: const CircularProgressIndicator(),
                  ),
                ),
                SizedBox(
                  height: 32.0,
                ),
                SizedBox(
                  height: 44.0,
                  child: TextField(
                    cursorHeight: 20.0,
                    controller: _emailController,
                    textAlignVertical: TextAlignVertical.top,
                    style: defaultTextTheme.labelLarge,
                    decoration: InputDecoration(
                      enabledBorder: OutlineInputBorder(
                        borderRadius: BorderRadius.circular(4.0),
                        borderSide:
                            BorderSide(color: ThemeColor.get(context).grey400),
                      ),
                      focusedBorder: OutlineInputBorder(
                        borderRadius: BorderRadius.circular(4.0),
                        borderSide:
                            BorderSide(color: ThemeColor.get(context).green),
                      ),
                      labelText: 'Email',
                      floatingLabelStyle: defaultTextTheme.bodySmall!
                          .copyWith(color: ThemeColor.get(context).green),
                      labelStyle: defaultTextTheme.labelLarge!
                          .copyWith(color: ThemeColor.get(context).grey400),
                    ),
                  ),
                ),
                SizedBox(
                  height: 16.0,
                ),
                SizedBox(
                  height: 44.0,
                  child: TextField(
                    keyboardType: TextInputType.visiblePassword,
                    textInputAction: TextInputAction.done,
                    obscureText: isPasswordVisible,
                    cursorHeight: 20.0,
                    controller: _passwordController,
                    textAlignVertical: TextAlignVertical.top,
                    style: defaultTextTheme.labelLarge,
                    decoration: InputDecoration(
                      suffixIcon: IconButton(
                        icon: Icon(isPasswordVisible
                            ? Icons.visibility_outlined
                            : Icons.visibility_off_outlined),
                        color: ThemeColor.get(context).grey400,
                        onPressed: () {
                          setState(() {
                            isPasswordVisible = !isPasswordVisible;
                          });
                        },
                      ),
                      enabledBorder: OutlineInputBorder(
                        borderRadius: BorderRadius.circular(4.0),
                        borderSide:
                            BorderSide(color: ThemeColor.get(context).grey400),
                      ),
                      focusedBorder: OutlineInputBorder(
                        borderRadius: BorderRadius.circular(4.0),
                        borderSide:
                            BorderSide(color: ThemeColor.get(context).green),
                      ),
                      labelText: 'Kata Sandi',
                      floatingLabelStyle: defaultTextTheme.bodySmall!
                          .copyWith(color: ThemeColor.get(context).green),
                      labelStyle: defaultTextTheme.labelLarge!
                          .copyWith(color: ThemeColor.get(context).grey400),
                    ),
                  ),
                ),
                SizedBox(height: 16.0),
                isLoading(name: 'loadLogin')
                    ? ElevatedButton.icon(
                        icon: Container(
                          width: 24,
                          height: 24,
                          padding: const EdgeInsets.all(2.0),
                          child: const CircularProgressIndicator(
                            color: Colors.white,
                            strokeWidth: 3,
                          ),
                        ),
                        style: ElevatedButton.styleFrom(
                          elevation: 2.0,
                          minimumSize: const Size.fromHeight(50),
                          shape: RoundedRectangleBorder(
                            borderRadius: BorderRadius.circular(8.0),
                          ),
                        ),
                        onPressed: () {
                          _login(
                              _emailController.text, _passwordController.text);
                        },
                        label: Text(
                          'Loading...',
                          style: defaultTextTheme.labelLarge,
                        ),
                      )
                    : ElevatedButton(
                        style: ElevatedButton.styleFrom(
                          elevation: 2.0,
                          minimumSize: const Size.fromHeight(50),
                          shape: RoundedRectangleBorder(
                            borderRadius: BorderRadius.circular(8.0),
                          ),
                        ),
                        onPressed: () {
                          _login(
                              _emailController.text, _passwordController.text);
                        },
                        child: Text(
                          'Masuk',
                          style: defaultTextTheme.labelLarge,
                        ),
                      ),
                SizedBox(height: 16.0),
                Row(
                  crossAxisAlignment: CrossAxisAlignment.center,
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: <Widget>[
                    Padding(
                      padding: EdgeInsets.symmetric(horizontal: 10.0),
                      child: Container(
                        height: 1.0,
                        width: 80.0,
                        color: ThemeColor.get(context).grey600,
                      ),
                    ),
                    Text('atau'),
                    Padding(
                      padding: EdgeInsets.symmetric(horizontal: 10.0),
                      child: Container(
                        height: 1.0,
                        width: 80.0,
                        color: ThemeColor.get(context).grey600,
                      ),
                    ),
                  ],
                ),
                SizedBox(height: 16.0),
                ElevatedButton.icon(
                  onPressed: () {
                    showModalBottomSheet(
                      context: context,
                      builder: (context) {
                        return Container(
                          margin: const EdgeInsets.all(16.0),
                          clipBehavior: Clip.antiAlias,
                          decoration: BoxDecoration(
                            borderRadius:
                                const BorderRadius.all(Radius.circular(16.0)),
                          ),
                          child: Column(
                            mainAxisSize: MainAxisSize.min,
                            children: <Widget>[
                              Container(
                                height: 4.0,
                                width: 100.0,
                                decoration: BoxDecoration(
                                  color: ThemeColor.get(context).grey400,
                                  borderRadius: const BorderRadius.all(
                                      Radius.circular(16.0)),
                                ),
                              ),
                              Padding(
                                padding: const EdgeInsets.all(20.0),
                                child: Column(
                                  mainAxisSize: MainAxisSize.min,
                                  children: [
                                    Lottie.network(
                                        'https://res.cloudinary.com/stunt-shield-cloudinary/raw/upload/v1688393467/Stunt%20Shield%20App%20Assets/under-construction-ilus_ywftja.json'),
                                    Text(
                                      'Fitur masih dalam proses pengerjaan',
                                      textAlign: TextAlign.center,
                                      style: defaultTextTheme.titleLarge!
                                          .copyWith(
                                              color: ThemeColor.get(context)
                                                  .grey600),
                                    ),
                                  ],
                                ),
                              ),
                            ],
                          ),
                        );
                      },
                    );
                  },
                  icon: SvgPicture.network(
                    'https://res.cloudinary.com/stunt-shield-cloudinary/image/upload/v1688390287/Stunt%20Shield%20App%20Assets/google-logo_gtegfo.svg',
                    placeholderBuilder: (BuildContext context) => Container(
                      padding: const EdgeInsets.all(30.0),
                      child: const CircularProgressIndicator(),
                    ),
                  ),
                  label: Text(
                    'Masuk dengan Google',
                    style: defaultTextTheme.titleSmall,
                  ),
                  style: ElevatedButton.styleFrom(
                    backgroundColor: ThemeColor.get(context).white,
                    foregroundColor: ThemeColor.get(context).grey600,
                    surfaceTintColor: ThemeColor.get(context).white,
                    elevation: 4.0,
                    minimumSize: const Size.fromHeight(50),
                    shape: RoundedRectangleBorder(
                      borderRadius: BorderRadius.circular(8.0),
                    ),
                  ),
                ),
                SizedBox(height: 16.0),
                RichText(
                  textAlign: TextAlign.center,
                  text: TextSpan(
                    style: defaultTextTheme.bodySmall,
                    children: <TextSpan>[
                      TextSpan(
                        style: TextStyle(
                          color: ThemeColor.get(context).grey600,
                        ),
                        text: 'Belum punya akun? ',
                      ),
                      TextSpan(
                          style:
                              TextStyle(color: ThemeColor.get(context).green),
                          text: 'Daftar ',
                          recognizer: TapGestureRecognizer()
                            ..onTap = () {
                              routeTo(
                                RegisterPage.path,
                                navigationType: NavigationType.pushReplace,
                              );
                            }),
                    ],
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }

  Future<void> _login(String identifier, String password) async {
    await validate(
      rules: {"email address": "email", "Password": "not_empty"},
      data: {"email address": identifier, "Password": password},
      onSuccess: () async {
        setLoading(true, name: 'loadLogin');
        User? user = await api<ApiService>(
            (request) => request.fetchUser(identifier, password),
            context: context);
        await Auth.set(user);
        routeTo(RecipePage.path,
            navigationType: NavigationType.pushAndForgetAll);
        setLoading(false, name: 'loadLogin');
      },
    );
  }
}

```

### Figure 1 Page Code
```dart
import 'package:flutter/gestures.dart';
import 'package:flutter/material.dart';
import 'package:flutter_app/app/models/web_view_target.dart';
import 'package:flutter_app/bootstrap/extensions.dart';
import 'package:flutter_app/bootstrap/helpers.dart';
import 'package:flutter_app/resources/pages/login_page.dart';
import 'package:flutter_app/resources/pages/register_page.dart';
import 'package:flutter_app/resources/pages/web_view_page.dart';
import 'package:flutter_app/resources/themes/text_theme/default_text_theme.dart';
import 'package:flutter_app/resources/widgets/logo_widget.dart';
import 'package:flutter_svg/flutter_svg.dart';
import '/app/controllers/home_controller.dart';
import '/resources/widgets/safearea_widget.dart';
import 'package:nylo_framework/nylo_framework.dart';

class HomePage extends NyStatefulWidget {
  HomePage({Key? key}) : super(key: key);

  static const path = '/';

  @override
  final HomeController controller = HomeController();

  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends NyState<HomePage> {
  bool _darkMode = false;

  @override
  init() async {
    super.init();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeAreaWidget(
        child: SingleChildScrollView(
          child: Center(
            child: Padding(
              padding: const EdgeInsets.all(16.0),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.center,
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  Logo(),
                  SvgPicture.network(
                    'https://res.cloudinary.com/stunt-shield-cloudinary/image/upload/v1687707417/Stunt%20Shield%20App%20Assets/toodler-ilus_moqji2.svg',
                    placeholderBuilder: (BuildContext context) => Container(
                      padding: const EdgeInsets.all(30.0),
                      child: const CircularProgressIndicator(),
                    ),
                  ),
                  SizedBox(height: 32),
                  Text('Selamat Datang!').displayMedium(context),
                  Text("Bergabunglah dengan kami sekarang buat akun atau masuk",
                          textAlign: TextAlign.center)
                      .bodySmall(context)
                      .setColor(
                          context,
                          (color) =>
                              _darkMode == true ? color.grey50 : color.grey600),
                  SizedBox(
                    height: 32.0,
                  ),
                  ElevatedButton(
                    style: ElevatedButton.styleFrom(
                        elevation: 2.0,
                        minimumSize: const Size.fromHeight(50),
                        shape: RoundedRectangleBorder(
                            borderRadius: BorderRadius.circular(8.0))),
                    onPressed: () {
                      routeTo(LoginPage.path);
                    },
                    child: Text(
                      'Masuk',
                      style: defaultTextTheme.labelLarge,
                    ),
                  ),
                  SizedBox(height: 16.0),
                  OutlinedButton(
                    style: OutlinedButton.styleFrom(
                        minimumSize: const Size.fromHeight(50),
                        shape: RoundedRectangleBorder(
                            borderRadius: BorderRadius.circular(8.0))),
                    onPressed: () {
                      routeTo(RegisterPage.path);
                    },
                    child: Text(
                      'Daftar',
                      style: defaultTextTheme.labelLarge,
                    ),
                  ),
                  SizedBox(
                    height: 16.0,
                  ),
                  RichText(
                    textAlign: TextAlign.center,
                    text: TextSpan(
                      style: defaultTextTheme.bodySmall,
                      children: <TextSpan>[
                        TextSpan(
                          style:
                              TextStyle(color: ThemeColor.get(context).grey600),
                          text: 'Dengan mendaftar, Anda menyetujui ',
                        ),
                        TextSpan(
                            style:
                                TextStyle(color: ThemeColor.get(context).green),
                            text: 'Perjanjian Pengguna ',
                            recognizer: TapGestureRecognizer()
                              ..onTap = () {
                                WebViewTarget data = new WebViewTarget();
                                data.name = "Perjanjian Pengguna";
                                data.url =
                                    "https://github.com/degalih/stunt-shield-docs/blob/main/user-aggrement-id.md";
                                routeTo(WebViewPage.path, data: data);
                              }),
                        TextSpan(
                            style: TextStyle(
                                color: ThemeColor.get(context).grey600),
                            text: 'dan '),
                        TextSpan(
                            style:
                                TextStyle(color: ThemeColor.get(context).green),
                            text: 'Kebijakan Privasi ',
                            recognizer: TapGestureRecognizer()
                              ..onTap = () {
                                WebViewTarget data = new WebViewTarget();
                                data.name = "Kebijakan Privasi";
                                data.url =
                                    "https://github.com/degalih/stunt-shield-docs/blob/main/privacy-policy-id.md";
                                routeTo(WebViewPage.path, data: data);
                              }),
                        TextSpan(
                            style: TextStyle(
                                color: ThemeColor.get(context).grey600),
                            text: 'kami'),
                      ],
                    ),
                  )
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}

```

### Figure 2 Page Code
```dart
import 'package:flutter/material.dart';
import 'package:flutter_app/app/models/user.dart';
import 'package:flutter_app/resources/pages/calculator_page.dart';
import 'package:flutter_app/resources/pages/favorite_page.dart';
import 'package:flutter_app/resources/pages/food_page.dart';
import 'package:flutter_app/resources/pages/profile_page.dart';
import 'package:flutter_app/resources/widgets/safearea_widget.dart';
import 'package:nylo_framework/nylo_framework.dart';
import 'package:salomon_bottom_bar/salomon_bottom_bar.dart';
import '/app/controllers/controller.dart';
import 'search_page.dart';

class RecipePage extends NyStatefulWidget {
  final Controller controller = Controller();

  static const path = '/recipe';

  RecipePage({Key? key}) : super(key: key);

  @override
  _RecipePageState createState() => _RecipePageState();
}

class _RecipePageState extends NyState<RecipePage> {
  int _currentIndex = 0;

  final List<Widget> _listWidget = <Widget>[
    FoodPage(),
    SearchPage(),
    CalculatorPage(),
    FavoritePage(),
    ProfilePage()
  ];

  @override
  init() async {
    super.init();
    if (await Auth.loggedIn() == true) {
      User? user = await Auth.user<User>();
      print(user);
    }
  }

  @override
  void dispose() {
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeAreaWidget(
        child: Container(
          child: Column(
            children: [
              Text('Halo!'),
              _listWidget[_currentIndex],
            ],
          ),
        ),
      ),
      bottomNavigationBar: SalomonBottomBar(
        itemPadding: EdgeInsets.all(16.0),
        currentIndex: _currentIndex,
        onTap: (index) {
          setState(() {
            _currentIndex = index;
          });
        },
        items: [
          SalomonBottomBarItem(
            icon: Icon(Icons.home),
            title: Text('Resep'),
          ),
          SalomonBottomBarItem(
            icon: Icon(Icons.search),
            title: Text('Cari'),
          ),
          SalomonBottomBarItem(
            icon: Icon(Icons.calculate),
            title: Text('Hitung'),
          ),
          SalomonBottomBarItem(
            icon: Icon(Icons.bookmark),
            title: Text('Favorit'),
          ),
          SalomonBottomBarItem(
            icon: Icon(Icons.person),
            title: Text('Profil'),
          ),
        ],
      ),
    );
  }
}

```

I would greatly appreciate any assistance or insights you can provide. If you have any relevant resources, tutorials, or references that could help me, please feel free to share them as well.

Thank you in advance for your time and support. I'm looking forward to hearing from you and learning from your experiences.

Best regards,
Galih
