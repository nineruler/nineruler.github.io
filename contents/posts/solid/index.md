---
title: "객체지향 디자인 원칙(SOLID)"
description: "아키텍처의 유연성, 확장성, 유지보수성을 향상시키기 위한 가이드라인"
date: 2023-08-17
update: 2023-08-17
tags:
  - OOP, SOLID
---

## SOLID 디자인 원칙

SOLID는 객체 지향 프로그래밍과 소프트웨어 디자인 원칙을 나타내는 다섯 가지 기본 원칙의 앞글자를 모아놓은 말로, 소프트웨어 아키텍처의 유연성, 확장성, 유지보수성을 향상시키기 위한 가이드라인을 제시한다. SOLID 원칙은 소프트웨어의 품질을 높이기 위한 중요한 개념들을 포함하고 있다.

### SRP(Single Responsibility Principle): 단일 책임 원칙

* 하나의 클래스는 하나의 책임만 가져야 한다.
* 클래스가 변경되어야 하는 이유는 오직 하나여야 한다.
* 클래스가 너무 많은 책임을 갖게 되면 변경 사항에 대한 영향이 많아지고 유지보수가 어려워진다.

### OCP(Open-Closed Principle): 개방-폐쇄 원칙

* 소프트웨어 개체(클래스, 모듈 등)는 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다.
* 새로운 기능을 추가할 때 기존 코드를 수정하지 않고 확장할 수 있도록 설계해야 한다.

### LSP(Liskov Substitution Principle): 리스코프 치환 원칙

* 하위 클래스는 언제나 기본 클래스로 대체 가능해야 한다.
* 상속 관계에서 하위 클래스는 상위 클래스의 기능을 유지하면서 추가적인 기능을 확장해야 한다.

### ISP(Interface Segregation Principle): 인터페이스 분리 원칙

* 클라이언트가 자신이 사용하지 않는 인터페이스에 의존하도록 강요하지 말아야 한다.
* 클라이언트는 필요한 메서드만 있는 인터페이스를 갖게 되어야 한다.
* 성격이 다른 경우 인터페이스를 분리하여 의존성을 높이지 않도록 한다.

### DIP(Dependency Inversion Principle): - 의존성 역전 원칙

* 고수준 모듈은 저수준 모듈에 의존해서는 안 되며, 둘 모두 추상화된 내용에 의존해야 한다.
* 추상화된 내용은 구체적인 내용에 의존해서는 안된다.
