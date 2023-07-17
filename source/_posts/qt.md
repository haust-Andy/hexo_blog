---
title: qt知识点
tags: qt
categories: C++
---

## 信号与槽
### 信号(Signal)
信号（Signal）就是在特定情况下被发射的事件，例如PushButton 最常见的信号就是鼠标单击时发射的 clicked() 信号，一个 ComboBox 最常见的信号是选择的列表项变化时发射的 CurrentIndexChanged() 信号。
### 槽(slot)
就是对信号响应的函数。槽就是一个函数，与一般的C++函数是一样的，可以定义在类的任何部分（public、private 或 protected），可以具有任何参数，也可以被直接调用。槽函数与一般的函数不同的是：槽函数可以与一个信号关联，当信号被发射时，关联的槽函数被自动执行。
### 信号与槽关联是用 QObject::connect() 函数实现的，其基本格式是：
QObject::connect(sender, SIGNAL(signal()), receiver, SLOT(slot()));
参数1、信号的发送者
参数2、发送的信号
参数3、信号的接收者
参数4、处理函数（槽函数）

## 对话框-模态和非模态
模态对话框：不能对其他窗口进行操作
非模态对话框：可以对其他窗口进行操作。