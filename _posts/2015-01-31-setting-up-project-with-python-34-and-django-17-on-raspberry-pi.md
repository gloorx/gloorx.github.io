---
tag: Hacking
title: Raspberry Pi에서 Python 3.4와 Django 1.7로 프로젝트 셋업하기
---

Raspberry Pi 전용 OS인 Raspbian에서 작업했다. Debian에서도 똑같이 진행 할 수 있을 듯 하다. 처음엔 AWS을 이용하려고 했지만, django도 python도 익숙치 않아서 삽질 많이 할 텐데 AWS에까지 에너지를 소비하고 싶지 않았다. 사실 몇 달 전에 사진 관련 SNS 만들 때 AWS를 사용했는데, 무료 사용 1년이 지나면 가입할 때 입력한 카드로 과금이 된다고 한다. 애플 연회비도 안내서 앱도 내려간 마당에 AWS도 그냥 닫아버렸다. 서버 닫는거 잊어버리고 과금될까봐 그런 것도 있고, 라즈베리파이는 리눅스 장난감 같은 느낌이라 재미있다.

##Raspberry Pi 설정

    $ sudo raspi-config

    $ sudo apt-get install rpi-update
    $ sudo rpi-update
    $ sudo reboot
    $ sudo apt-get update
    $ sudo apt-get dist-upgrade

##vim 설치

콘솔창에서 파일 편집은 항상 nano를 쓰곤 했는데 이번 기회에 vi(vim)는 어떤지 궁금해서 설치했다. 

    $ sudo apt-get install vim

##mySQL 설치, 실행

파이썬에 내장된 sqlite를 사용하려면 mySQL은 설치하지 않아도 된다. 이 글에서는 sqlite 대신 mySQL을 사용하기 위해 설치한다.

    $ sudo apt-get install mysql-server mysql-client

설치가 끝나면 ID와 PW를 설정하는 화면이 나타난다. 나중에 프로젝트 DB를 설정할 때 쓰이기 때문에 기억해 두어야 한다.

##데이터베이스 만들기

mySQL을 쓸 것이기 때문에 데이터베이스를 미리 만들어 주어야 한다. sqlite에서는 이 과정이 필요없는 듯 보인다.

    $ sudo service mysql start
    $ mysql -u 사용자명 -p
    > CREATE DATABASE 데이터베이스명;
    > quit;

##pip 설치

파이썬의 라이브러리를 쉽게 관리하기 위해 pip를 설치한다. 근데 설치하고보니 3.4에서는 빌트인 되어 있는 것 같다.

    $ wget https://bootstrap.pypa.io/get-pip.py
    $ sudo python get-pip.py

##Python 다른버전 설치

설치되어 있는 파이썬 외의 다른 버전의 파이썬이 필요하면 아래 명령어를 통해 설치할 수 있다. `make`는 약 한 시간정도 시간을 잡아먹으니 타이밍을 잘 맞춰서 밥 먹을 때 즈음 실행해놓자.

    $ wget https://www.python.org/ftp/python/3.4.2/Python-3.4.2.tgz
    $ tar xvzf Python-3.4.2.tgz
    $ cd Python-3.4.2/
    $ ./configure
    $ make                  # 한 시간 소요
    $ sudo make install 

`sudo make install` 도중 Ignoring ensurepip failure: pip 1.5.6 requires SSL/TLS 오류가 뜨면 아래 명령어로 해결 할 수 있다.

    $ sudo apt-get install libssl-dev openssl 
    $ sudo make install

INFO: Can't locate Tcl/Tk libs and/or headers라고도 떴지만 일단은 무시했다.

##virtualenv

*3.x의 파이썬을 쓴다면 virtualenv 대신 아래에 pyvenv를 설치하자.*

>virtualenv는 가상의 파이썬 작업환경을 만들어 준다. 작업환경을 따로따로 만들어주면 해당 환경 내의 파이썬으로 무슨 짓을 해도 시스템 파이썬이나 다른 가상의 작업환경에게 영향을 주지 않는다. 

라고 대부분의 튜토리얼에서 설명되어 있는데, 사실은 시스템에 설치된 파이썬을 다른 곳에 복사하고, 파이썬 코드를 실행 할 때 어느 파이썬을 쓸 것인지 선택하는 기능 밖에 없는 것 같다. 위의 설명과 같이 고립된 작업환경을 만들려면 디렉토리 구조를 잘 짜야한다.

    $ sudo pip install virtualenv

    $ mkdir django_tests
    $ cd django_tests
    $ virtualenv --no-site-packages env
    $ source env/bin/activate   # env의 파이썬 활성화

