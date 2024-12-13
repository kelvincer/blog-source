---
title: TCP Socket Communication using C++, Qt 6 and QML
tags:
  - Qt 6
  - C++
  - QML
  - TCP Server
  - TCP Client
date: 2024-12-09 13:12:40
---


In this post we are going to see how to build a TCP socket communication. This idea came to me from the example of [Qt 6 C++ GUI Programming Cookbook](https://www.amazon.com/GUI-Programming-Cookbook-cross-platform-applications/dp/1805122630/ref=sr_1_1_sspa?__mk_es_US=%C3%85M%C3%85%C5%BD%C3%95%C3%91&crid=17E6UZ6UINEC4&dib=eyJ2IjoiMSJ9.ylfwhGsReKKyxkmegoDUeE6gMPVgE7F_9MogTpWbmXoJqp--3xCHXBTcFt3uBwUReTD6bsvsfXUlKjO8mj0ZuKQmoUqWa-cl_ItOT1PwXATDgXd86g_Tbr-msfmJdUChvKSIIDc2Dm7DkIYrnzYvfw.29JFP4ut3_vDC5yhgvyv1E7Hcx8vNmY-X3B3iMIXSZQ&dib_tag=se&keywords=qt+6+c%2B%2B&qid=1733755885&sprefix=qt+6+c%2B%2B%2Caps%2C198&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1) book. You can find this demo on chapter 7 "Using Network and  Managing Large Documents". In this chapter the client UI is built using Widgets but in this demo we are going to use QML. We will use the TCP server from the book for this demo and we will only build the TCP client.

## TCP Server
You can find the TCP server implementation in [this link](https://github.com/kelvincer/QT6-C-GUI-Programming-Cookbook---Third-Edition-/tree/main/Chapter07/Server). Furthermore, you will find the TCP client implementation and FTP implementation (which we aren't going to discuss in this article).

## TCP Client
In [this repository](https://github.com/kelvincer/QMLClient) you will find my TCP client implementation using C++ and QML.
This is the header file of TCP client implementation
```cpp
#ifndef CLIENTCONNECTION_H
#define CLIENTCONNECTION_H

#include <QObject>
#include <QQmlEngine>
#include <QString>
#include <QDebug>
#include <QTcpSocket>

class ClientConnection : public QObject
{
    Q_OBJECT
    QML_ELEMENT
    Q_PROPERTY(QString userName READ userName WRITE setUserName NOTIFY userNameChanged)
    Q_PROPERTY(QString messages READ messages WRITE setMessages NOTIFY messagesChanged)
    Q_PROPERTY(QString message READ message WRITE setMessage NOTIFY messageChanged)
    Q_PROPERTY(bool isConnected READ isConnectedToHost NOTIFY isConnectedChanged)

    QString m_userName;
    QString m_messages;
    QString m_message;
    bool connectedToHost = false;
    QTcpSocket* socket;

    QString userName();
    QString messages();
    QString message();
    void setUserName(const QString &userName);
    void setMessages(const QString &messages);
    void setMessage(const QString &message);
    void printMessage(QString message);

public:
    explicit ClientConnection(QObject *parent = nullptr);

public slots:
    void on_connectButton_clicked();
    void on_sendButton_clicked();

    void socketConnected();
    void socketDisconnected();
    void socketReadyRead();
    bool isConnectedToHost();

signals:

    void userNameChanged();
    void messagesChanged();
    void isConnectedChanged();
    void messageChanged();

};

#endif // CLIENTCONNECTION_H

```
This is the source code of TCP client implementation
```cpp
#include "clientconnection.h"

ClientConnection::ClientConnection(QObject *parent)
    : QObject{parent}
{}

QString ClientConnection::userName()
{
    return m_userName;
}

QString ClientConnection::messages()
{
    return m_messages;
}

QString ClientConnection::message()
{
    return m_message;
}

void ClientConnection::setUserName(const QString &userName)
{
    if (userName == m_userName)
        return;
    
    m_userName = userName;
    emit userNameChanged();
}

void ClientConnection::setMessages(const QString &messages)
{
    m_messages = messages;
    emit messagesChanged();
}

void ClientConnection::setMessage(const QString &message)
{
    m_message = message;
    emit messageChanged();
}

void ClientConnection::on_connectButton_clicked()
{
    if (!connectedToHost)
    {
        socket = new QTcpSocket();
        
        connect(socket, &QTcpSocket::connected, this, &ClientConnection::socketConnected);
        connect(socket, &QTcpSocket::disconnected, this, &ClientConnection::socketDisconnected);
        connect(socket, &QTcpSocket::readyRead, this, &ClientConnection::socketReadyRead);
        
        socket->connectToHost("127.0.0.1", 8001);
    }
    else
    {
        socket->write("<font color=\"Orange\">" + m_userName.toUtf8() + " has left the chat room.</font><br>");
        
        socket->disconnectFromHost();
        
        delete socket;
    }
}

void ClientConnection::on_sendButton_clicked()
{
    socket->write("<br><font color=\"Blue\">" + m_userName.toUtf8() + "</font>: " + m_message.toUtf8());
}

void ClientConnection::socketConnected()
{
    qDebug() << "Connected to server.";
    
    printMessage("<font color=\"Green\">Connected to server.</font>");
    
    socket->write("<br><font color=\"Purple\">" + m_userName.toUtf8() + " has joined the chat room.</font>");
    
    connectedToHost = true;
    
    emit isConnectedChanged();
}

void ClientConnection::socketDisconnected()
{
    qDebug() << "Disconnected from server.";
    
    printMessage("<br><font color=\"Red\">Disconnected from server.</font><br>");
    
    connectedToHost = false;
    
    emit isConnectedChanged();
}

void ClientConnection::socketReadyRead()
{
    printMessage(socket->readAll());
}

void ClientConnection::printMessage(QString message)
{
    m_messages.append(message);
    emit messagesChanged();
}

bool ClientConnection::isConnectedToHost()
{
    return connectedToHost;
}
```
This is the QML file to build the UI of the TCP client immplementation.
```qml
import QtQuick 2.0
import QtQuick.Controls 2.0
import QtQuick.Layouts
import QtQuick.Controls.Universal
import QML_Client 1.0

Window {
    id: window
    width: 640
    height: 480
    visible: true
    title: qsTr("Client Window")
    color: "mediumseagreen"

    ClientConnection {
        id: clientConnection
    }

    ColumnLayout {
        id: infoColumn

        anchors {
            left: parent.left
            right: parent.right
            top: parent.top
            bottom: parent.bottom
            margins: 16
        }

        RowLayout {

            id: userRow
            height: 40
            width: infoColumn.width

            Text {
                id: nameLabel
                text: qsTr("Name:")
                font.pointSize: 15
                color: "white"
                font.weight: Font.Medium
            }

            TextField {
                id: textFieldName
                Layout.fillWidth: true
                onTextChanged: clientConnection.userName = text
                placeholderText: qsTr("Input your name")
                readOnly: clientConnection.isConnected

                background: Rectangle {
                    implicitHeight: 30
                    anchors.fill: parent
                    height: 120
                    color: "white"
                    border.color: textFieldName.activeFocus ? "greenyellow" : "lightslategrey"
                    border.width: 3
                    radius: 5
                }
            }

            Button {
                id: connectButton
                text: clientConnection.isConnected ? "Disconnect" : "Connect"
                Layout.alignment: Qt.AlignRight
                palette.buttonText: "white"
                enabled: {
                    if(textFieldName.text === "") {
                        false
                    } else {
                        true
                    }
                }

                onClicked: {
                    clientConnection.on_connectButton_clicked()
                    forceActiveFocus()
                }

                contentItem: Text {
                    text: connectButton.text
                    font.weight: Font.Medium
                    color: "white"
                    horizontalAlignment: Text.AlignHCenter
                    verticalAlignment: Text.AlignVCenter
                    elide: Text.ElideRight
                }
            }
        }

        ScrollView {
            Layout.fillWidth: true
            Layout.fillHeight: true
            ScrollBar.vertical.policy: ScrollBar.AlwaysOff

            TextArea {
                id: textArea
                wrapMode: TextArea.Wrap
                text: clientConnection.messages
                textFormat: Text.RichText
                readOnly: true
                selectionColor: "green"

                background: Rectangle {
                    color: "white"
                    border.color: "springgreen"
                    border.width: 3
                    radius: 5
                }
            }
        }

        RowLayout {
            TextField {
                id: messageTextField
                Layout.fillWidth: true
                //text: clientConnection.message
                placeholderText: qsTr("Input your message")
                onTextChanged: clientConnection.message = text;

                background: Rectangle {
                    implicitHeight: 30
                    color: "white"
                    border.color: messageTextField.activeFocus ? "greenyellow" : "lightslategrey"
                    border.width: 3
                    radius: 5
                }
            }

            Button {
                id: sendButton
                text: "Send"
                Layout.alignment: Qt.AlignRight

                contentItem: Text {
                    text: sendButton.text
                    font.weight: Font.Medium
                    color: "white"
                    horizontalAlignment: Text.AlignHCenter
                    verticalAlignment: Text.AlignVCenter
                    elide: Text.ElideRight
                }

                onClicked: {
                    clientConnection.on_sendButton_clicked()
                    messageTextField.clear();
                    forceActiveFocus()
                }

                enabled: {
                    if(messageTextField.text === "" || textFieldName.text === "" || !clientConnection.isConnected) {
                        false
                    } else {
                        true
                    }
                }
            }
        }
    }
}
```
