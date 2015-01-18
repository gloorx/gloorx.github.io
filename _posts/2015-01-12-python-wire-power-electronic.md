---
tag: Hacking
title: Python - 전선, 전원, 그리고 전기
---

*파이썬으로 습작하면서 막혔던 부분에 대한 기록입니다. 코드가 조잡합니다.*

##조건

    [ ][ ][ ][ ]
    [ ][ ][ ][ ]
    [ ][ ][ ][ ]
    [ ][ ][ ][ ]

- 위와 같이 2차원으로 이루어진 비선형적 공간에 전선, 전원을 놓을 수 있다.
- 전원에 연결된 전선은 ON 상태가 되며, 그렇지 않으면 OFF가 된다.
- 인접한 전선끼리도 전기가 통한다.
- 전원은 항상 ON를 유지한다.
- 배치되고 제거될 때마다 상황에 맞게 ON-OFF 상태가 변화한다.

##1차시

전원에서 전기를 보낸다는 생각으로 함수를 만들었다. 주변 전선이 주체. 제거는 생각하지 않았다.

    # 전선
    class Wire(object):
    
        # 블럭의 위치. Wire간의 위치관계를 빠르게 알 수 있게 하기위해 만든 변수.
        x = None
        y = None
        
        # Wire가 놓여있는 공간(컨트롤러). Wire가 초기화될 때 대입된다. Wire에서 상위 공간에 접근할 방법을 몰라서 쓴 변수. 
        controller = None
    
        # 전기가 연결되었는지 여부
        power = False
    
        def transmit_power(self):
            if self.power:
                if (self.x != 0) and self.field[self.x-1][self.y] and not self.field[self.x-1][self.y].power:
                    self.field[self.x-1][self.y].power = True
                    self.field[self.x-1][self.y].transmit_power()
    
                if (self.x != self.field.WIDTH-1) and self.field[self.x+1][self.y] and not self.field[self.x+1][self.y].power:
                    self.field[self.x+1][self.y].power = True
                    self.field[self.x+1][self.y].transmit_power()
    
                if (self.y != 0) and self.field[self.x][self.y-1] and not self.field[self.x][self.y-1].power:
                    self.field[self.x][self.y-1].power = True
                    self.field[self.x][self.y-1].transmit_power()
                    
                if (self.y != self.field.HEIGHT-1) and self.field[self.x][self.y+1] and not self.field[self.x][self.y+1].power:
                    self.field[self.x][self.y+1].power = True
                    self.field[self.x][self.y+1].transmit_power()
                # if 문이 너무 길다고 pep8에서 경고를 보내는데, 여러줄로 쓰니까 SyntaxError가 뜬다.
                # 가장자리나 모서리에 있을 경우 IndexError가 나서 '!='를 이용해 접근을 제한했다.
    
    
    # 전원
    class Source(Wire):
    
        power = True
    
    
    # 컨트롤러
    class Controller(object):
        ...
    
        def setWire(self, wire, x, y):
            self[x][y] = wire
            wire.x = x
            wire.y = y
            wire.transmit_power()
    
        def removeWire(self, x, y):
            self[x][y].controller = None    # Wire가 제거될 때는 None을 대입해서 가비지 컬렉션이 작동할 수 있게 했다.
            self[x][y] = None

###코드 검사

전선을 먼저 배치하고 마지막에 전원을 배치한 경우

    [+][ ][ ][ ]
    [+][ ][ ][ ]
    [+][ ][!][ ]
    [+][+][+][ ]

전선을 먼저 배치하고 마지막에 전원을 배치한 경우 + 전원을 제거

    [+][ ][ ][ ]
    [+][ ][ ][ ]
    [+][ ][ ][ ]
    [+][+][+][ ]

전원을 먼저 배치한 경우

    [ ][ ][ ][ ]
    [ ][ ][-][ ]
    [ ][ ][-][ ]
    [ ][ ][!][ ]