새로운 디렉토리 django_tests를 만들고 그 안에서 가상환경을 만들어 준다. 그러면 django_tests 안에 env가 생기는데 이게 가상환경의 파이썬이 들어있는 공간이다. 가상환경을 활성화시키고 django_tests 내부에서만 작업하여 이 디렉토리 안에 있는 파이썬 코드들은 env의 파이썬으로 실행해야한다는 것을 명확하게 알 수 있다. 가상환경을 활성화시키면 명령어 입력줄 앞에 `(env)`가 표시된다.

다시 시스템 파이썬으로 복귀하고 싶으면 `deactivate`를 입력하면 된다.

    (env)$ deactivate           # 시스템 파이썬으로 복귀

##pyvenv
3.3에서부터 pyvenv에 기본으로 설치되어 있다. 다만 3.3에서는 pip를 가상 환경을 만들 때마다 설치해주어야 한다. 3.4에서는 pip까지 기본으로 설치되어 있다.

    $ mkdir django_tests
    $ cd django_tests
    $ pyvenv-3.4 env
    $ source env/bin/activate   # env의 파이썬 활성화
    (env)$ deactivate           # 시스템 파이썬으로 복귀

##django 설치

가상환경을 다시 활성화 시키고 장고를 설치한다.

    (env)$ pip install django

##django 프로젝트 만들기

    (env)$ django-admin.py startproject django_test01

##github 연동

    (env)$ cd django_test01
    (env)$ git init
    (env)$ git remote add origin https://github.com/아이디/프로젝트명.git
    (env)$ git pull origin master

이미 있던 프로젝트를 깃허브에 올리려면 remote하고 pull을 먼저해야 push할 수 있다. github연동부터 하고 `django-admin.py startproject`하면 빈 디렉토리가 아니라면서 프로젝트가 만들어지지 않는다.

##python 3.x django에 mySQL 설정하기

    (env)$ pip install https://dev.mysql.com/get/Downloads/Connector-Python/mysql-connector-python-2.0.1.tar.gz

최신 버전 주소는 http://dev.mysql.com/downloads/connector/python/의 Platform Independent에서 확인할 수 있다.

    (env)$ nano django_test01/settings.py

`settings.py`에서 다음 부분을 찾은 다음, ENGINE과 NAME을 다음과 같이 수정한다. NAME은 위에 mySQL에 접속해 만들었던 데이터베이스 이름을 적어준다. django에서 sqlite 대신 mySQL을 사용하기 위해선 USER와 PASSWORD 항목을 만들어 mySQL 설치시 입력했던 값들을 넣어준다.

파이썬에 내장된 sqlite를 쓸 경우에는 데이터베이스를 미리 만들지 않아도 여기에 데이터베이스명을 쓰면 자동으로 만들어 주는 것 같다. 그런데 mySQL의 경우에는 미리 만들어 주지 않으니까 에러가 발생한다. 그래서 mySQL을 설치하고나서  데이터베이스를 만들어 주었던 것이다.

    DATABASES = {
        'default': {
            'ENGINE': 'mysql.connector.django',
            'NAME': '데이터베이스명',
            'USER': '사용자명',
            'PASSWORD': '비밀번호',
        }
    }

##django 실행

    (env)$ ./manage.py runserver

또는 `python manage.py runserver`도 가능하다.
에러없이 실행되었다면 웹 브라우저에서 http://127.0.0.0:8000 접속 후 "It worked!"를 확인할 수 있다.

네트워크 접속이 필요한 경우에는 

    (env)$ ./manage.py runserver 0.0.0.0:8000

웹 브라우저에서 http://0.0.0.0:8000 접속 후  "It worked!"를 확인할 수 있다.

##데이터베이스 사용하기

    (env)$ python manage.py migrate
    (env)$ python manage.py createsuperuser

migrate를 하면 프로젝트가 데이터베이스를 사용할 수 있게 된다. createsuperuser로 프로젝트 데이터베이스에서 사용할 관리자 권한을 만든다.

##접속해보기

    (env)$ ./manage.py runserver 0.0.0.0:8000

웹 브라우저에서 http://0.0.0.0:8000/admin 접속 후 슈퍼유저 로그인 확인

##어플리케이션 생성

django 프로젝트는 django 어플리케이션들로 구성된다. 각각의 기능을 가진 어플리케이션이 모여 하나의 프로젝트가 된다.

    (env)$ ./manage.py startapp 앱이름


##프로젝트에 어플리케이션 추가

`settings.py` 파일을 수정하여 어플리케이션을 프로젝트에 추가시켜준다.

    (env)$ nano django_test01/settings.py
    INSTALLED_APPS = (
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        '앱이름',
    )

##수정 내용에 대한 migration을 생성.

    (env)$ ./manage.py makemigrations

##수정 내용을 데이터베이스에 적용

    (env)$ ./manage.py migrate

`makemigrations`를 git의 commit, `migrate`를 git의 push 정도로 생각하면 된다.