---
tag: Hacking
title: Sublime Text
---

##Package Control 설치하기
Package Control은 플러그인을 간편하게 설치할 수 있도록 해주는 플러그인이다. 플러그인들의 설치, 관리, 삭제, 업데이트를 편하게 해준다.

``Ctrl + ` `` 혹은 View - Show console을 통해 하단에 콘솔 창을 불러주자. 그 다음 [https://packagecontrol.io/installation](https://packagecontrol.io/installation)의 코드를 붙혀 넣는다. 설치가 완료되면 `Ctrl + Shift + P`를 누르고 텍스트 박스에 'Package Control'를 입력해보자. 접두사로 'Package Control: '가 붙은 명령들이 새로 추가된 것들이다. 전부 입력할 필요 없이 간단히 'pc'라고만 입력해도 Package Control 플러그인 위주로 목록이 적당히 축약된다.

'Package Control: Install Package'로 새로운 플러그인을 찾아서 설치할 수 있으며, 기존 목록에 존재하지 않는 플러그인을 설치하려면 'Package Control: Add Repository'를 통해 플러그인의 Reposistory 주소를 추가할 수 있다. 주소를 추가한 후에는 Sublime Text를 재실행해주자.


##플러그인

###SublimeREPL
Python을 공부하는 중 input()가 있는 코드를 빌드하면 EOFError가 발생한다. SublimeREPL은 이런 코드도 Sublime Text 내에서 실행할 수 있도록 해준다. [https://github.com/wuub/SublimeREPL](https://github.com/wuub/SublimeREPL)를 Repository를 추가해주자. 그럼 설치할 수 있는 패키지 목록에서 SublimeREPL을 찾을 수 있다. 

사용법은 `Ctrl + Shift + P`로 'SublimeREPL: Python - RUN current file'을 실행하면 된다.

###IMESupport
Windows버전 Sublime Text에서는 글자를 조합하는 중에 한글이 표시되지 않는다. 대신 작은 IME창이 뜨는 걸 볼 수 있는데, IMESupport는 IME창 대신에 코드 안에서 한글이 조합되는 모습을 보여준다. 완전히 조합이 되어야 글자가 입력된다는 게 단점이다.

Repository를 추가할 필요없이 설치 가능한 목록에서 IMESupport를 찾을 수 있다.

###Anaconda
[Anaconda](http://damnwidget.github.io/anaconda)는 Sublime Text를 파이썬의 IDE로 만들어준다. 

###SideBarEnhancements
사이드바의 기능을 확장시켜준다. 파일에 우클릭으로 메뉴를 열여보자.


##기타

###맥에서 Python3로 빌드하기
맥에 Python3가 설치되어 있는걸 전제로 한다. Tool - Build System - New Build System으로 새로운 창을 띄우고, 터미널을 열어 `which python3`를 치고 나오는 경로를 다음 코드에서 치환해 넣어준 후 저장한다.

    {
        "cmd": ["/Library/Frameworks/Python.framework/Versions/3.4/bin/python3", "$file"],
        "selector": "source.python",
        "file_regex": "file \"(...*?)\", line ([0-9]+)"
    }

###파일 저장 시 access denied 뜰 때
위의 글꼴 바꾸기와 마찬가지로 Setting - User에 `"atomic_save": false,`를 추가한다.

### SyntaxError: Non-ASCII character 뜰 때
소스 상단에 다음 코드를 넣어준다.
    #-*- coding:utf-8 -*-