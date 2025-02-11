---
layout: post
title: "(WPF) 3. WPF에서 컨트롤 사용하기"
subtitle: "라벨(Label), 체크 박스(CheckBox), 라디오 버튼(RadioButton), 텍스트 박스(TextBox), 버튼(Button), 패스워드 박스(PasswordBox), 리스트 뷰(ListView) 활용"
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c#, wpf, controls, ui, xaml]
comments: true
filename: "2023-05-20-wpf-project-03-controls.md"
---

# (WPF) 3. WPF에서 컨트롤 사용하기
- 최초 작성일: 2025년 2월 11일 (화)

## 목차
1. [WPF 컨트롤 개요](#wpf-컨트롤-개요)
2. [라벨(Label) 사용하기](#라벨label-사용하기)
3. [체크 박스(CheckBox)와 라디오 버튼(RadioButton)](#체크-박스checkbox와-라디오-버튼radiobutton)
4. [텍스트 박스(TextBox)와 패스워드 박스(PasswordBox)](#텍스트-박스textbox와-패스워드-박스passwordbox)
5. [버튼(Button)과 이벤트 핸들러](#버튼button과-이벤트-핸들러)
6. [리스트 뷰(ListView)와 데이터 바인딩](#리스트-뷰listview와-데이터-바인딩)
7. [스택 패널(StackPanel) 활용](#스택-패널stackpanel-활용)

---

## WPF 컨트롤 개요

WPF(Windows Presentation Foundation)에서는 다양한 UI 컨트롤을 제공하며, 이를 활용하여 사용자 인터페이스를 직관적으로 구성할 수 있다. 이번 글에서는 **라벨(Label), 체크 박스(CheckBox), 라디오 버튼(RadioButton), 텍스트 박스(TextBox), 버튼(Button), 패스워드 박스(PasswordBox), 리스트 뷰(ListView)** 등을 사용하는 방법을 설명한다.

각 컨트롤의 동작 방식과 기본적인 사용법을 예제 코드와 함께 학습해보자.

---

## 라벨(Label) 사용하기

라벨(Label)은 화면에 텍스트를 표시하는 컨트롤이다.

### 라벨 추가하기

1. `MainWindow.xaml`에서 `도구 상자`를 열고 `Label`을 드래그하여 추가한다.
2. XAML 코드에서 다음과 같이 정의할 수 있다.

```xml
<Label Name="LabelTest1" Content="기본 텍스트" Width="200" Height="30"/>
```

3. C# 코드에서 Label의 텍스트를 변경할 수 있다.

```csharp
LabelTest1.Content = "변경된 텍스트";
```

![Label 컨트롤 예제](https://github.com/user-attachments/sample-label.png)

---

## 체크 박스(CheckBox)와 라디오 버튼(RadioButton)

### 체크 박스 추가하기

```xml
<CheckBox Name="CheckBox1" Content="동의합니다" IsChecked="False"/>
```

C#에서 체크 상태를 확인하려면 다음과 같이 작성한다.

```csharp
if (CheckBox1.IsChecked == true)
{
    MessageBox.Show("체크되었습니다.");
}
```

### 라디오 버튼 사용하기

```xml
<StackPanel>
    <RadioButton Name="Radio1" Content="옵션 1" GroupName="Options"/>
    <RadioButton Name="Radio2" Content="옵션 2" GroupName="Options"/>
</StackPanel>
```

라디오 버튼은 **GroupName**을 지정하면 그룹 내에서 하나의 옵션만 선택할 수 있다.

![CheckBox 및 RadioButton 예제](https://github.com/user-attachments/sample-checkbox-radio.png)

---

## 텍스트 박스(TextBox)와 패스워드 박스(PasswordBox)

### 텍스트 박스 추가하기

```xml
<TextBox Name="TextBox1" Width="200" Height="30"/>
```

사용자가 입력한 텍스트를 가져오는 코드:

```csharp
string userInput = TextBox1.Text;
```

### 패스워드 박스 사용하기

```xml
<PasswordBox Name="PasswordBox1" Width="200" Height="30"/>
```

비밀번호 입력 값 가져오기:

```csharp
string password = PasswordBox1.Password;
```

---

## 버튼(Button)과 이벤트 핸들러

버튼 클릭 이벤트를 추가하는 방법:

```xml
<Button Name="BtnClick" Content="클릭" Click="BtnClick_Click"/>
```

C# 코드에서 클릭 이벤트 처리:

```csharp
private void BtnClick_Click(object sender, RoutedEventArgs e)
{
    LabelTest1.Content = "버튼이 클릭되었습니다!";
}
```

---

## 리스트 뷰(ListView)와 데이터 바인딩

리스트 뷰를 사용하여 데이터를 표 형태로 표시할 수 있다.

### 리스트 뷰 추가하기

```xml
<ListView Name="ListView1">
    <ListView.View>
        <GridView>
            <GridViewColumn Header="이름" DisplayMemberBinding="{Binding Name}"/>
            <GridViewColumn Header="나이" DisplayMemberBinding="{Binding Age}"/>
        </GridView>
    </ListView.View>
</ListView>
```

### 데이터 모델 생성 및 바인딩

```csharp
public class User
{
    public string Name { get; set; }
    public int Age { get; set; }
}

List<User> users = new List<User>
{
    new User { Name = "홍길동", Age = 25 },
    new User { Name = "이순신", Age = 40 }
};

ListView1.ItemsSource = users;
```

![ListView 예제](https://github.com/user-attachments/sample-listview.png)

---

## 스택 패널(StackPanel) 활용

스택 패널(StackPanel)은 컨트롤을 세로 또는 가로로 배치할 때 사용한다.

```xml
<StackPanel Orientation="Vertical">
    <Button Content="버튼 1" Width="100" Height="30"/>
    <Button Content="버튼 2" Width="100" Height="30"/>
</StackPanel>
```

위 예제는 버튼을 세로로 정렬한다.

![StackPanel 예제](https://github.com/user-attachments/sample-stackpanel.png)

---

이 글에서는 WPF에서 다양한 컨트롤을 사용하는 방법을 살펴보았다. 이를 활용하여 보다 직관적인 UI를 개발할 수 있다.

