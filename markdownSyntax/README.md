# 마크다운 문법(Markdown Syntax)

## 마크다운(Markdown)

- 마크다운(Markdown)은 일반 텍스트 기반의 마크업 언어다.

- 깃(Git)의 README.md 파일이나 온라인 문서, 혹은 일반 텍스트 편집기로 문서 양식을 편집할 때,
  쉽게 쓰고 읽을 수 있으며 HTML로 변환이 가능하다.

## 1. 줄 바꾸기

- 줄 바꿈 1번 : `Enter` 키를 2번 누르기

- 줄 바꿈 여러 번 : `<br/>` 를 사용하자 

## 2. 제목(Header) 

HTML의 `<h1> ~ <h6>`를 `#`의 개수로 표현한다.

```
# h1 
## h2 
### h3 
#### h4 
##### h5 
###### h6
```

## 3. 목록 (List)

```
순서가 있는 목록
1. Item 1
2. Item 2
    1. SubList Item 1
    2. SubList Item 2
    3. SubList Item 3

순서가 없는 목록
* Item 1
* Item 2
    * SubList Item 1
    * SubList Item 2
    * SubList Item 3
```

순서가 없는 목록에서 * 대신에 - 또는 +를 사용 해도 된다.

## 4. 하이퍼링크

```
[깃허브](http://github.com "링크에 대한 설명")

구글 : https://www.google.com/
```

## 5. 이미지

* 먼저, 프로젝트 폴더에 images 폴더를 만들고 이미지를 저장한다.

* 아래 명령어로 이미지를 표시한다.

    * `![이미지 alt명](이미지 url)`

<br/>

[코드]

<br/>

```
![Markdown logo](images/Markdown_logo.png) 
```

<br/>

[실행결과]

<br/>

![Markdown logo](images/Markdown_logo.png) 

## 6. 인라인 코드 강조

인라인 코드 강는 백틱(｀)을 이용하여 표현한다.

```
문단 중간에 `Code`를 넣을 수 있습니다. 
예를 들어 `printf("hello world!");` 이런 식으로 들어갑니다.
```

## 7. 블록(Block) 코드 강조

블록 코드 강조는 백틱 3개(```) 이용하여 표현한다.

## 8. 인용구

`>` 를 이용하여 인용구를 표시한다.

```
> 인용구 1
>> 인용구 2
>>> 인용구 3
```

## 9. 강조(Emphasis)

이탤릭체는 `*텍스트*` 또는 `_텍스트_` 으로 표현한다.

볼드체는 `**텍스트**` 또는 `__텍스트__` 으로 표현한다.

취소는 `~~텍스트~~` 으로 표현한다.

볼드체는 `~~텍스트~~` 으로 표현한다.

밑줄은 `<u>텍스트</u>` 으로 표현한다.


```
*이탤릭체* 

_이탤릭체_ 

**볼드체**

__볼드체__ 

~~취소선~~

<u>밑줄</u>
```

## 10. 테이블

파이프(`|`) 기호를 이용하여 테이블을 작성한다.

중앙 정렬은 `:---:` 으로 하며 오른쪽 정렬은 `---:` 으로 한다. 
      
```
| 제품 | 개수 | 가격 |
| --- | :---: | ---:|
| `시계` | `1`개 | `200`|
| `컴퓨터` | `1`개 | `150` |
| `라디오` | `가`개 | `10` |
```

## 11. 체크박스

빈 체크 박스는 `- [ ]` 로 만든다.
완료된 체크 박스는 `- [x]` 로 만든다.

```
- [X] Java
- [X] Spring
- [ ] JavaScript
```

## 12. 수평선

```
--- 
___

*** 
```

## 13. 이스케이프 문자

* \를 사용하여 마크다운 문법을 적용하지 않는 것이 가능하다.

```
\*
\\
\`
\#
```

## 14. 배지(Badge)

* [배지(Badge)](https://shields.io/ "이모지")에 대한 더 자세한 내용은 링크를 참조하자.

```
<img src="https://img.shields.io/github/commit-activity/m/kevinntech/TIL> 
```

<img src="https://img.shields.io/github/commit-activity/m/kevinntech/TIL"> 

[배지(Badge)](https://img.shields.io/github/commit-activity/m/kevinntech/TIL")

## 15. 이모지(Emoji)

* [이모지(Emoji)](https://www.webfx.com/tools/emoji-cheat-sheet/ "이모지")에 대한 더 자세한 내용은 링크를 참조하자.

```
:octocat:
:rocket:
:blue_car:
```

:octocat:
:rocket:
:blue_car:

