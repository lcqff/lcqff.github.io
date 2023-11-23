---
layout: post
title: "안드로이드 스튜디오 오류: Tag mismatch!"
author: Jeffrey
categories: flutter
banner:
  image: https://bit.ly/3xTmdUP
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold;"
  subheading_style: "color: gold"
tags: flutter android_studio 오류
sidebar: []
---

## An error occurred while preparing SDK package Android SDK Command-line Tools (latest): <mark>Tag mismatch!.</mark>

<br/>
안드로이드 스튜디오에서 sdk파일을 다운로드 하는 중 위와 같은 오류가 발생했다.
<br/><br/>

![스크린샷 2023-03-06 172034](https://user-images.githubusercontent.com/71930280/223070694-9006b123-ec8e-44f0-bf5d-c476a34e05fb.png)
**C:\Users\user\AppData\Local\Android\Sdk** 폴더의 **.downloadIntermediates**를 삭제 후 안드로이드를 재실행한 뒤 다시 시도하니 문제없이 작동했다.

[https://www.youtube.com/watch?v=TiqOCg_10n0](https://www.youtube.com/watch?v=TiqOCg_10n0) <br/> 해결방법은 해당 영상을 참고했다.
만약 위 방식으로 해결되지 않는다면 방화벽을 해제해보길 권한다. (해당 영상 1분 50초부터.)