###문제점

- 전원보다 나중에 배치된 전선은 연결되어 있어도 power를 받지 않음.
- power를 받은 전선은 전원과 끊어져도 계속 ON 상태

##2차시

주변의 전선을 탐색해서 ON, OFF를 정하고, 다시 주변 전선의 함수를 실행하는 2단계로 구분되는 함수로 바꾸었다.

    # 전선
    class Wire(object):
        ...
    
        def transmit_power(self):
            if (self.x != 0) and self.field[self.x-1][self.y] and self.field[self.x-1][self.y].power:
                self.power = True
            elif (self.x != self.field.WIDTH-1) and self.field[self.x+1][self.y] and self.field[self.x+1][self.y].power:
                self.power = True
            elif (self.y != 0) and self.field[self.x][self.y-1] and self.field[self.x][self.y-1].power:
                self.power = True
            elif (self.y != self.field.HEIGHT-1) and self.field[self.x][self.y+1] and self.field[self.x][self.y+1].power:
                self.power = True
    
            if self.power:
                if (self.x != 0) and self.field[self.x-1][self.y] and not self.field[self.x-1][self.y].power:
                    self.field[self.x-1][self.y].transmit_power()
    
                if (self.x != self.field.WIDTH-1) and self.field[self.x+1][self.y] and not self.field[self.x+1][self.y].power:
                    self.field[self.x+1][self.y].transmit_power()
    
                if (self.y != 0) and self.field[self.x][self.y-1] and not self.field[self.x][self.y-1].power:
                    self.field[self.x][self.y-1].transmit_power()
    
                if (self.y != self.field.HEIGHT-1) and self.field[self.x][self.y+1] and not self.field[self.x][self.y+1].power:
                    self.field[self.x][self.y+1].transmit_power()
    
    
    # 전원
    class Source(Wire):
    
        power = True
    
    
    # 컨트롤러
    class Controller(object):
        ...
    
        def setWire(self, wire, x, y):
            self[x][y] = wire
            wire.x = x
            wire.y = y
            wire.transmit_power()
    
        def removeWire(self, x, y):
            self[x][y].controller = None
            self[x][y] = None

###코드 검사

전선을 먼저 배치하고 마지막에 전원을 배치한 경우

    [+][ ][ ][ ]
    [+][ ][ ][ ]
    [+][ ][!][ ]
    [+][+][+][ ]

전원을 먼저 배치한 경우

    [+][ ][ ][ ]
    [+][ ][ ][ ]
    [+][ ][!][ ]
    [+][+][+][ ]

##3차시

power가 On될 때 함수와 Off될 때의 함수를 분리했다.
전선이 제거되면 연결되었던 전선들을 Off시키고 전원에 연결된 전선만 다시 On으로 변하도록 했다.

    # 전선
    class Wire(object):
        ...
    
        def power_on(self):  # transmit_power() 함수에서 이름만 바꾸었다.
        ...
    
        def power_off(self):
            self.power = False
    
            if (self.x != 0) and self.field[self.x-1][self.y] and self.field[self.x-1][self.y].power:
                self.field[self.x-1][self.y].power_off()
    
            if (self.x != self.field.WIDTH-1) and self.field[self.x+1][self.y] and self.field[self.x+1][self.y].power:
                self.field[self.x+1][self.y].power_off()
    
            if (self.y != 0) and self.field[self.x][self.y-1] and self.field[self.x][self.y-1].power:
                self.field[self.x][self.y-1].power_off()
    
            if (self.y != self.field.HEIGHT-1) and self.field[self.x][self.y+1] and self.field[self.x][self.y+1].power:
                self.field[self.x][self.y+1].power_off()
    
    
    # 전원
    class Source(Wire):
    
        power = True
    
        def power_off(self):    # 전원은 항상 On이여야 하기 때문에 오버라이드했다.
            self.power_on()
    
    # 컨트롤러
    class Controller(object):
        ...
    
        def setWire(self, wire, x, y):
            self[x][y] = wire
            wire.x = x
            wire.y = y
            wire.power_on()
    
        def removeWire(self, x, y):
            wire = self[x][y]  # power_off() 함수의 무한 실행을 막기위해 객체를    컨트롤러에서 제거하고  power_off()함수를 실행했다.
            self[x][y] = None
            wire.power_off()
            self.controller = None  #wire가 컨트롤러에 접근해서 함수를 실행하기  때문에 이 변수는 마지막에 None으로 한다.

