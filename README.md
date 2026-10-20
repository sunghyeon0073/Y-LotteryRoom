<div align="center">

# LotteryRoom

**로또 구매 · 당첨 결과 · 판매점 찾기를 한 사이트에서 하는 모의 로또 웹 서비스**

마일리지로 로또를 자동/수동으로 사고, 회차별 결과를 확인하고, 근처 판매점을 지도에서 찾을 수 있습니다.<br/>
프레임워크 없이 Servlet · JSP로 MVC 구조를 직접 구성한 팀 프로젝트입니다.

![Java](https://img.shields.io/badge/Java-007396?style=flat-square&logo=openjdk&logoColor=white)
![JSP](https://img.shields.io/badge/Servlet_·_JSP-F8981D?style=flat-square&logoColor=white)
![MyBatis](https://img.shields.io/badge/MyBatis_3.5-000000?style=flat-square&logoColor=white)
![Oracle](https://img.shields.io/badge/Oracle_DB-F80000?style=flat-square&logo=oracle&logoColor=white)
![Tiles](https://img.shields.io/badge/Apache_Tiles_3-D22128?style=flat-square&logo=apache&logoColor=white)
![Kakao](https://img.shields.io/badge/Kakao_Map_·_Login-FFCD00?style=flat-square&logo=kakao&logoColor=black)

</div>

<!-- 스크린샷: 이미지를 넣고 경로를 바꿔주세요 -->
<!--
| 메인 | 로또 구매 | 판매점 지도 | 마이페이지 |
|:---:|:---:|:---:|:---:|
| <img src="screenshots/main.png" width="220"/> | <img src="screenshots/buy.png" width="220"/> | <img src="screenshots/map.png" width="220"/> | <img src="screenshots/mypage.png" width="220"/> |
-->

## 📌 프로젝트 소개

| 항목 | 내용 |
|---|---|
| 종류 | 팀 프로젝트 (5명) <!-- 과정명/조 이름에 맞게 수정해주세요 --> |
| 기간 | 2023.05 (단기 프로젝트) |
| 원본 | [mrkim82/LotteryRoom](https://github.com/mrkim82/LotteryRoom) |
| 목표 | Spring 없이 Servlet · JSP · MyBatis만으로 MVC 웹 서비스를 끝까지 만들어 보기 |

> 이 저장소는 팀 프로젝트 LotteryRoom을 커밋 기록과 함께 옮겨온 개인 포트폴리오용 사본입니다.

## ✨ 주요 기능

| 기능 | 설명 |
|---|---|
| 🎰 **로또 구매** | 자동 · 수동 번호 선택, 한 번에 최대 5게임, 마일리지로 결제 |
| 🏆 **당첨 결과** | 회차별 당첨 번호 조회, 실시간 당첨 결과 화면 |
| 📍 **내 주변 판매점 찾기** | 카카오맵 API로 실시간 내 위치 기준 근처 복권 판매점을 지도에 표시, 전국 판매점 목록 · 지역 검색 · 위치 보기 |
| 💰 **마일리지** | 마일리지 충전, 구매 시 차감, 화면에 바로 반영 |
| 📰 **멀티 게시판** | 공지사항 · 언론보도 · 커뮤니티를 게시판 하나로 운영, 댓글 |
| 🙋 **고객센터** | FAQ, 1:1 문의 |
| 👤 **회원** | 회원가입, 로그인 · 카카오 로그인, 아이디 · 비밀번호 찾기, 마이페이지(정보 수정 · 탈퇴) |

## 🛠 기술 스택

| 분류 | 기술 |
|---|---|
| Backend | Java, Servlet, JSP, JSTL |
| Persistence | MyBatis 3.5, Oracle DB (ojdbc8) |
| View | JSP, Apache Tiles 3 (레이아웃), JavaScript |
| 외부 연동 | Kakao Map API (지도 · 장소 검색 · 주소→좌표 변환), 공공데이터포털 복권 판매점 API, 카카오 로그인 |
| 기타 | Lombok, Gson, COS(파일 업로드), Maven(WAR), Tomcat |

## 🏗 아키텍처

```mermaid
flowchart LR
    Browser["브라우저"]

    subgraph WAS["Tomcat"]
        FC["FrontController<br/>(*.do)"]
        Ctrl["Control 구현체<br/>(기능별 40여 개)"]
        Svc["Service"]
        Mapper["MyBatis Mapper"]
        Tiles["Tiles 레이아웃<br/>+ JSP"]
        FC -- "URL → Control 조회" --> Ctrl
        Ctrl --> Svc --> Mapper
        Ctrl -- "뷰 이름 반환" --> Tiles
    end

    DB[("Oracle DB")]
    Kakao["Kakao Map API<br/>(지도 · 장소 검색 · 지오코딩)"]
    OpenAPI["공공데이터포털<br/>복권 판매점 API"]

    Browser -- "*.do 요청" --> FC
    Mapper --> DB
    Browser -- "내 위치 · 주변 판매점" --> Kakao
    Browser -- "판매점 목록" --> OpenAPI
```

## 🔥 기술적 고민과 해결

### 1. 내 위치 기반으로 근처 복권 판매점 찾기 (카카오맵 API)
- **문제:** 판매점 목록만으로는 "지금 내 근처에 어디가 있는지" 알기 어려웠습니다. 주소 텍스트를 보고 사용자가 직접 지도 앱에서 다시 찾아야 했습니다.
- **해결:** 카카오맵 API로 지도를 띄우고 **현재 내 위치를 별 모양 마커**로 표시했습니다. `services.Places`로 내 위치 주변의 복권 판매점을 검색해 각 판매점에 마커를 찍고, 마커를 누르면 인포윈도우에 판매점 이름이 나옵니다. 검색된 판매점 좌표를 모두 `LatLngBounds`에 담아 `setBounds()`를 호출하므로, 결과가 한 화면에 모두 들어오도록 지도 범위가 자동으로 맞춰집니다.
- 📄 `views/store/storesearch.jsp`, `store/control/StoreSearchControl.java`

### 2. 전국 판매점 목록과 주소 → 지도 연결
- **문제:** 공공데이터의 판매점 정보에는 좌표가 없고 **주소 문자열**만 있었습니다. 그래서 목록에서 바로 지도에 표시할 수 없었습니다.
- **해결:** 공공데이터포털 복권 판매점 API에서 전국 판매점(약 6,900곳)을 받아 지역으로 필터링하고, 클라이언트에서 페이징해 보여줍니다. 판매점을 누르면 주소와 상호를 `storemap.do`로 넘깁니다. 지도 화면에서는 카카오 `Geocoder.addressSearch()`로 주소를 좌표로 바꿔 마커와 인포윈도우를 띄웁니다.
- 📄 `views/store/storelist.jsp`, `views/store/map.jsp`, `store/control/StoreMapControl.java`

### 3. 게시판 다섯 종류를 두 구조로 묶기
- **문제:** 공지사항, 언론보도, 커뮤니티, FAQ, 1:1 문의를 게시판마다 따로 만들면 목록 · 상세 · 등록 · 수정 · 삭제 코드가 다섯 벌이 됩니다.
- **해결:** 성격이 같은 게시판끼리 묶어 **게시판 유형 코드**로 구분했습니다. 멀티 게시판은 공지사항(E) · 언론보도(K) · 커뮤니티(J)를, 고객센터 게시판은 FAQ(F) · 1:1 문의(C)를 맡습니다. 각각 테이블 하나와 Control · Mapper 한 벌로 CRUD를 처리하고, 멀티 게시판에는 댓글 기능을 붙였습니다.
- 📄 `multiboard/`, `csboard/`, `reply/`

### 4. 프레임워크 없이 요청을 정리하는 방법: Front Controller 패턴
- **문제:** 기능마다 서블릿을 하나씩 만들면 `web.xml` 매핑이 끝없이 늘어나고, 인코딩 설정 같은 공통 처리가 여러 곳에 흩어집니다.
- **해결:** 모든 `*.do` 요청을 `FrontController` 하나가 받습니다. URL과 `Control` 구현체를 `Map`에 등록해 두고(커맨드 패턴), 요청 URL로 꺼내서 실행합니다. `Control.execute()`가 돌려준 값의 끝으로 응답 방식을 정합니다. `.do`로 끝나면 redirect, `.json`이면 JSON 응답(AJAX), 그 밖에는 JSP · Tiles로 forward합니다. 기능을 추가할 때는 클래스 하나와 `map.put` 한 줄만 쓰면 됩니다.
- 📄 `FrontControl/FrontController.java`, `FrontControl/Control.java`

### 5. 화면마다 반복되는 레이아웃: Apache Tiles
- **문제:** 상단 메뉴, 사이드 메뉴, 하단 영역을 JSP마다 include하면 메뉴 하나를 고칠 때 모든 화면을 손대야 했습니다.
- **해결:** Tiles 와일드카드 정의(`main/*`, `*/*`)로 **메인 레이아웃**과 **메뉴형 레이아웃** 두 가지만 만들었습니다. Control은 `"mypage/mypage.tiles"`처럼 뷰 이름만 돌려주면 알맞은 레이아웃이 자동으로 입혀집니다.
- 📄 `WEB-INF/tiles.xml`, `WEB-INF/layout/`

### 6. 마일리지 결제와 화면 반영
- 로또를 사면 DB에서 마일리지를 차감하고, 세션에 들고 있는 마일리지 값도 같이 갱신합니다. 상단 메뉴의 잔액이 페이지를 다시 불러오지 않아도 바로 맞게 보입니다. 한 번에 여러 게임을 사면 게임마다 구매 기록과 번호를 따로 저장합니다.
- 📄 `lotto/control/AddAutoControl.java`, `lotteryuser/control/MinusMileageControl.java`

## 👤 담당 역할

- **내 주변 판매점 찾기:** 카카오맵 API로 실시간 내 위치 기반 근처 복권 판매점 조회, 마커 · 인포윈도우, 지도 범위 자동 조정
- **판매점 목록 · 지도:** 공공데이터 판매점 API 연동, 지역 검색 · 페이징, 주소 → 좌표 변환으로 판매점 위치 표시
- **게시판:** 멀티 게시판(공지사항 · 언론보도 · 커뮤니티)과 고객센터 게시판(FAQ · 1:1 문의) CRUD, 댓글

## 📂 폴더 구조

```
LotteryRoom/src/main/
├── java/com/lottery/
│   ├── FrontControl/        # FrontController, Control 인터페이스
│   ├── main/                # 메인 화면
│   ├── lotteryuser/         # 회원 · 로그인 · 마이페이지 · 마일리지 · 로또 구매
│   ├── lotto/               # 로또 번호 저장 · 회차별 결과
│   ├── store/               # 판매점 목록 · 검색 · 지도
│   ├── multiboard/          # 공지 · 언론보도 · 커뮤니티
│   ├── csboard/             # FAQ · 1:1 문의
│   ├── reply/               # 댓글
│   └── common/              # DataSource(MyBatis), 페이징, DB 설정
│       (각 도메인: control / service / mapper / domain)
└── webapp/
    ├── WEB-INF/
    │   ├── views/           # 화면 JSP (main, mypage, store, lottoresult ...)
    │   ├── layout/          # Tiles 레이아웃 (dashboard, dashmenu, top ...)
    │   ├── tiles.xml
    │   └── web.xml
    ├── css/
    └── images/
```

## 🚀 실행 방법

1. Oracle DB를 준비하고 `src/main/java/com/lottery/common/db.properties`에 접속 정보를 넣습니다.
2. Eclipse 등에서 Maven 프로젝트로 가져온 뒤 Tomcat 서버에 올려 실행합니다.
3. 브라우저에서 `http://localhost:<포트>/<컨텍스트>/main.do`로 접속합니다.

```bash
# WAR 빌드
cd LotteryRoom
mvn clean package
```

## 🔗 관련 레포

- [mrkim82/LotteryRoom](https://github.com/mrkim82/LotteryRoom): 팀 프로젝트 원본 저장소
- [Y-MODU](https://github.com/sunghyeon0073/Y-MODU): 협업 툴 팀 프로젝트
- [ANG-Capstone](https://github.com/sunghyeon0073/ANG-Capstone): 2026 캡스톤 그룹웨어 프로젝트
