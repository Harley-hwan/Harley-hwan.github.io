---
layout: post
title: Liquid Syntax Error Solution
subtitle: Handling Liquid parsing errors in Jekyll & GitHub Pages
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [liquid, jekyll, githubpages, syntaxerror, webdevelopment]
comments: true
---

# Liquid Syntax Error 해결하기

- 최초 작성일: 2023년 9월 7일(목)

## 목차

[TOC]

## 내용

여느 때와 같이 Git Blog를 업데이트하던 중에, 아래의 이미지처럼 에러가 발생하여 build가 되지 않아 Git Blog가 업데이트되지 않는 현상을 겪게 되었다.

<br/>

![image](https://github.com/harley-hwan/harley-hwan.github.io/assets/68185569/e2c4f2cf-302f-4f06-82a9-6fcc5293d34f)

```
Liquid Exception: Liquid syntax error (line 105): Tag '{%' was not properly terminated with regexp: /\%\}/ in /github/workspace/cuda/_posts/2023-08-22-StartCuda.md
/usr/local/bundle/gems/liquid-4.0.4/lib/liquid/block_body.rb:132:in `raise_missing_tag_terminator': Liquid syntax error (line 105): Tag '{%' was not properly terminated with regexp: /\\%\\}/ (Liquid::SyntaxError)
```

<br/>

에러 메시지에 따르면 "Liquid syntax error (line 105): Tag '{%' was not properly terminated"로 나타나므로 {% 또는 %}와 같은 Liquid 태그가 제대로 닫히지 않았을 가능성이 있다는 것이다.

이 문제를 해결하려면 

Liquid 태그의 문제점 파악: 2023-08-22-(1)StartCuda.md 파일에서 {% 또는 %} 태그를 검색하여 해당 태그가 제대로 열렸는지, 닫혔는지 확인한다. 

이 태그는 주로 Jekyll plugins 또는 Liquid 로직에서 사용된다. 

제시된 내용에는 이러한 태그가 없으므로, 이 포스트 전후로 태그를 열고 닫아주면 된다.

<br/>

```c++
int main()
{
    const int arraySize = 5;
    const int a[arraySize] = { 1, 2, 3, 4, 5 };
    const int b[arraySize] = { 10, 20, 30, 40, 50 };
```

위의 코드는 실제로 105번째 line인 __const int b[arraySize] = { 10, 20, 30, 40, 50 };__ 부분인데,

'{'과 '}'를 잘못 해석하고 파싱해서 그런 것 같았다. 

<br/>

그래서 `{% raw %}` 와 `{% endraw %}` 태그로 C++ 코드 블록을 감싸주면, 

Liquid template engine이 `{}` 내부의 내용을 파싱하려고 시도하지 않을 것이므로 GitHub Pages에서 문제 없이 빌드가 된다.

