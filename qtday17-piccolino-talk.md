footer: QtDay 2017 - *A clean architecture for QML applications* - Marco Piccolino - marco.a.piccolino@gmail.com
slidenumbers: true
<!--autoscale: true-->

![inline 100%](https://www.datocms-assets.com/962/1495099293-logo-sito.png)

## A clean architecture <br/>for QML applications
### Marco Piccolino - Maply
*marco.a.piccolino@gmail.com*
*@marco_piccolino*

---

![](http://unsplash.com/photos/JRVxgAkzIsM/download?force=true)
## QML is awesome!

"A set of building blocks for your GUI"

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
* Cases where WorkerScripts can be used
* Programmers skillset

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

## The exact details do not matter, <br/><br/>as long as your team is in control

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

Possible usecases:

* browse talks
* filter talks by author
* check talk details
* check venue information
* ...

---

## Usecases: gherkin files

```gherkin
Feature: browse_talks

Scenario: browse_talks__one_or_more
  Given there_is_at_least_one_talk
  When i_browse_talks
  Then i_am_given_a_list_of_talks
  And the_list_of_talks_cointains_at_least_one_talk

```

... and so on for all usecases (features) and scenarios

---

## Usecases: project structure

```
qtday2017-app [template: subdirs]
  usecases [template: app]*
    Steps.qml [step definitions] + qmldir
    tst_usecases.cpp [tests entry point]
    tst_browse_talks.qml [usecase tests]
    ...
    usecases.pri [usecase implementations]
```

\*[tinyurl.com/quicktests](https://tinyurl.com/quicktests)

---

## Usecases: scenario test/1

```qml
import QtQml 2.2
import QtTest 1.0
import "."

TestCase {
    name: "browse_talks" // test name; useful for output and AutoTest plugin

    property var usecases
    Component {
        id: usecasesC
        QtObject {  // usecase implementation references
            property var browse_talks: BrowseTalks {}
        }
    }

    function initTestCase() {Steps.testcase = this} // pass testcase env. to Steps
    function cleanupTestCase() {Steps.testcase = null}
```
---

## Usecases: scenario test/2

```qml
    function init() { // called before each scenario
        browse_talks = browseTalksC.createObject()
    }
    function cleanup() {
        browse_talks.destroy()
    }

    function test_browse_talks__one_or_more() {
        //Given
        Steps.there_is_at_least_one_talk()
        // When
        Steps.i_browse_talks()
        // Then
        Steps.i_am_given_a_list_of_talks()
        // And
        Steps.the_list_of_talks_cointains_at_least_one_talk()
    }
}
```

---

## Usecases: step definitions/1

```qml
// usecases/Steps.qml
...
QtObject {
    property TestCase testcase
    property var entities: testcase ? testcase.entities : null
    property var usecases: testcase ? testcase.usecases : null
    function there_is_at_least_one_talk() {
        testcase.fail("implement")
    }
    function i_browse_talks() {
        testcase.fail("implement")
    }
    function i_am_given_a_list_of_talks() {
        testcase.fail("implement")
    }
    function the_list_of_talks_cointains_at_least_one_talk() {
        testcase.fail("implement")
    }
}
```

---

## Usecases: step definitions/2

```qml
QtObject {
    ...
    function there_is_at_least_one_talk() {
        testcase.fail("implement") // will deal with this later
    }
    function i_browse_talks() {
        usecases.browse_talks.run();
        usecases.browse_talks.successSignal.wait();
    }
    function i_am_given_a_list_of_talks() {
        testcase.verify(entities.talks.list);
    }
    function the_list_of_talks_cointains_at_least_one_talk() {
        testcase.verify(entities.talks.count > 0);
    }
}
```

---

## Entities: add required ones

```qml
// entities/Talks.qml
...
Object { // patched QtObject; accepts children types
    readonly property alias count: list_d.count
    readonly property alias list: list_d

    property var repo // interface for data repository

    Object {
        id: d
        ListModel { // simple implementation
            id: list_d
        }
    }
}
```

---

## Repos: add test repo/1

```qml
// repos/TalksLocal3.qml
QtObject {
    signal dataRead(var records)
    function readData() {
        dataRead([
                     {"title":"From Now to Qt 6 -
                      The Current Roadmap of Qt",
                         "author":"Artem Sidyakin"},
                     {"title":"Qt Quick Controls 2:
                      gli strumenti giusti per le interfacce embedded",
                         "author":"Luca Ottaviano"},
                     {"title":"Case Study:
                     Starting with QT - dall'idea al prodotto",
                         "author":"Edoardo Slaviero"}
                 ]);
    }
}
```

---

## Repos: add test repo/2

```qml
// tst_browse_talks.qml
...
import "../repos" as R
...
property var entities
Component {
    id: entitiesC
    QtObject { // entity implementation references
        id: entities
        property var talks: E.Talks {
            repo: R.TalksLocal3 {}
        }
        property var talksDataReadSpy: SignalSpy {
            target: entities.talks.repo
            signalName: "dataRead"
        }
    }
}
```

---

## Repos: add test repo/3

```qml
// entities/Talks.qml
...
signal wasRead(var message)
function readAll() {
    if (repo) repo.readData()
}
Connections {
    target: repo
    ignoreUnknownSignals: true
    onDataRead: {
        for (var i=0;i<records.length;i++) {
            list_d.append(records[i]);
            wasRead({});
        }
    }
}
...
```

---

## Repos: add test repo/4

```qml
// usecases/Steps.qml
...
    function there_is_at_least_one_talk() {
        var dataRead = entities.talksDataReadSpy;
        entities.talks.repo.readData();
        dataRead.wait();
        var records = dataRead.signalArguments[0][0];
        testcase.verify(records.length > 0);
    }
    ...
```

---

## Usecases: implement

```qml
// BrowseTalks.qml
Object {
    signal success(var message)
    signal failure(var message)
    function run() {
        Entities.talks.readAll(); // Entities is a register
    }
    Connections { // normally use promise-like constructs
        target: Entities.talks
        ignoreUnknownSignals: true
        onWasRead:
            if (Entities.talks.count > 0) {success({});}
            else {failure({});}
    }
}
```

---

## Client: add entities/usecases

```qml
// main.qml
ApplicationWindow {
    E.Talks {
        Component.onCompleted: E.Entities.talks = this
        repo: R.TalksLocal3 {}
    }
    U.BrowseTalks {
        id: browse_talks // could create a register like for entities
    }
    Connections {
        // in a real app, better use a state machine for routing
        target: browse_talks
        onSuccess: stackView.push(talksOverviewPage)
    }
    ...
```

---

## Client: actions and results

```qml
// WelcomeViewer.qml
...
Button {
    onClicked: Usecases.browse_talks.run()
}
// TalksViewer.qml
...
ListView {
    model: Entities.talks.list
    delegate: ItemDelegate {
        width: parent.width
        text: "%1 %2".arg(model.author).arg(model.title)
    }
}
```
---

## Benefits

![](http://unsplash.com/photos/tUCW9zv8FaE/download?force=true)

* Test usecases in full without having to implement web, database, GUI etc.
* Bring in technologies at any time and swap them without affecting usecases or entities
* "Activate" usecases in client app effortlessly and with minimal added complexity
* Easier refactoring
* Quicker prototyping and development (QML)

---

## Costs

![](http://unsplash.com/photos/DSeHNMrevFA/download?force=true)

* At present, requires some boilerplate code and knowing qmake's peculiarities
* "Project startup" cost, which is however compensated when bugfixing and adding features
* Potential performance costs (QML)

---

## QML only?

![](http://unsplash.com/photos/qQMKgH_tMcw/download?force=true)

* No! It can all be done with Qt's C++ APIs as well!
* Or, mixing QML and C++ (QVariantMap is your friend)
* Or, prototyping with the former and deploying the latter

---

## Resources

![](http://unsplash.com/photos/6GjHwABuci4/download?force=true)

__Code__
[tinyurl.com/qtday17app](https://tinyurl.com/qtday17app)

__Clean Architecture book__ by Uncle Bob (sept. 2017)
[tinyurl.com/CleanArchBook](https://tinyurl.com/CleanArchBook)

__QtMob Slack__ (\#architecture)
[slackin.qtmob.org](http://slackin.qtmob.org)

__JSON Model for QML__ (by Cutehacks)
[github.com/Cutehacks/gel](https://github.com/Cutehacks/gel)

---
