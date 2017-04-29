# Qt library to help easily test DBus services

A simple library to create a private DBus environment for your test fixtures.

DBus is the most common RPC middleware used in Linux desktop shell development. Unfortunately it does not provide much support for testing services using it. I created [libqtdbustest](https://github.com/pete-woods/libqtdbustest) and [libqtdbusmock](https://github.com/pete-woods/libqtdbusmock) to fill this gap, for Qt/C++ based services, at least.

# A private bus

The first thing you need when testing DBus based services is your own private pair of DBus servers. When you instantiate the `QtDBusTest::DBusTestRunner` class, this is exactly what you get.

It has a zero argument constructor, so can easily be composed into your test fixture. It starts a new set of bus daemons, and terminates them on destrution.

```cpp
class TestMyStuff: public ::testing::Test {
protected:
  QtDBusTest::DBusTestRunner dbus_;
}
```

It provides methods to access the new instances of `QDBusConnection` for the private session and system buses:

```cpp
TEST_F(TestMyStuff, DoesSomething) {
  dbus_.sessionConnection();
  dbus_.systemConnection();
}
```

# Starting services

You can register DBus services to be started and waited for. These services will be terminated when the test runner object is deleted.

```cpp
class TestMyStuff: public ::testing::Test {
protected:
  TestMyStuff() {
    dbus_.registerService(
        DBusServicePtr(new QProcessDBusService("org.freedesktop.MyBusName",
            QDBusConnection::SessionBus, "/path/to/executable/,
            QStringList{argument1, argument2})));

    dbus_.startServices();
  }

  QtDBusTest::DBusTestRunner dbus_;
}
```
