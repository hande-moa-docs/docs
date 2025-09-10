
# 반응형 레이아웃 테스트 가이드

-----

## 문서 정보

- **문서명**: 반응형 레이아웃 가이드
- **버전**: v1.0.0
- **작성일**: 2025.09.10
- **작성자**: [고동현](https://github.com/rhehdgus8831)
- **최종 수정일**: 2025.09.10

-----

## 1\. 개요 (Overview)

본 문서는 '반띵' 프로젝트의 **반응형 웹 레이아웃**을 시각적으로 확인하기 위한 HTML 코드 예제입니다. 이 코드는 팀의 '모바일 우선' 정책에 따라, 다양한 화면 크기(데스크톱, 태블릿, 모바일)에서 콘텐츠가 어떻게 중앙 정렬되고 양옆 여백이 자동으로 조절되는지 보여주는 것을 목적으로 합니다.

## 2\. 소스 코드

아래 코드를 `layout-test.html`과 같은 파일로 저장한 뒤 웹 브라우저에서 실행하면, 실제 레이아웃 동작을 확인할 수 있습니다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>반응형 레이아웃 테스트 (여백 증가)</title>
    <style>
        /* 기본 스타일 초기화 */
        body, html {
            margin: 0;
            padding: 0;
            font-family: sans-serif;
            background-color: #FAFAFA; /* 양옆 빈 공간을 채울 프로젝트 배경색 */
            color: #2D3436;
        }

        /* ★★★★★ 이 부분의 max-width 값을 조절해서 여백을 바꿀 수 있습니다 ★★★★★ */
        .container {
            width: 100%;
            max-width: 900px; /* PC 같은 넓은 화면에서는 최대 900px까지만 커짐 */
            margin: 0 auto; /* 좌우 여백을 'auto'로 설정해 자동으로 중앙 정렬 */

            padding-left: 20px;
            padding-right: 20px;
            box-sizing: border-box; /* 패딩이 너비에 영향을 주지 않도록 설정 */
        }

        /* 각 영역의 배경색과 추가 스타일 */
        .header {
            background-color: #A29BFE; /* 프로젝트 Primary 색 (라벤더) */
            color: white;
            padding-top: 20px;
            padding-bottom: 20px;
            text-align: center;
            font-size: 24px;
            font-weight: bold;
        }

        .main-content {
            background-color: white;
            min-height: 60vh;
            padding-top: 20px;
            padding-bottom: 20px;
        }

        .footer {
            background-color: #f2f2f2;
            color: #555;
            padding-top: 20px;
            padding-bottom: 20px;
            margin-top: 40px;
            text-align: center;
        }

        /* 콘텐츠 내용 예시 스타일 */
        h1 {
            color: #A29BFE;
        }
        p {
            line-height: 1.6;
        }
    </style>
</head>
<body>

<header class="container header">
    반띵 (BanThing)
</header>

<main class="container main-content">
    <h1>메인 콘텐츠 영역</h1>
    <p><strong>max-width</strong> 값을 더 작게 조절하면 여백은 더 넓어지고, 더 크게 조절하면 여백은 더 좁아집니다. 원하시는 느낌에 맞춰 이 값을 조절하시면 됩니다.</p>
</main>

<footer class="container footer">
    © 2025 Team Nonagajyeo. All Rights Reserved.
</footer>

</body>
</html>
```

-----

## 변경 이력

| 버전   | 날짜        | 변경 내용      | 작성자 |
| ------ | ----------- | -------------- |-----|
| v1.0.0 | 2025.09.10 | 초기 문서 작성 | 고동현 |