###코드 검사

전선 배치

    [+][ ][ ][ ]
    [+][ ][ ][ ]
    [+][ ][!][ ]
    [+][+][+][ ]

제거 예시 1

    [-][ ][ ][ ]
    [-][ ][ ][ ]
    [-][ ][!][ ]
    [-][-][ ][ ]

제거 예시 2

    [-][ ][ ][ ]
    [-][ ][ ][ ]
    [-][ ][!][ ]
    [-][ ][+][ ]

제거 예시 3

    [+][ ][ ][ ]
    [+][ ][ ][ ]
    [+][ ][ ][ ]
    [+][+][+][ ]

###문제점

- 전원을 제거해도 On 상태가 유지됨.

##4차시

전원의 power_off() 함수를 오버라이드 했다.

    # 전선
    class Wire(object):
        ...
    
    
    # 전원
    class Source(Wire):
    
        power = True
    
        def power_off(self):
            if self.controller[self.x][self.y]:
                self.power_on()
            else:
                super(Source, self).power_off()
    
    # 컨트롤러
    class Controller(object):
        ...

##코드 검사

제거 예시 1

    [-][ ][-][-]
    [-][ ][-][ ]
    [-][ ][ ][ ]
    [-][-][-][ ]

제거 예시 2
(2, 2)의 전선을 제거하면 함수에서 빠져나오지 못하고 점점 깊게 들어가 에러가 난다.

    [!][ ][ ][ ]
    [+][ ][ ][ ]
    [+][+][+][ ]
    [ ][ ][ ][ ]
    # 에러

##5차시

전원에 인접한 전선은 power_off()를 실행해 power가 Off가 되어도 전원의 power_off()에 의해 if문을 지나면서 다시 On 상태가 된다. On 상태에서 다른 전선의 power_off()를 실행하면 두 전선은 서로의 power_off()를 계속 실행하게 된다. if문의 순서에 따라 특정 방향에서만 이 문제가 발생한다.

power_off()의 if문에서 power가 Off인지 확인하도록 했다.

    # 전선
    class Wire(object):
        ...
        def power_off(self):
            self.power = False
    
            if (self.x != 0) and self.field[self.x-1][self.y] and self.field[self.x-1][self.y].power:
                if self.power == False:
                    self.field[self.x-1][self.y].power_off()
    
            if (self.x != self.field.WIDTH-1) and self.field[self.x+1][self.y] and self.field[self.x+1][self.y].power:
                if self.power == False:
                    self.field[self.x+1][self.y].power_off()
    
            if (self.y != 0) and self.field[self.x][self.y-1] and self.field[self.x][self.y-1].power:
                if self.power == False:
                    self.field[self.x][self.y-1].power_off()
    
            if (self.y != self.field.HEIGHT-1) and self.field[self.x][self.y+1] and self.field[self.x][self.y+1].power:
                if self.power == False:
                    self.field[self.x][self.y+1].power_off()
    
    
    # 전원
    class Source(Wire):
        ...
    
    # 컨트롤러
    class Controller(object):
        ...

##코드 검사

전선배치

    [!][ ][ ][ ]
    [+][ ][ ][ ]
    [+][+][+][ ]
    [ ][ ][ ][ ]

제거 예시 2

    [!][ ][ ][ ]
    [+][ ][ ][ ]
    [+][+][ ][ ]
    [ ][ ][ ][ ]