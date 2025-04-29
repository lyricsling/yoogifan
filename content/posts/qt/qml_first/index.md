+++
date = '2025-04-15T10:49:54+08:00'
draft = true
title = 'Qml_first'
+++
## qml 与 c++ 的数据交互
Q_PROPERTY宏的使用
```cpp
class MyData : public QObject
{
    Q_OBJECT
    Q_PROPERTY(QString message READ message WRITE setMessage NOTIFY messageChanged FINAL)
};
```

使用 qmlRegisterType 函数，注册到qml中
```
int main()
{
	qmlRegisterType<MyData>("MyApp", 1, 0, "MyData");
}
```

或者注册实例给engine
```
int main()
{
	MyData myData;
	engine.rootContext()->setContextProperty("myData", &myData);
}
```

在qml端
```qml
import MyApp 1.0

Window {
    width: 640
    height: 480
    visible: true
    title: qsTr("Hello World")

    MyData {
         id: myData
         message: "Hello from QML"
    }

    Text {
        text: myData.message
    }
}
```
message: "Hello from QML"就会调用 setMessage() 方法

## 使用Q_INVOKABLE
```cpp
class MyData : public QObject {
    Q_OBJECT

public:
    explicit MyData(QObject *parent = nullptr) : QObject(parent) {}

    Q_INVOKABLE void sayHello(const QString &name) {
        qDebug() << "Hello from C++," << name;
    }
};
```
在 qml 中就可以使用 sayHello 方法了
```qml
Button {
    text: "Say Hello"
    onClicked: myData.sayHello("QML")
}
```

