# 1. Widget

1. 什么是widget

    在官网中的解释如下

    > Flutter widgets are built using a modern framework that takes inspiration from [React](https://reactjs.org/). The central idea is that you build your UI out of widgets. Widgets describe what their view should look like given their current configuration and state. When a widget’s state changes, the widget rebuilds its description, which the framework diffs against the previous description in order to determine the minimal changes needed in the underlying render tree to transition from one state to the next.

    

# 2. Flex(弹性布局)

```dart
class FlexDemo extends StatelessWidget {
  const FlexDemo({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Flex Demo"),
      ),
      body: Container(
        color: Colors.green,
        child: Flex(		//Flex布局
          direction: Axis.vertical,	//方向，纵轴
          children: [
            Expanded(				//可伸缩,
              child: Container(
                color: Colors.red,
                width: 100,
                height: 100,
              ),
              flex: 1,				//弹性布局中的组件占比
            ),
            Expanded(
              child: Container(
                color: Colors.black38,
                width: 150,
                height: 100,
              ),
              flex: 1,
            ),
            Expanded(
              child: Container(
                color: Colors.yellow,
                width: 100,
                height: 200,
              ),
              flex: 1,
            ),
          ],
        ),
      ),
    );
  }
}
```

<img src="https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/202111082213034.png" alt="image-20211108221259906" style="zoom:50%;" />

# 3. [Wrap](https://api.flutter.dev/flutter/widgets/Wrap-class.html)

A widget that displays its children in multiple horizontal or vertical runs.

A [Wrap](https://api.flutter.dev/flutter/widgets/Wrap-class.html) lays out each child and attempts to place the child adjacent to the previous child in the main axis, given by [direction](https://api.flutter.dev/flutter/widgets/Wrap/direction.html), leaving [spacing](https://api.flutter.dev/flutter/widgets/Wrap/spacing.html) space in between. If there is not enough space to fit the child, [Wrap](https://api.flutter.dev/flutter/widgets/Wrap-class.html) creates a new *run* adjacent to the existing children in the cross axis.

After all the children have been allocated to runs, the children within the runs are positioned according to the [alignment](https://api.flutter.dev/flutter/widgets/Wrap/alignment.html) in the main axis and according to the [crossAxisAlignment](https://api.flutter.dev/flutter/widgets/Wrap/crossAxisAlignment.html) in the cross axis.

The runs themselves are then positioned in the cross axis according to the [runSpacing](https://api.flutter.dev/flutter/widgets/Wrap/runSpacing.html) and [runAlignment](https://api.flutter.dev/flutter/widgets/Wrap/runAlignment.html).

```dart
class WrapDemo extends StatefulWidget {			//有状态widget
  const WrapDemo({Key? key}) : super(key: key);

  @override
  _WrapDemoState createState() => _WrapDemoState();
}

class _WrapDemoState extends State<WrapDemo> {
  List<int> list = [];

  @override
  void initState() {
    super.initState();
    for (var i = 0; i < 20; i++) {
      list.add(i);				
    }
  }

  @override
  Widget build(BuildContext context) {
    Random random = Random();
    return Scaffold(
      appBar: AppBar(
        title: Text("WrapText"),
        centerTitle: true,
      ),
      body: Wrap(
        // direction: Axis.vertical,
        alignment: WrapAlignment.start,
        spacing: 10,		//左右间距
        runSpacing: 5,		//上下间距
        children: list		//list先map出对应的widget,然后再转化为对应的list
            .map(
              (e) => Container(
                height: 20,
                width: 100.0 * (0.5 + random.nextDouble()),
                child: Center(
                  child: Text(e.toString()),
                ),
                color: Color.fromARGB(
                  255,
                  random.nextInt(200),
                  random.nextInt(200),
                  random.nextInt(200),
                ),
              ),
            )
            .toList(),
      ),
    );
  }
}
```

<img src="https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/202111082256633.png" alt="image-20211108225656501" style="zoom:80%;" />

# 4. [Stack](https://api.flutter.dev/flutter/widgets/Stack-class.html)

A widget that positions its children relative to the edges of its box.

This class is useful if you want to overlap several children in a simple way, for example having some text and an image, overlaid with a gradient and a button attached to the bottom.

```dart
class StackDemo extends StatelessWidget {
  const StackDemo({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: Text("Stack Demo"),
        ),
        body: Container(
          width: double.infinity,		//父widget宽最大
          color: Colors.grey,
          child: Stack(
            alignment: AlignmentDirectional.center,	//居中
            children: [
              Container(
                color: Colors.green,
                width: 200,
                height: 200,
              ),
              Container(
                color: Colors.yellow,
                width: 100,
                height: 100,
              ),
              Positioned(		//这个指定了具体位置，不受alignment属性影响
                // width: 10,
                // height: 10,
                child: Container(
                  color: Colors.black,
                ),
                top: 10,
                left: 10,
                right: 300,
                bottom: 100,
              )
            ],
          ),
        ));
  }
}
```

<img src="https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/202111082301600.png" alt="image-20211108230102548" style="zoom:80%;" />

# 5. [align](https://api.flutter.dev/flutter/widgets/Align-class.html)

A widget that aligns its child within itself and optionally sizes itself based on the child's size.

For example, to align a box at the bottom right, you would pass this box a tight constraint that is bigger than the child's natural size, with an alignment of [Alignment.bottomRight](https://api.flutter.dev/flutter/painting/Alignment/bottomRight-constant.html).

```dart
class AlignDemo extends StatelessWidget {
  const AlignDemo({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("ALign demo"),
      ),
      body: Container(
        width: 200,
        height: 200,
        color: Colors.green,
        child: Align(
          alignment: Alignment(-0.8, 0.5),	//child对其哪个位置
          child: FlutterLogo(
            size: 40,
          ),
        ),
      ),
    );
  }
}
```

<img src="https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/202111082303172.png" alt="image-20211108230350125" style="zoom:80%;" />

# 6. [container](https://api.flutter.dev/flutter/widgets/Container-class.html)

A convenience widget that combines common painting, positioning, and sizing widgets.

```dart
class EdgeDemo extends StatelessWidget {
    const EdgeDemo({Key? key}) : super(key: key);

    @override
    Widget build(BuildContext context) {
        return Scaffold(
            appBar: AppBar(
                title: Text("Edge demo"),
            ),
            body: Container(
                width: 100,
                height: 100,
                color: Colors.red,
                margin: EdgeInsets.all(8), //外边距
                padding: EdgeInsets.all(20), //内边距
                child: Text("data"),
            ),
        );
    }
}
```

<img src="https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/202111082307398.png" alt="image-20211108230758312" style="zoom:80%;" />

# 7. DecoratedBox

```dart

class DecorateBoxDemo extends StatelessWidget {
  const DecorateBoxDemo({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("DecorateBox demo"),
      ),
      body: DecoratedBox(
        decoration: BoxDecoration(
          gradient: const LinearGradient(		//线性渐变
            colors: [Colors.red, Colors.blue, Colors.black],
          ),
          borderRadius: BorderRadius.circular(8.0),	//圆角
          boxShadow: [
            BoxShadow(		//阴影
              color: Colors.black26,
              offset: Offset(3.0, 10.0),
              blurRadius: 5,
            )
          ],
        ),
        child: Padding(
          padding: EdgeInsets.only(
            left: 100,
            right: 100,
            top: 20,
            bottom: 20,
          ),
          child: Text(
            "注册",
            style: TextStyle(
              color: Colors.purple,
              fontSize: 30,
            ),
            textAlign: TextAlign.center,
          ),
        ),
      ),
    );
  }
}
```

# 8. [TabBar](https://api.flutter.dev/flutter/material/TabBar-class.html)

A material design widget that displays a horizontal row of tabs.

Typically created as the [AppBar.bottom](https://api.flutter.dev/flutter/material/AppBar/bottom.html) part of an [AppBar](https://api.flutter.dev/flutter/material/AppBar-class.html) and in conjunction with a [TabBarView](https://api.flutter.dev/flutter/material/TabBarView-class.html).

```dart
import 'package:flutter/material.dart';

class PageDemo extends StatefulWidget {
  List<Widget> widgets = [
    FlutterView(),
    AndroidView(),
    IosView(),
  ];

  @override
  _PageDemoState createState() => _PageDemoState();
}

class _PageDemoState extends State<PageDemo>
    with SingleTickerProviderStateMixin {
  List tabs = ["Flutter", "Android", "IOS"];		//三个tabbar标题
  int _index = 0;
  TabController? _controller;

  @override
  void initState() {
      //固定写法，注意类要with SingleTickerProviderStateMixin
    _controller = TabController(
      initialIndex: _index,		//初始化下标bar
      length: tabs.length,
      vsync: this,
    );
    super.initState();
  }

  @override
  void dispose() {
    _controller!.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        // leading: IconButton(
        //   icon: Icon(Icons.home),
        //   onPressed: () {
        //     print("leading");
        //   },
        // ),
        actions: [		//appbar 右上角的按钮
          IconButton(
            onPressed: () {
              print("add");
            },
            icon: Icon(Icons.add),
          ),
          IconButton(
            onPressed: () {
              print("add");
            },
            icon: Icon(Icons.add_link),
          ),
        ],
        elevation: 20.0,	//appbar的阴影
        title: Text("TabBar"),
        centerTitle: true,
        bottom: TabBar(		//appbar对应的几个tabbar
          controller: _controller,
          tabs: tabs
              .map((e) => Tab(
                    text: e,
                  ))
              .toList(),
        ),
      ),
      body: TabBarView(		//每个tabar都要可点击，并且要有对应的widget
        children: widget.widgets,
        controller: _controller,	//又控制器来监听变化，并做出响应
      ),
      drawer: MyDrawer(),		//左划菜单
    );
  }
}

class FlutterView extends StatelessWidget {
  const FlutterView({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const Center(
      child: Text("Flutter"),
    );
  }
}

class AndroidView extends StatelessWidget {
  const AndroidView({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const Center(
      child: Text("Android"),
    );
  }
}

class IosView extends StatelessWidget {
  const IosView({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const Center(
      child: Text("ios"),
    );
  }
}

class MyDrawer extends StatelessWidget {
  const MyDrawer({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Drawer(
      child: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text('This is the Drawer'),
            ElevatedButton(
              onPressed: () {
                print("button");
              },
              child: const Text('Close Drawer'),
            ),
          ],
        ),
      ),
    );
  }
}
```

<img src="https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/202111082320477.png" alt="image-20211108232009080" style="zoom:80%;" />

<img src="https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/202111082320737.png" alt="image-20211108232025589" style="zoom:80%;" />

# 9. [BottomNavigationBar](https://api.flutter.dev/flutter/material/BottomNavigationBar-class.html)

底部导航栏

```dart
class BottomNavigatorBarDemo extends StatefulWidget {
    List<Widget> widgets = [
        PageDemo(),
        DecorateBoxDemo(),
        ListViewDemo(),
        DecorateBoxDemo(),
    ];

    @override
    _BottomNavigatorBarDemoState createState() => _BottomNavigatorBarDemoState();
}

class _BottomNavigatorBarDemoState extends State<BottomNavigatorBarDemo> {
    int _index = 0;

    @override
    Widget build(BuildContext context) {
        return Scaffold(
            appBar: AppBar(
                title: Text("底部选项卡"),
                centerTitle: true,
            ),
            body: widget.widgets[_index],
            floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked,
            floatingActionButton: FloatingActionButton(
                child: Icon(Icons.widgets_rounded),
                onPressed: () {
                    print("object");
                },
            ),
            // BottomAppBar 这种是使用默认的
            // bottomNavigationBar: BottomAppBar(	
            //   // color: Theme.of(context).primaryColor,
            //   color: Colors.blue,
            //   shape: CircularNotchedRectangle(),
            //   child: Row(
            //     mainAxisAlignment: MainAxisAlignment.spaceAround,
            //     children: [
            //       IconButton(
            //         onPressed: () {},
            //         icon: Icon(Icons.add),
            //       ),
            //       SizedBox(),
            //       IconButton(
            //         onPressed: () {},
            //         icon: Icon(Icons.home),
            //       ),
            //     ],
            //   ),
            // ),
            
            // BottomNavigationBar 这种是可以自定义的
            bottomNavigationBar: BottomNavigationBar(
                type: BottomNavigationBarType.fixed,
                items: [
                    BottomNavigationBarItem(
                        icon: Icon(Icons.add),
                        label: "add",
                    ),
                    BottomNavigationBarItem(
                        icon: Icon(Icons.home),
                        label: "home",
                    ),
                    BottomNavigationBarItem(
                        icon: Icon(Icons.https),
                        label: "list",
                    ),
                    BottomNavigationBarItem(
                        icon: Icon(Icons.home),
                        label: "home",
                    ),
                ],
                currentIndex: _index,
                onTap: (v) {
                    setState(() {
                        _index = v;
                    });
                },
            ),
        );
    }
}
```

<img src="https://cdn.jsdelivr.net/gh/dopamine-joker/image-host/202111082323463.png" alt="image-20211108232346163" style="zoom:80%;" />

# 9. [Scrolling widgets](https://flutter.dev/docs/development/ui/widgets/scrolling)

滚动列表，内容较多
