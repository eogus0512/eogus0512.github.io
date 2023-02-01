---
layout: post
title: (Spring Boot) 웹 개발 프로젝트 - 1
subtitle: 프로젝트 생성 및 구축
categories: Project
tags: [Spring, Java, Web]
---

먼저 개발을 하기 위해서 프로젝트를 생성해야한다. 

Spring은 **spring initializr** 이라는 프로젝트 생성 사이트를 제공한다. 
[spring initializr](https://start.spring.io/)

이 프로그램을 이용하면 Spring 프로젝트 파일을 매우 간편하게 생성할 수 있다. 

![springInitialzr](https://user-images.githubusercontent.com/71585151/215978714-f72d0022-b5d3-43d6-8ddc-63d0a2b52e48.png)

필자는 위와 같이 프로젝트를 설정하였다.
언어는 Java를 사용하고 Spring Boot는 제일 최신버전을 선택하였다. (SNAPSHOT 버전 같은 경우 베타버전이라고 생각하면 된다.)
Packaging을 Jar를 선택하고 자바버전은 11을 사용할 것이다.

그리고 spring initializr은 Dependencies를 추가할 수 있도록 해주는데 필자는 웹 개발에 필요한 'Spring Web', 검증에 필요한 'Validation', 코드를 간결하게 만들어주는 'Lombok', 서버사이트 자바템플릿인 'Thymeleaf'를 추가하였다.

나중에 build.gradle을 통하여 Dependency를 더 추가할 수 있다.

설정이 완료가 되었으면 왼쪽 하단 'GENERATE' 버튼을 클릭하면 프로젝트의 압축파일 다운로드가 시작된다.

다운로드 후에 압축을 풀어 

An h2 header
------------

Here's a numbered list:

 1. first item
 2. second item
 3. third item

Note again how the actual text starts at 4 columns in (4 characters
from the left side). Here's a code sample:

    # Let me re-iterate ...
    for i in 1 .. 10 { do-something(i) }

As you probably guessed, indented 4 spaces. By the way, instead of
indenting the block, you can use delimited blocks, if you like:

~~~
define foobar() {
    print "Welcome to flavor country!";
}
~~~

(which makes copying & pasting easier). You can optionally mark the
delimited block for Pandoc to syntax highlight it:

~~~python
import time
# Quick, count to ten!
for i in range(10):
    # (but not *too* quick)
    time.sleep(0.5)
    print(i)
~~~



### An h3 header ###

Now a nested list:

 1. First, get these ingredients:

      * carrots
      * celery
      * lentils

 2. Boil some water.

 3. Dump everything in the pot and follow
    this algorithm:

        find wooden spoon
        uncover pot
        stir
        cover pot
        balance wooden spoon precariously on pot handle
        wait 10 minutes
        goto first step (or shut off burner when done)

    Do not bump wooden spoon or it will fall.

Notice again how text always lines up on 4-space indents (including
that last line which continues item 3 above).

Here's a link to [a website](http://foo.bar), to a [local
doc](local-doc.html), and to a [section heading in the current
doc](#an-h2-header). Here's a footnote [^1].

[^1]: Some footnote text.

Tables can look like this:

Name           Size  Material      Color
------------- -----  ------------  ------------
All Business      9  leather       brown
Roundabout       10  hemp canvas   natural
Cinderella       11  glass         transparent

Table: Shoes sizes, materials, and colors.

(The above is the caption for the table.) Pandoc also supports
multi-line tables:

--------  -----------------------
Keyword   Text
--------  -----------------------
red       Sunsets, apples, and
          other red or reddish
          things.

green     Leaves, grass, frogs
          and other things it's
          not easy being.
--------  -----------------------

A horizontal rule follows.

***

Here's a definition list:

apples
  : Good for making applesauce.

oranges
  : Citrus!

tomatoes
  : There's no "e" in tomatoe.

Again, text is indented 4 spaces. (Put a blank line between each
term and  its definition to spread things out more.)

Here's a "line block" (note how whitespace is honored):

| Line one
|   Line too
| Line tree

and images can be specified like so:

![example image](https://user-images.githubusercontent.com/9413601/123900693-1d9ebd00-d99c-11eb-8e9e-cf7879187606.png "An exemplary image")

Inline math equation: $\omega = d\phi / dt$. Display
math should get its own line like so:

$$I = \int \rho R^{2} dV$$

And note that you can backslash-escape any punctuation characters
which you wish to be displayed literally, ex.: \`foo\`, \*bar\*, etc.
