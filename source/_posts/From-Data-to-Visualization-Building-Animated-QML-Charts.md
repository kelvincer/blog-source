---
title: 'How to Animate Charts in QML: Heart Rate and Pace Visualization'
tags:
  - QML
  - Qt 6
  - Animation
  - Chart
  - PropertyAnimation
  - ParallelAnimation
  - Repeater
categories:
  - Animation
  - Chart
date: 2025-03-24 16:00:20
---

{% video <iframe width="560" height="315" src="https://www.youtube.com/embed/onlKNx6fNJc?rel=0&amp;showinfo=0" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe> %}
<br/>

On this post I want to replicate an animation from [this page](https://developer.apple.com/tutorials/swiftui/animating-views-and-transitions), it uses SwiftUI. I use basic animations from QML, for instance <mark>PropertyAnimation</mark> and <mark>ParallelAnimation</mark> to animate properties such as <mark>color</mark>, <mark>height</mark>, <mark>y</mark> and <mark>x</mark>.  Additionally I use <mark>Repeater</mark> to dynamically create any number of lines for chart. You can find the full project [here](https://github.com/kelvincer/HeartPaceAnimation). Below you can find the main files.
```qml
import QtQuick
import QtQuick.Controls.iOS
import QtQuick.Layouts

Window {
    width: 640
    height: 480
    visible: true
    title: qsTr("From Data to Visualization: Building Animated QML Charts")

    property int iDuration: 400
    property int interval: 60

    property var chartData :
        [{duration: iDuration, height: 20, y: 170, x: 10, newY: 170, fHeight: 20, paceY: 10, sHeight: 20}
        , {duration: iDuration + interval, height: 25, y: 163, x: 40, newY: 163, fHeight: 25, paceY: 20, sHeight: 25 }
        , {duration: iDuration + interval * 2, height: 20, y: 160, x: 70, x: 70, newY: 160, fHeight: 20, paceY: 20, sHeight: 20}
        , {duration: iDuration + interval * 3, height: 20, y: 150, x: 100, newY: 150, fHeight: 20, paceY: 23, sHeight: 30}
        , {duration: iDuration + interval * 4, height: 25, y: 140, x: 130, newY: 140, fHeight: 25, paceY: 23, sHeight: 35}
        , {duration: iDuration + interval * 5, height: 20, y: 150, x: 160, newY: 150, fHeight: 20, paceY: 10, sHeight: 20}
        , {duration: iDuration + interval * 6, height: 20, y: 150, x: 190, newY: 150, fHeight: 20, paceY: 20, sHeight: 45}
        , {duration: iDuration + interval * 7, height: 100, y: 50, x: 220, newY: 50, fHeight: 100, paceY: 90, sHeight: 75}
        , {duration: iDuration + interval * 8, height: 30, y: 40, x: 250, newY: 40, fHeight: 35, paceY: 50, sHeight: 85}
        , {duration: iDuration + interval * 9, height: 30, y: 45, x: 280, newY: 75, fHeight: 50, paceY: 10, sHeight: 20}
        , {duration: iDuration + interval * 10, height: 80, y: 100, x: 310, newY: 140, fHeight: 20, paceY: 10, sHeight: 25}
        , {duration: iDuration + interval * 11, height: 20, y: 170, x: 340, newY: 120, fHeight: 20, paceY: 10, sHeight: 20}]

    property int rowSpacing: 10
    property int lineWidth: 20
    property int graphWidth: lineWidth * chartData.length + rowSpacing * (chartData.length + 1)

    ColumnLayout {
        anchors.fill: parent
        spacing: 10

        Item {
            Layout.fillWidth: true
            Layout.fillHeight: true
        }

        Item {
            height: 200

            Rectangle {
                id: chart
                width: graphWidth
                height: 200
                color: "dodgerblue"
                x: 640

                Repeater {
                    id: repeater
                    model: chartData

                    RoundedLine {
                        required property var modelData
                        property alias line: line
                        property alias parallelAnim: parallelAnim
                        property alias paceAnim: paceAnim

                        id: line
                        recHeight: modelData.height
                        recY: modelData.y
                        recX: modelData.x

                        ParallelAnimation {
                            id: parallelAnim

                            PropertyAnimation {
                                target: line
                                property: "color"
                                to: "red"
                                duration: modelData.duration
                            }

                            PropertyAnimation {
                                target: line
                                property: "y"
                                easing.type: Easing.OutBounce
                                to: modelData.newY
                                duration: modelData.duration
                            }

                            PropertyAnimation {
                                target: line
                                property: "height"
                                easing.type: Easing.OutBounce
                                to: modelData.fHeight
                                duration: modelData.duration
                            }
                        }

                        ParallelAnimation {
                            id: paceAnim

                            PropertyAnimation {
                                target: line
                                property: "color"
                                to: "mediumturquoise"
                                duration: modelData.duration
                            }

                            PropertyAnimation {
                                target: line
                                property: "y"
                                easing.type: Easing.OutBounce
                                to: modelData.paceY
                                duration: modelData.duration
                            }

                            PropertyAnimation {
                                target: line
                                property: "height"
                                easing.type: Easing.OutBounce
                                to: modelData.sHeight
                                duration: modelData.duration
                            }
                        }
                    }
                }

                PropertyAnimation {
                    id: recAnim
                    target: chart
                    property: "x"
                    easing.type: Easing.Linear
                    to: 320 - graphWidth / 2
                    duration: 500
                }
            }
        }

        Button {
            text: "Heart Rate"
            Layout.alignment: Qt.AlignHCenter
            onClicked: {
                for (let i = 0; i < repeater.count; i++) {
                    let item = repeater.itemAt(i);
                    if (item) {
                        item.line.parallelAnim.start();
                    }
                }
            }
        }

        Button {
            text: "Pace"
            Layout.alignment: Qt.AlignHCenter
            onClicked: {
                for (let i = 0; i < repeater.count; i++) {
                    let item = repeater.itemAt(i);
                    if (item) {
                        item.line.paceAnim.start();
                    }
                }
            }
        }

        Button {
            text: "Reset Animations"
            Layout.alignment: Qt.AlignHCenter
            onClicked: {
                for (let i = 0; i < repeater.count; i++) {
                    let item = repeater.itemAt(i).line;
                    if (item) {
                        item.parallelAnim.stop();
                        item.paceAnim.stop()
                        item.color = "darkgrey";
                        item.height = chartData[i].height
                        item.y = chartData[i].y
                    }
                }
            }
        }

        Button {
            text: "Show Chart"
            Layout.alignment: Qt.AlignHCenter
            onClicked: {
                recAnim.start()
            }
        }

        Item {
            Layout.fillWidth: true
            Layout.fillHeight: true
        }
    }
}
```
```qml
import QtQuick

Rectangle {

    property int recHeight: 20
    property string recColor: "darkgrey"
    property int recY: 0
    property int recX: 0

    width: 20
    y: recY
    x: recX
    height: recHeight
    color: recColor
    bottomLeftRadius: 10
    bottomRightRadius: 10
    topLeftRadius: 10
    topRightRadius: 10
}
```