footer: *A clean architecture for QML applications* - Marco Piccolino
slidenumbers: true
<!--autoscale: true-->

![inline 100%](https://www.datocms-assets.com/962/1495099293-logo-sito.png)

## A clean architecture <br/>for QML applications
### Marco Piccolino - Maply
### *marco.a.piccolino@gmail.com*

---

![](http://unsplash.com/photos/JRVxgAkzIsM/download?force=true)
## QML is awesome!

A set of building blocks for your GUI

* object-oriented
* declarative
* property bindings
* javascript syntax
* garbage collection
* wrap C++ classes and objects

---

![](http://unsplash.com/photos/JRVxgAkzIsM/download?force=true)
## Only for GUIs?

What's the difference between these two imports?

```qml
import QtQuick 2.9
import QtQml 2.2
```

<br/><br/><br/><br/>
Did you ever use the second?

---

![](http://unsplash.com/photos/VEOk8qUl9DU/download?force=true)
## You can write a full application in QML
<br/>
Are the features that we listed on slide 2 <br/>specific to GUIs?

---

![](http://unsplash.com/photos/VEOk8qUl9DU/download?force=true)
## Does it make sense to write a full application in QML?
<br/>
It depends.

---

![](http://unsplash.com/photos/m7zKB91brGo/download?force=true)

## Sometimes it does.

* Thin clients
* Thicker clients with QML-wrapped C++ libraries
* Cases where JS scripting amounts to "glue code"

<br/><br/><br/><br/><br/>
Newer Qt libraries expose very rich QML APIs <br/>(e.g. Qt3D, see Peppe's talk)

---

![](http://unsplash.com/photos/45m01-tUIJY/download?force=true)

## But... QML is too pleasant, <br/>it tricks you!

<br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/></br>
If you don't care about your QML architecture, prepare for flying-spaghetti-binding madness!

---

## Think clean

![inline](https://8thlight.com/blog/assets/posts/2012-08-13-the-clean-architecture/CleanArchitecture-5c6d7ec787d447a81b708b73abba1680.jpg)

*Credits and more info: https://tinyurl.com/cleanarch*

---

![](http://unsplash.com/photos/jMpLgHLeXZQ/download?force=true)

## The details do not matter, <br/>as long as your team is in control

---

## What works for me

* __Behaviour-driven development__ for usecases
* Model __usecases__ in code, test-driven with QtTest
* Model __entities__ after usecases, if not already given
* Create __test doubles__ for data repositories, caches etc.
* In parallel, work on __GUI components__ library (*styleguide* - see [tinyurl.com/PiccolinoQtDay16](https://tinyurl.com/PiccolinoQtDay16))
* Connect usecases, entities and GUI components into __client app__

---

## A simple example
### QtDay 2017 app

* browse talks
* check talk details
* check venue information
