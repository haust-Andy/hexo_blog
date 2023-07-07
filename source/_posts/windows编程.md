---
title: windows知识点
tags: C++
categories: C++
---

# 基础知识
### 程序三种开发方式：
SDK方式开发、MFC方式开发、托管方式开发
三种方式都是基于消息机制开发

### Windows数据类型
DWORD、DWORD32——32字节无符号整型数据<br>
DWORD64——64字节无符号整型数据<br>
INT、INT32、LONG32——32位符号整型数据类型<br>
INT_PTR——指向INT类型数据的指针类型<br>
INT64、LONG64——64位符号整型<br>
#### 参数类
LPARAM——消息的L参数<br>
WPARAM——消息的W参数<br>
LPCSTR——Windows，ANSI，字符串常量<br>
LPCTSTR——根据环境配置，如果定义了UNICODE宏，则是LPCWSTR类型，否则是LPCSTR类型<br>
LPCWSTR——UNICODE字符串常量<br>
LPDWORD——指向DWORD类型数据的指针<br>
LPSTR——Window，ANSI，字符串变量<br>
LPTSTR——根据环境配置，如果定义了UNICODE，则是LPWSTR类型，否则是LPSTR类型<br>
LPWSTR——UNICODE字符串变量<br>
SIZE_T——表示内存大小，以字节为单位，其最大值是CPU最大寻址范围<br>
TCHAR——如果定义了UNICODE，则为WCHAR，否则为CHAR<br>
WCHAR——16位Unicode字符<br>
#### 句柄类
HANDLE——对象的句柄，最基本的句柄类型<br>
HICON——图标的句柄<br>
HINSTANCE——程序实例的句柄<br>
HKEY——注册表键的句柄<br>
HMODULE——模块的句柄<br>
HWND——窗口的句柄<br>
<table><tr><td> 前缀 </td><td>含义</td><td>前缀</td><td>含义</td></tr>
    <tr><td> a </td><td>数组 array</td><td> b </td><td>布尔值 bool</td></tr> 
    <tr><td> by </td><td>无符号字符(字节)</td><td> c </td><td>字符(字节)</td></tr>
    <tr><td> cb </td><td>字节计数</td><td> rgb </td><td>保存颜色值的长整型</td></tr>
    <tr><td> cx,cy </td><td>短整型(计算x,y的长度)</td><td> dw </td><td>无符号长整型</td></tr>
    <tr><td> fn </td><td>函数</td><td> h </td><td>句柄</td></tr>
    <tr><td> i </td><td>整形(integer)</td><td> m_ </td><td>类的数据成员member</td></tr>
    <tr><td> n </td><td>短整型或整型</td><td> np </td><td>近指针</td></tr>
    <tr><td> p </td><td>指针(pointer)</td><td> l </td><td>长整型(long)</td></tr>
    <tr><td> lp </td><td>长指针</td><td> s </td><td>字符串string</td></tr>
    <tr><td> sz </td><td>以零结尾的字符串</td><td> tm </td><td>正文大小</td></tr>
    <tr><td> w </td><td>无符号整型</td><td> x,y </td><td>无符号整型(表示x,y的坐标)</td></tr>
</table>



## ***Windows消息循环***
### 消息来源：
操作系统产生的消息<br>
用户触发事件消息<br>
由消息产生消息<br>

### 消息常见定义
#### 预定义消息
普通窗口消息WM<br>
设备消息DBT<br>
按钮消息BM<br>
#### 自定义消息 WM_USER + 1

### 消息结构
#### MSG
窗口句柄、消息类型、高位参数、地位参数、时间、xy轴


### 程序骨架
设计窗口类=>注册窗口类=>创建窗口=>显示以及更新窗口=>消息循环
```
#include<windows.h>
#include<stdio.h>
LPCTSTR clsName = (LPCTSTR)"My";
LPCTSTR msgName = (LPCTSTR)"hhhhhh";

int WINAPI WinMain(
    _In_ HINSTANCE hInstance,
    _In_opt_ HINSTANCE hPrevInstance,
    _In_ LPSTR lpCmdLine,
    _In_ int nShowCmd
) {
    //定义和配置窗口
    WNDCLASS wndcls;
    wndcls.cbClsExtra = NULL;
    wndcls.cbWndExtra = NULL;
    wndcls.hbrBackground = (HBRUSH)GetStockObject(WHITE_BRUSH);
    wndcls.hCursor = LoadCursor(NULL, IDC_ARROW);
    wndcls.hIcon = LoadIcon(NULL, IDI_APPLICATION);
    wndcls.hInstance = hInstance;
    wndcls.lpfnWndProc = MyMinProc;
    //定义窗口代号
    wndcls.lpszClassName = clsName;
    wndcls.lpszMenuName = NULL;
    wndcls.style = CS_HREDRAW | CS_VREDRAW;

    //注册窗口类
    RegisterClass(&wndcls);

    //创建窗口
    HWND hwnd;
    hwnd = CreateWindow(clsName, msgName, WS_OVERLAPPEDWINDOW, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, NULL, NULL, hInstance, NULL);

    //显示和刷新窗口
    ShowWindow(hwnd, SW_SHOWNA);
    UpdateWindow(hwnd);

    //消息循环 GetMessage只有在接收到WM_QUIT才会返回0
    //TranslateMessage 翻译消息WM_KEYDOWN W
    MSG msg;
    while (GetMessage(&msg,NULL,NULL,NULL))
    {
        TranslateMessage(&msg);//消息转换
        DispatchMessage(&msg);//消息分发
    }
    return msg.wParam;
}
LRESULT CALLBACK MyMinProc(
    HWND hwnd,
    UINT uMsg, //消息类型
    WPARAM wParam,
    LPARAM lParam
) {
    //uMsg消息类型
    int ret;
    HDC hdc;
    switch (uMsg) {
    case WM_CHAR:
        char szChar[20];
        sprintf_s(szChar, "您刚才按下了：%c", wParam);
        MessageBox(hwnd, szChar,"char",NULL);
        break;
    case WM_LBUTTONDOWN:
        MessageBox(hwnd, "检测鼠标左键","msg",NULL);
    case WM_PAINT:
        PAINTSTRUCT ps;
        hdc = BeginPaint(hwnd, &ps);
        TextOut(hdc, 0, 0, "www.baidu.com", strlen("www.baidu.com"));
        EndPaint(hwnd, &ps);
        MessageBox(hwnd, "重绘","msg", NULL);
        break;
    case WM_CLOSE:
        ret = MessageBox(hwnd, "是否真的关闭", "msg", MB_YESNO);
        if (ret = IDYES) {
            DestroyWindow(hwnd);
        }
        break;
    case WM_DESTROY:
        PostQuitMessage(0);
        break;
    default:
        return DefWindowProc(hwnd,  uMsg, wParam, lParam);
    }
    return 0;
}
```
主要函数：
```
 int WinMain(){
    // 设计窗口外观及交互响应，注册，申请专利
    RegisterClass(...)注册窗口类
    CreateWindow(...)// 创建窗口
    ShowWindow(...)// 展示窗口
    UpdateWindow(...)// 刷新窗口
    // 进入消息循环
    while (GetMessage(...)) {
        TranslateMessage(...);// 消息转换
        DispatchMessage(...);// 消息分发
    }
}
```
## ***网络编程***




